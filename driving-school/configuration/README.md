---
description: >-
  Which file holds which setting — and why most of the numbers are not in a file
  at all.
---

# 🛠️ Configuration

{% hint style="warning" %}
**Read this first.** Most of what you will want to change — fault allowances, tuition, time limits, the theory pass mark, where the instructor stands, where each course is laid out — is **not** in these files. It lives in the in-game admin panel, applies live, and survives updates.

{% content-ref url="../admin-panel.md" %}
[admin-panel.md](../admin-panel.md)
{% endcontent-ref %}
{% endhint %}

The files below hold the things a menu is the wrong place for: which classes exist, what each licence is called in your system, and the physical layout of every course.

***

## <mark style="color:yellow;">**The files**</mark>

| File                     | What it holds                                                                   |
| ------------------------ | ------------------------------------------------------------------------------- |
| `shared/sh.config.lua`   | The classes, the licence names, the instructor model, retention, the debug flag. |
| `shared/sh.courses.lua`  | Everything the obstacle courses share: the fault list, tolerances, markers.      |
| `shared/sh.traffic.lua`  | Everything the road exams share: traffic lights and the always-on rules.         |
| `shared/sh.<class>.lua`  | One file per class — the actual course layout, props and route.                  |
| `shared/sh.lang.lua`     | Every line of text the player and the panel can see.                             |
| `server/s.hooks.lua`     | Three open functions you fill in. Documented on the Exports & Events page.       |
| `html/config.js`         | Number formatting and interface sound volume.                                    |
| `html/colors.css`        | The interface palette.                                                           |

{% hint style="danger" %}
`shared_scripts` in `fxmanifest.lua` is an ordered list, not a glob, and the class files write into tables the two shared files create. Do not reorder it.
{% endhint %}

***

## <mark style="color:yellow;">**Pages**</mark>

{% content-ref url="sh.config.lua.md" %}
[sh.config.lua.md](sh.config.lua.md)
{% endcontent-ref %}

{% content-ref url="sh.courses.lua.md" %}
[sh.courses.lua.md](sh.courses.lua.md)
{% endcontent-ref %}

{% content-ref url="sh.traffic.lua.md" %}
[sh.traffic.lua.md](sh.traffic.lua.md)
{% endcontent-ref %}

{% content-ref url="class-courses.md" %}
[class-courses.md](class-courses.md)
{% endcontent-ref %}

{% content-ref url="sh.lang.lua.md" %}
[sh.lang.lua.md](sh.lang.lua.md)
{% endcontent-ref %}

{% content-ref url="config.js.md" %}
[config.js.md](config.js.md)
{% endcontent-ref %}
