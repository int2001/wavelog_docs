# Userdata on S3 Storage

Wavelog stores user files (QSL cards, eQSL cards, QSL postcard images) in the `userdata` folder, see [Centralized User Data](centralized-userdata.md). Wavelog has no native S3 support, but you can mount an S3 bucket as a filesystem on that folder. Wavelog doesn't notice the difference and no change in `config.php` is needed.

This works with any S3 compatible storage like AWS S3, RustFS, Ceph RGW, Garage or Hetzner Object Storage.

!!! info "Only userdata"
    Only put `userdata` on S3. All other writable folders (`application/config`, `uploads`, `backup`, `updates`, `application/logs`, `application/cache`) stay on a regular volume.

## Linux Server

Use [rclone](https://rclone.org/commands/rclone_mount/) to mount the bucket, started by a systemd unit. The examples assume Debian/Ubuntu with Wavelog in `/var/www/html`, see [Linux installation](../../getting-started/installation/linux.md).

### 1. Install rclone

```bash
sudo apt install rclone fuse3
```

### 2. Configure the S3 remote

Create `/etc/rclone/rclone.conf` and make it readable for root only with `sudo chmod 600 /etc/rclone/rclone.conf`:

```ini
[s3]
type = s3
provider = Other
endpoint = https://s3.example.com
access_key_id = YOUR_ACCESS_KEY
secret_access_key = YOUR_SECRET_KEY
```

Set `provider` to match your storage (`AWS`, `Ceph`, `Other`, ...). See the [rclone S3 docs](https://rclone.org/s3/) for all options.

### 3. Create the systemd unit

Create `/etc/systemd/system/wavelog-userdata.service`:

```ini
[Unit]
Description=Wavelog userdata on S3
Wants=network-online.target
After=network-online.target

[Service]
Type=notify
ExecStart=/usr/bin/rclone mount s3:wavelog-userdata /var/www/html/userdata \
  --config /etc/rclone/rclone.conf \
  --allow-other --uid 33 --gid 33 \
  --vfs-cache-mode writes
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

`uid`/`gid` 33 is `www-data` on Debian/Ubuntu. On other distributions check the webserver user with `id <user>`.

### 4. Start it

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wavelog-userdata.service
findmnt /var/www/html/userdata
```

If `userdata` already contains files, run `sudo mv /var/www/html/userdata /var/www/html/userdata.old && sudo mkdir /var/www/html/userdata` before starting the unit, then copy them back as described in [Migrating existing data](#migrating-existing-data).

## Docker

Use the [rclone Docker volume plugin](https://rclone.org/docker/). It mounts the bucket as a Docker volume, so the container setup stays the same. The plugin only works with Docker Engine on Linux, not with Docker Desktop.

### 1. Install the plugin

```bash
sudo mkdir -p /var/lib/docker-plugins/rclone/config /var/lib/docker-plugins/rclone/cache
docker plugin install rclone/docker-volume-rclone:amd64 args="-v" --alias rclone --grant-all-permissions
```

Use `:arm64` instead of `:amd64` on ARM hosts.

### 2. Configure the S3 remote

Create `/var/lib/docker-plugins/rclone/config/rclone.conf`. This keeps the credentials out of your compose file.

```ini
[s3]
type = s3
# RustFS has no own provider in rclone, use Other
provider = Other
endpoint = https://s3.example.com
access_key_id = YOUR_ACCESS_KEY
secret_access_key = YOUR_SECRET_KEY
```

Set `provider` to match your storage (`AWS`, `Ceph`, `Other`, ...). See the [rclone S3 docs](https://rclone.org/s3/) for all options.

### 3. Use the volume in docker-compose.yml

Replace the `wavelog-userdata` entry in the `volumes:` section of your [docker-compose.yml](../../getting-started/installation/docker.md):

```yaml
volumes:
  wavelog-userdata:
    driver: rclone
    driver_opts:
      remote: 's3:wavelog-userdata' # <- remote name from rclone.conf : bucket name
      allow_other: 'true'
      uid: '33'
      gid: '33'
      vfs_cache_mode: writes
```

`uid`/`gid` 33 is `www-data` inside the Wavelog container. Then recreate the stack with `docker compose up -d`.

## Kubernetes

Use the CSI driver [k8s-csi-s3](https://github.com/yandex-cloud/k8s-csi-s3) instead of a block storage PVC. As a bonus the volume is `ReadWriteMany`, so multiple Wavelog replicas can share the same userdata.

### 1. Install the driver

```bash
helm repo add yandex-s3 https://yandex-cloud.github.io/k8s-csi-s3/charts
helm install csi-s3 yandex-s3/csi-s3 --namespace kube-system \
  --set secret.endpoint=https://s3.example.com \
  --set secret.accessKey=YOUR_ACCESS_KEY \
  --set secret.secretKey=YOUR_SECRET_KEY \
  --set storageClass.singleBucket=wavelog-userdata \
  --set storageClass.mountOptions="--memory-limit 1000 --stat-cache-ttl 5s --uid 33 --gid 33 --dir-mode 0755 --file-mode 0644" \
  --set storageClass.reclaimPolicy=Retain
```

This creates the StorageClass `csi-s3`. Check the [chart values](https://github.com/yandex-cloud/k8s-csi-s3/tree/master/deploy/helm/csi-s3) for all options.

- `--stat-cache-ttl 5s`: GeeseFS caches file metadata for 1 minute by default. With multiple Wavelog replicas a freshly uploaded QSL card is invisible to the other replicas for that time and returns a 404. A few seconds is a good tradeoff between delay and S3 requests.
- `reclaimPolicy=Retain`: With the default `Delete` the data in the bucket is deleted together with the PVC.

!!! warning "Mount options can't be changed later"
    The options are stored in the StorageClass and in every PersistentVolume created from it. Both are immutable. To change them, delete and recreate the StorageClass, then recreate the PVC and copy the data again (see [Migrating existing data](#migrating-existing-data)).

### 2. Create the PVC and mount it

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wavelog-userdata
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: csi-s3
  resources:
    requests:
      storage: 10Gi # ignored by S3, but required by Kubernetes
```

In your Wavelog deployment:

```yaml
spec:
  template:
    spec:
      containers:
        - name: wavelog
          volumeMounts:
            - name: userdata
              mountPath: /var/www/html/userdata
      volumes:
        - name: userdata
          persistentVolumeClaim:
            claimName: wavelog-userdata
```

!!! note "Mountpoint for Amazon S3"
    The AWS CSI driver "Mountpoint for Amazon S3" is not recommended. By default it can't overwrite or delete files, which Wavelog needs.

## Things to keep in mind

!!! warning
    - **Latency:** Every file access goes over the network. For QSL images this is fine, keep the cache options enabled.
    - **Permissions:** The webserver runs as `www-data` (uid/gid 33). Without the correct `uid`/`gid` Wavelog can't write to the folder.
    - **PUID/PGID:** If you set `PUID`/`PGID` on the Wavelog container, use the same values for `uid`/`gid` on the mount. Otherwise the container tries to `chown` the mounted folder at startup, which fails on S3 and the container doesn't start.
    - **Keep the bucket private:** Wavelog serves the files itself. The bucket doesn't need any public access.

## Migrating existing data

1. Enable the [maintenance mode](../administration/maintenance-mode.md).
2. Back up your current `userdata` folder.
3. Start Wavelog with the new S3 volume and copy the old data into it:
    - Linux server: `sudo cp -r /var/www/html/userdata.old/. /var/www/html/userdata/`
    - Docker: `docker cp ./userdata/. wavelog-main:/var/www/html/userdata/`
    - Kubernetes: `kubectl cp ./userdata/. <wavelog-pod>:/var/www/html/userdata/`
4. Go to **Admin > Debug** and check that `userdata` is shown as writable.
5. Open some QSL and eQSL cards in the web UI, upload a test QSL card and delete it again.
6. Disable the maintenance mode.
