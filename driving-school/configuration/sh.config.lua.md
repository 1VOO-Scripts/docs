---
description: >-
  The classes the school teaches, what each one awards, and how the licence is
  printed.
---

# sh.config.lua

The main config. Everything here is a decision a menu is the wrong place for — which classes exist, what your licence system calls them, and how long records are kept.

***

## <mark style="color:yellow;">**Config.Debug**</mark>

```lua
Config.Debug = false
```

* **Description**: Turns on console tracing on both sides and registers the dev commands. <mark style="color:red;">**Ship it off.**</mark>
* **Note**: This is more than a logging switch. With it on, the school registers commands that force-pass stages and spawn practice courses. Those commands are **admin-only** — the server checks every one of them against your framework's admin permission — but the extra console output alone is enough reason to leave it off on a live server.

***

## <mark style="color:yellow;">**Config.AttemptRetentionDays**</mark>

```lua
Config.AttemptRetentionDays = 30
```

* **Description**: How many days of exam attempts to keep. Both attempt tables are written on every run and pruned once a day. `0` keeps them forever.
* **Example**: `30` keeps a month of history. Set it above your longest retry cooldown — a prune that removes the failure a cooldown is counting from hands the player a free retry.

***

## <mark style="color:yellow;">**Config.DefaultStages**</mark>

```lua
Config.DefaultStages = { 'theory', 'obstacle', 'traffic' }
```

* **Description**: The stages a class asks for, in unlock order, unless the class names its own list. Each stage opens when the one before it is passed.
* **Example**: Removing `'traffic'` here makes every class a two-stage licence.

***

## <mark style="color:yellow;">**Config.Schools**</mark>

```lua
Config.Schools = {
    { id = 'car',        label = 'Car',        class = 'Class B',    icon = 'fa-duotone fa-car' },
    { id = 'motorcycle', label = 'Motorcycle', class = 'Class A',    icon = 'fa-duotone fa-motorcycle' },
    { id = 'truck',      label = 'Truck',      class = 'Class C',    icon = 'fa-duotone fa-truck', requires = 'car' },
    { id = 'boat',       label = 'Boat',       class = 'Marine',     icon = 'fa-duotone fa-ship', stages = { 'theory', 'obstacle' } },
    { id = 'helicopter', label = 'Helicopter', class = 'Rotorcraft', icon = 'fa-duotone fa-helicopter', stages = { 'theory', 'obstacle' } },
    { id = 'plane',      label = 'Plane',      class = 'Fixed wing', icon = 'fa-duotone fa-plane', stages = { 'theory', 'obstacle' } },
}
```

* **Description**: The licence classes, in the order the menu draws them.
* **Fields**:
  * `id` *(string)* — the internal name. It is the key used by every other table in the script, by the course files, and by the admin panel. **Changing it orphans that class's saved progress.**
  * `label` *(string)* — what the player sees.
  * `class` *(string)* — the sub-heading on the menu card, e.g. `Class B`.
  * `icon` *(string)* — a Font Awesome class.
  * `requires` *(string, optional)* — a class that must be **fully completed** before this one can be enrolled in. Truck requires car by default.
  * `stages` *(table, optional)* — overrides `Config.DefaultStages` for this class. Boats and aircraft list two, because an exam in traffic at sea or in the air would only be the obstacle course again.
* **Note**: Tuition is **not** here — it is in the admin panel, keyed by `id`, so reordering this list never moves a saved price.

***

## <mark style="color:yellow;">**Config.License**</mark>

```lua
Config.License = {
    templates = {
        car        = 'driving_license_abc',
        motorcycle = 'driving_license_abc',
        truck      = 'driving_license_abc',
        boat       = 'driving_license_boat',
        helicopter = 'driving_license_planehelicopter',
        plane      = 'driving_license_planehelicopter',
    },
}
```

* **Description**: What each class awards, named the way **your** licence system names it. The school hands this name to devhub\_lib, which writes it wherever your server keeps licences.
* **What the name means on each system**:
  * **devhub\_licenses** — a template name from `devhub_licenses_templates`.
  * **ESX** — a `user_licenses` type.
  * **QB / QBOX** — a metadata key.
  * **vRP** — a permission name.
* **Example**: Three cards, not six. Car, motorcycle and truck all print `driving_license_abc`, so the classes earned inside a group share one card and the second one passed is written onto the card already in the player's pocket.
* **Note**: Omit a class to have it award nothing — it still teaches the course, it just hands over no licence.
* **Note**: On/off, grant-only and expiry live in the admin panel under <mark style="color:yellow;">Licence</mark>, not here.

***

## <mark style="color:yellow;">**Config.LicenceCategories**</mark>

```lua
Config.LicenceCategories = {
    groups = {
        car   = { 'car', 'motorcycle', 'truck' },
        boat  = { 'boat' },
        plane = { 'plane', 'helicopter' },
    },

    fields = {
        car   = 'driving_school_ABC',
        plane = 'driving_school_PLANEHELICOPTER',
    },

    names = {
        -- car        = 'B',
        -- motorcycle = 'A',
        -- truck      = 'C',
    },

    separator = ', ',
    empty     = '',
}
```

* **Description**: Which classes print on which card, and how that line is written. Read from outside with [`getCategories`](../exports-and-events.md).
* **Fields**:
  * `groups` — which classes share a card. Any class `id` also resolves to the group containing it, so a caller with a class in hand does not need to know the layout.
  * `fields` — <mark style="color:red;">devhub\_licenses only.</mark> The data field each group's line is written into, so a card already printed is brought up to date when a second class is passed. A group left out is never rewritten — which is right for `boat`, since a one-class group can never gain a second class. These names must match the `Config.DataFields` entries you added during [installation](../installation.md).
  * `names` — what a class is called **on the card**. Falls back to its `label`. Uncomment to print `A, B, C` instead of `Car, Motorcycle, Truck`.
  * `separator` — placed between two classes on one line.
  * `empty` — what a group with nothing earned in it prints.

***

## <mark style="color:yellow;">**Config.DrivingSchoolPed**</mark>

```lua
Config.DrivingSchoolPed = {
    model = 'a_m_y_business_02',
    blip = {
        enabled = true,
        sprite = 280,
        color = 5,
        scale = 0.8,
    },
}
```

* **Description**: The instructor. The `model` here is the **starting** one, and the blip settings are permanent.
* **Fields**:
  * `model` *(string)* — the ped model. Changing it here needs a restart; changing it in the admin panel swaps the ped live and checks the name against your own game first.
  * `blip.enabled` *(boolean)* — draw a map marker at the school.
  * `blip.sprite` / `blip.color` / `blip.scale` — standard FiveM blip values.
* **Note**: **Where** the instructor stands is not here — it is in the admin panel under <mark style="color:yellow;">Locations</mark>, where it can be captured from your own position. What the blip **says** is `blip_label` in [sh.lang.lua](sh.lang.lua.md).

***

## <mark style="color:yellow;">**Ev and the helpers**</mark>

The top of the file also defines `Ev()`, `debugPrint()`, `safeEncode()` and `_T()`. These are plumbing that has to exist before any other file runs.

{% hint style="danger" %}
Do not move or rename them. Shared scripts load before client and server scripts, which is the only reason a trace on the first line of another file has something to call.
{% endhint %}
