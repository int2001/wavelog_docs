# Userdata on S3 Storage

Wavelog stores user files (QSL cards, eQSL cards, QSL postcard images) in the `userdata` folder, see [Centralized User Data](centralized-userdata.md). Wavelog has no native S3 support, but you can mount an S3 bucket as a filesystem on that folder. Wavelog doesn't notice the difference and no change in `config.php` is needed.

This works with any S3 compatible storage like AWS S3, RustFS, Ceph RGW, Garage or Hetzner Object Storage.

!!! info "Which folders belong on S3"
    `userdata` and `backup` are a good fit: files are written once and read rarely. Mount `backup` the same way as shown below for `userdata`, with its own bucket or prefix.

    The other writable folders (`application/config`, `application/logs`, `application/cache`, `uploads`, `updates`) technically work on S3 too, but are **not recommended**. They are read on every request or written often in small pieces, which is slow and causes many S3 requests. Keep them on a regular volume.

## Linux Server

Use [rclone](https://rclone.org/commands/rclone_mount/) to mount the bucket, started by a systemd unit. The examples assume Debian/Ubuntu with Wavelog in `/var/www/html`, see [Linux installation](../../getting-started/installation/linux.md).

### 1. Install rclone

```bash
sudo apt install rclone fuse3 ca-certificates
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

## Kubernetes (experimental)

!!! danger "Experimental"
    S3 on Kubernetes works, but has a drawback that matters in production: if the CSI driver pod restarts, Wavelog loses access to its files until the Wavelog pods are restarted as well. See [Driver restarts](#driver-restarts) below. For production a regular `ReadWriteMany` volume (CephFS, NFS, ...) is the safer choice.

Use the CSI driver [csi-rclone](https://github.com/SwissDataScienceCenter/csi-rclone) by the Swiss Data Science Center instead of a block storage PVC. It uses rclone like the setups above. As a bonus the volume is `ReadWriteMany`, so multiple Wavelog replicas can share the same userdata.

### 1. Install the driver

```bash
helm repo add renku https://swissdatasciencecenter.github.io/helm-charts
helm install csi-rclone renku/csi-rclone --namespace csi-rclone --create-namespace
```

This creates the StorageClass `csi-rclone`. Check the [chart values](https://github.com/SwissDataScienceCenter/csi-rclone/blob/master/deploy/csi-rclone/values.yaml) for all options.

The driver pods run privileged (FUSE). If your cluster enforces Pod Security Standards, label the namespace with `pod-security.kubernetes.io/enforce=privileged`. If NetworkPolicies restrict egress, the driver pods need access to the Kubernetes API and your S3 endpoint.

### 2. Create the Secret

The driver reads the rclone configuration from a Secret with the **same name as the PVC** in the same namespace:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: wavelog-userdata # must match the PVC name
  namespace: wavelog
type: Opaque
stringData:
  remote: s3
  remotePath: wavelog-userdata # bucket name, optionally with a prefix: bucket/prefix
  configData: |
    [s3]
    type = s3
    provider = Other
    endpoint = https://s3.example.com
    access_key_id = YOUR_ACCESS_KEY
    secret_access_key = YOUR_SECRET_KEY
  vfsOpt: '{"UID": 33, "GID": 33, "DirCacheTime": "5s", "CacheMode": "writes"}'
```

- `remotePath` is mounted as is. If you delete and recreate the PVC, it finds its data again. The driver never deletes data in the bucket.
- `vfsOpt` takes [rclone VFS options](https://rclone.org/commands/rclone_mount/#vfs-virtual-file-system) in JSON. `UID`/`GID` 33 is `www-data` inside the Wavelog image.
- `DirCacheTime` is how long rclone caches directory listings (default 1 minute). With multiple Wavelog replicas a freshly uploaded QSL card is invisible to the other replicas for that time and returns a 404. A few seconds is a good tradeoff between delay and S3 requests.
- The options are read when the volume is mounted. To change them, edit the Secret and restart the Wavelog pods.

### 3. Create the PVC and mount it

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wavelog-userdata
  namespace: wavelog
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: csi-rclone
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

!!! warning "Multiple replicas"
    Even with a short `DirCacheTime` a new file takes a few seconds to show up on the other replicas, because rclone uploads it shortly after it was written. Sticky sessions on your ingress controller send each user to the same replica, so the uploader sees the file immediately.

!!! note "Mountpoint for Amazon S3"
    The AWS CSI driver "Mountpoint for Amazon S3" is not recommended. By default it can't overwrite or delete files, which Wavelog needs.

### Driver restarts

rclone runs inside the driver's node pod (`csi-rclone-nodeplugin`). If that pod restarts, every mount on that node is gone. The Wavelog pods on that node keep running, but `userdata` fails with `Transport endpoint is not connected`: QSL images return errors and uploads fail. Kubernetes doesn't remount volumes of running pods, and restarting only the container doesn't help either. The whole pod has to be recreated. This is a limitation of FUSE based CSI drivers in general.

When it happens:

- **Driver upgrade:** The DaemonSet replaces all driver pods right away, also on nodes with running Wavelog pods. This is the main cause.
- **Driver crash:** Rare, for example when the rclone daemon fails its liveness probe.
- **Node reboot or drain:** Harmless, the Wavelog pods are restarted or moved anyway.

After a driver restart, recreate the Wavelog pods:

```bash
kubectl -n wavelog rollout restart deployment/wavelog
```

To avoid surprise restarts on upgrades, set the update strategy of the driver DaemonSet to `OnDelete`. A new driver version is then only rolled out when its pod is deleted, for example during the next node drain, when the Wavelog pods move anyway. The chart has no value for this, use a [Helm post-renderer](https://helm.sh/docs/topics/advanced/#post-rendering) or patch the DaemonSet:

```bash
kubectl -n csi-rclone patch daemonset csi-rclone-nodeplugin -p '{"spec":{"updateStrategy":{"type":"OnDelete"}}}'
```

A `helm upgrade` resets a manual patch, so a post-renderer is the permanent solution.

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
