---
description: Number formatting and interface sound volume.
---

# config.js

A small file in `html/`, read by the interface as it loads. It ships un-bundled, so you can edit it without rebuilding anything.

```javascript
window.config = {
    numberFormatting: [/\B(?=(\d{3})+(?!\d))/g, " "],
    soundVolume: 0.25,
};
```

***

## <mark style="color:yellow;">**numberFormatting**</mark>

```javascript
numberFormatting: [/\B(?=(\d{3})+(?!\d))/g, " "],
```

* **Description**: How thousands are separated wherever the interface prints a number — tuition prices, most of all. The first entry is the pattern that finds each break point; the second is what gets inserted there.
* **Example**: The shipped value renders `12000` as `12 000`. For a comma, use `","` and you get `12,000`. For a full stop, `"."` gives `12.000`.

***

## <mark style="color:yellow;">**soundVolume**</mark>

```javascript
soundVolume: 0.25,
```

* **Description**: Volume of the interface sounds, from `0.0` to `1.0`.
* **Example**: `0` silences them entirely.

***

{% hint style="info" %}
This file is loaded at page load. Changing it needs the player to reopen the menu, not a server restart.
{% endhint %}
