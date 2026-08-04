# Confirmations & Cards

Once your log is synchronised with the [online services](index.md#service-overview), Wavelog gives you three views on what came back: a combined confirmation list, a gallery of scanned paper cards and a gallery of eQSL card images. All three live in the **Logbook** menu.

## View last confirmations

`Logbook -> View last confirmations`

A filterable list of the most recent confirmations across all services. Pick one or more confirmation types:

* LoTW
* QSL card (paper)
* eQSL
* QRZ.com
* Clublog

The types preselected when you open the page come from **Default QSL-Methods** in your user profile (`User menu -> Edit profile -> Third Party Services`). Set them to the services you actually use, and the page opens with a useful filter every time.

!!! note
    A maximum of 1000 rows is shown, for performance reasons. Use the filters to narrow the result if you hit that limit.

## View QSL Cards

`Logbook -> View QSL Cards`

The gallery of **scanned paper cards** you uploaded yourself. Switch between *List View* and *Gallery View*; the page also shows how much disk space your card images use.

### Uploading a card

Open a QSO and upload the scan for the **front** and, optionally, the **back** of the card. Wavelog validates that the file really is an image and stores it under a randomly generated filename.

!!! warning
    Uploads are refused when free disk space runs low. Card images are stored on your server, so keep an eye on the storage figure shown on the QSL Cards page.

### One card, several QSOs

A single card often confirms several QSOs. Use **Add Qsos** on a card to search for a callsign and attach the same image to further QSOs, instead of uploading it again.

### Viewing and deleting

**View** opens a carousel with front and back of the card. **Delete** removes the image from the QSO.

!!! note
    This feature can be switched off instance-wide with `$config['disable_qsl'] = true;` in `config.php`. The menu entry then disappears.

## View eQSL Cards

`Logbook -> View eQSL Cards`

The gallery of card images fetched from **eQSL.cc**. Cards are downloaded and cached on your server the first time you click the received arrow of a confirmed QSO in the logbook — so you can look at the card without logging into eQSL.cc.

If you would rather fetch many cards at once, the eQSL page offers a mass download. It is disabled by default and has to be enabled in `config.php`:

```php
$config['enable_eqsl_massdownload'] = true;
```

A single run downloads at most 150 cards; repeat it until you are done.

See [eQSL.cc](eqsl.md) for the sync setup itself.
