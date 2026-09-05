---
description: >-
  A full driving school for your city: theory exams, a private obstacle course
  and an exam in real traffic, ending in a licence your licence system actually
  issues.
---

# 🚗 DRIVING SCHOOL

Six licence classes — car, motorcycle, truck, boat, helicopter and plane — each with its own stages. Students pay tuition at the instructor, sit a written theory exam, then drive a private obstacle course and an exam in live traffic, scored on faults you decide the value of. Pass every stage a class asks for and the licence is issued through whichever system your server runs. Almost all the tuning is done from the in-game admin panel and applies live, with no restart and no file editing.

{% hint style="info" %}
[Tebex Store](https://store.devhub.gg/product/PLACEHOLDER)
{% endhint %}

{% hint style="success" %}
**New here? Read these three pages in order:**

1. [installation.md](installation.md "mention") to get it running — including how to connect it to devhub\_licenses.
2. [script-flow.md](script-flow.md "mention") to understand what the script actually does.
3. [admin-panel.md](admin-panel.md "mention") to tune it. This is where almost all the configuration happens.
{% endhint %}

***

## <mark style="color:yellow;">**Everything in this documentation**</mark>

{% content-ref url="installation.md" %}
[installation.md](installation.md)
{% endcontent-ref %}

{% content-ref url="script-flow.md" %}
[script-flow.md](script-flow.md)
{% endcontent-ref %}

{% content-ref url="admin-panel.md" %}
[admin-panel.md](admin-panel.md)
{% endcontent-ref %}

{% content-ref url="configuration/README.md" %}
[README.md](configuration/README.md)
{% endcontent-ref %}

{% content-ref url="exports-and-events.md" %}
[exports-and-events.md](exports-and-events.md)
{% endcontent-ref %}

{% content-ref url="ui-color-customization.md" %}
[ui-color-customization.md](ui-color-customization.md)
{% endcontent-ref %}

***

## <mark style="color:yellow;">**At a glance**</mark>

| | |
| --- | --- |
| **Classes** | Car, Motorcycle, Truck, Boat, Helicopter, Plane |
| **Stages** | Theory paper, obstacle course, exam in traffic — per class, any of them switchable off |
| **Licence systems** | devhub\_licenses, ESX, QB, QBOX, vRP |
| **Inventories** | qb-inventory, ox\_inventory |
| **Live tuning** | Allowances, tolerances, fault points, tuition, time limits, the theory paper, the instructor and every course anchor |
| **Requires** | [devhub\_lib](../scripts/devhub_lib-needed-for-each-script/), oxmysql |

***

## <mark style="color:yellow;">**What makes it fair**</mark>

Driving is physics on the player's machine, so the client reports **which faults happened** and the server decides what they are worth — re-adding the points from its own config, checking the run was plausible, and recording every attempt either way. A run that could not have been driven is stored as unverified rather than dropped, so a pattern of them is visible afterwards.

Everything else — enrolment, tuition, the theory grade, stage unlocking and the licence itself — is decided on the server outright. The theory answer key never leaves it.
