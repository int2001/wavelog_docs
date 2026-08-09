# On-Air Widget

On-Air widget displays on which frequency you are currently active, utilizing your [CAT radio](../../user-guide/integrations/radio-interface.md) connected to Wavelog.

The callsign shown in the widget is the callsign of your **active station location**, not your account callsign. If no station location is active, your account callsign is used instead.

The widget texts are translated. Visitors see the widget in their own language, if a translation is available, otherwise in English.

!!! warning
    Please be aware of possible privacy implications of using this widget, as this will show your current QRG in real-time.

## How does it look?

### On-Air variant

<img width="598" alt="Screenshot 2025-02-23 at 19 28 18" src="https://github.com/user-attachments/assets/a607cbd1-db54-4c4c-941c-f2f4321d49cf" />

### Off-Air variant

<img width="541" alt="Screenshot 2025-02-23 at 19 28 40" src="https://github.com/user-attachments/assets/d0eccf86-1d4f-4dfb-aeb4-864f1a7bb58b" />

!!! note
    The frequency also includes the mode reported by the radio. If your radio operates in split, the widget displays `RX / TX`.

## Widget configuration

- As already noted, in order for this widget to work, you need at least one [CAT radio](../../user-guide/integrations/radio-interface.md) connected and working.
- The widget is **disabled by default** and needs to be explicitly enabled by user. Widget configuration is located in `Account > Widgets` settings.
- The **URL to your widget** is displayed under the "Enabled" setting. This URL is unique for each user of the Wavelog, is generated automatically and cannot be changed.

| setting | purpose | default |
|----|----|----|
|`Enabled`|Enables the widget and reveals your personal widget URL|off|
|`Display "Last seen" time`|Shows how long ago the radio was last seen, when you are off-air|off|
|`Display radio name`|Shows the name of the radio in front of the frequency|off|
|`Display only most recently updated radio`|With multiple CAT radios: show only the most recently updated one, or all radios that are on-air. Has no effect with a single radio|Yes|

### URL settings

Some aspects of the widget can be further configured via GET parameters appended to the URL. If some option is not supplied, the default value is used.

```text
https://[wavelog url]/index.php/widgets/on_air/xxxxxxxxxx?[option0]=[value]&[option1]=[value]&[option2]=[value]
```

| option | expected value | purpose | default value |
|----|----|----|---|
|`text_size`|`1` - `6`|font size used for the text displayed in widget|1|
|`theme`|folder name of any theme installed in your Wavelog, e.g. `default` / `cyborg` / `darkly` / `cosmo` / `superhero` / `color_impaired`|appearance theme|global theme of the Wavelog instance|
|`radio_timeout_seconds`|`60` or higher|how many seconds must pass since the last radio update for radio to be considered off-air|value, that is set in global Wavelog settings|
|`nojs`|`1`|Forces qrz.com to update the widget from time to time without reloading the whole page|disabled|

### Example URL

```text
https://[wavelog url]/index.php/widgets/on_air/xxxxxxxxxx?text_size=3&theme=darkly&radio_timeout_seconds=90&nojs=1
```

## Usage

For embedding this widget on your website, or for example on QRZ.com, you can use the following code snippet. Do not forget to use **your** widget URL that you will find in `Account > Widgets` settings.

```html
<iframe align="top"
  frameborder="0"
  height="200"
  id="on_air_widget"
  name="on-air"
  src="https://YOUR_WAVELOG_URL.com/index.php/widgets/on_air/XXXXXXXXXX?text_size=4&amp;theme=superhero&amp;nojs=1"
  style="border-radius: 1.5rem;" width="700">
</iframe>
```
