---
description: Recolour the school menu, the exam paper and the course HUD.
---

# 🎨 UI Color Customization

## <mark style="color:yellow;">Usage</mark>

{% stepper %}
{% step %}
### Enter the folder with the script
{% endstep %}

{% step %}
### Enter `html` folder
{% endstep %}

{% step %}
### Open the `colors.css` file
{% endstep %}
{% endstepper %}

#### The code looks like this

```css
/* YOU CAN'T USE A COMMA BETWEEN RGB */
--primary: 253 209 64;          /* #fdd140 */
--background: 32 46 59;         /* #202e3b */
--disabled: 114 114 114;        /* #727272 */
--background-body: 11 19 32;    /* #0b1320 */
--background-light: 42 61 78;   /* #2a3d4e */

/*!!! YOU MUST USE A COMMA BETWEEN RGB !!!*/
--primary-rgba: 253, 209, 64;   /* #fdd140 */
--grid-dark-rgba: 11, 19, 32;   /* #0b1320 */
```

#### To customize the UI for yourself, simply edit the RGB color.

| Variable             | Where it shows                                                        |
| -------------------- | --------------------------------------------------------------------- |
| `--primary`          | The accent — buttons, headings, the course markers' matching gold, the HUD's fault counter. |
| `--background`       | Panels and cards inside the menu.                                      |
| `--background-body`  | The page behind them.                                                  |
| `--background-light` | Raised rows — a selected class, an answer being hovered.               |
| `--disabled`         | Locked classes and stages that are not open yet.                       |

{% hint style="danger" %}
If you are editing the primary color, we recommend changing it in two locations! `--primary` uses **spaces** between the numbers, `--primary-rgba` uses **commas**. Getting that the wrong way round makes the value invalid and the colour falls back.
{% endhint %}

{% hint style="info" %}
The rest of the file is a full Tailwind palette — slate, red, green and so on. Most of it is unused; it is there so you have the whole scale available if you restyle further. Editing an unused variable changes nothing.
{% endhint %}

***

## <mark style="color:yellow;">Course markers</mark>

The gold discs and chevrons drawn on the ground during an exam are **not** part of this file — they are drawn by the game, not the browser. Their colours are the `r`, `g`, `b`, `a` values in `Config.CourseMarker`, `Config.CourseCircle`, `Config.CourseOrbit` and `Config.CourseBox`.

{% content-ref url="configuration/sh.courses.lua.md" %}
[sh.courses.lua.md](configuration/sh.courses.lua.md)
{% endcontent-ref %}

To keep the world and the interface matching, set them to the same RGB you put in `--primary`. The shipped values are both `253 209 64`.
