---
description: >-
  Install the driving school, register its items, and connect it to
  devhub_licenses so passing a class prints a real licence.
---

# 💻 Installation

{% stepper %}
{% step %}
### Install devhub\_lib

Download and install the required library and configure it according to your framework.

Download [https://github.com/DEVHUB-GG/devhub\_lib ](https://github.com/DEVHUB-GG/devhub_lib)or use command

```bash
git clone https://github.com/DEVHUB-GG/devhub_lib.git
```
{% endstep %}

{% step %}
### Install oxmysql

`oxmysql` is **required** — the school stores enrolment, stage progress and exam attempts in your database. It ships with most frameworks; if yours does not have it, download it from [overextended/oxmysql](https://github.com/overextended/oxmysql).
{% endstep %}

{% step %}
### Install resources from keymaster

Download the <mark style="color:red;">DRIVING SCHOOL</mark> script file from keymaster.
{% endstep %}

{% step %}
### Install the driving school MLO

The school comes with its own map. Download <mark style="color:red;">devhub\_drivingschoolMLO</mark> from keymaster as well — it is a separate asset there — and move it into your `resources` folder alongside the script.

{% hint style="danger" %}
The map resource must keep the name **`devhub_drivingschoolMLO`** exactly, capital letters included.
{% endhint %}

The shipped instructor position and every course anchor are laid out for this map. Skip it and the school still runs, but it will be standing on the stock GTA map — move the instructor and the course anchors somewhere that suits it under <mark style="color:yellow;">/admindevhub → Driving School → Locations</mark>.
{% endstep %}

{% step %}
### Start resources

Move the files to the `resources` folder on your server and add the following lines to your server.cfg in the correct order:

```javascript
ensure oxmysql
ensure devhub_lib
ensure devhub_licenses
ensure devhub_drivingschoolMLO
ensure devhub_drivingschool
```

{% hint style="warning" %}
Ensure the map **before** the script. The school places its instructor and course markers as it starts, and the ground they stand on has to exist first.
{% endhint %}

{% hint style="info" %}
`devhub_licenses` is only needed if you use it as your licence system. The school also issues through ESX `user_licenses`, QB/QBOX metadata and vRP — remove that line if you are on one of those.
{% endhint %}
{% endstep %}

{% step %}
### Database Setup

SQL will be automatically added. In case you don't see the new table(s) in your database, import the `dh_drivingschool.sql` file manually.
{% endstep %}

{% step %}
### Add the items to your inventory

The school hands out three licence items, one per card. Register them in your inventory resource using the file that matches it:

* **qb-inventory / qb-core** — copy the entries from `items/qb/items.lua` into your `qb-core/shared/items.lua`.
* **ox\_inventory** — copy the entries from `items/ox/items.lua` into your `ox_inventory/data/items.lua`.

Then copy the three icons from `items/images` into your inventory's image folder (`qb-inventory/html/images` or `ox_inventory/web/images`).

{% hint style="warning" %}
The item names must stay exactly as shipped — `devhub_driving_license_abc`, `devhub_driving_license_boat` and `devhub_driving_license_planehelicopter`. They are the join between the card template and the physical item, and nothing checks the spelling at runtime.
{% endhint %}
{% endstep %}

{% step %}
### Import the licence card templates

<mark style="color:red;">devhub\_licenses only.</mark>

**Import `items/devhub_licenses.sql` into your database.** It is in the `items` folder of this resource, and unlike the schema above it is **not** imported for you — run it by hand, once.

It adds three ready-made card designs to `devhub_licenses_templates`:

| Template name                     | Card                  | Classes printed on it  |
| --------------------------------- | --------------------- | ---------------------- |
| `driving_license_abc`             | Driving License – ABC | Car, Motorcycle, Truck |
| `driving_license_boat`            | Boat License          | Boat                   |
| `driving_license_planehelicopter` | Pilot License         | Plane, Helicopter      |

Three cards rather than six: the classes inside a group share one card, and a second class earned in that group is written onto the card already in the player's pocket.

These names are what `Config.License.templates` in `shared/sh.config.lua` points at. If you rename a template, rename it there too.
{% endstep %}

{% step %}
### Add the Categories fields to devhub\_licenses

<mark style="color:red;">devhub\_licenses only.</mark> The ABC and Pilot cards each print a **Categories** line listing the classes the holder has actually earned. That line is filled by a data field, which you add to devhub\_licenses.

Open `devhub_licenses/configs/s.main.lua` and add both entries to `Config.DataFields`:

```lua
{
    name = 'driving_school_ABC',
    label = 'Categories',
    default = 'None',
    getData = function(source)
        local ok, categories = pcall(function()
            return exports['devhub_drivingschool']:getCategories(source, 'car')
        end)

        return (ok and categories) or ''
    end,
    fakeFields = {"Car", "Motorcycle", "Truck"},
    showInProfile = true, -- If set to true, this data field will be shown in the profile view
},
{
    name = 'driving_school_PLANEHELICOPTER',
    label = 'Categories',
    default = 'None',
    getData = function(source)
        local ok, categories = pcall(function()
            return exports['devhub_drivingschool']:getCategories(source, 'plane')
        end)

        return (ok and categories) or ''
    end,
    fakeFields = {"Plane", "Helicopter"},
    showInProfile = true, -- If set to true, this data field will be shown in the profile view
},
```

{% hint style="danger" %}
The `name` of each field must match `Config.LicenceCategories.fields` in the school's `shared/sh.config.lua` (`driving_school_ABC` and `driving_school_PLANEHELICOPTER`) **and** the `dataField` baked into the card templates you imported in the previous step. Change one and the Categories line goes blank.
{% endhint %}

The Boat card has no Categories field on purpose — a one-class group can never gain a second class, so there is nothing to list.

The `pcall` is deliberate: if the school is stopped or restarted, the field comes back empty instead of throwing inside a profile view.
{% endstep %}

{% step %}
### Add a licence pickup point at the school

<mark style="color:red;">devhub\_licenses only.</mark> Students collect the printed card from a clerk rather than having it appear at the end of a course. Open `devhub_licenses/configs/sh.main.lua` and add an entry to `Config.LicensePickUp`:

```lua
{
    coords = vector4(-710.85, -1307.95, 5.4, 56.99),
    model = "a_m_y_business_03",
    licenses = { "driving_license_abc", "driving_license_boat", "driving_license_planehelicopter" }, -- optional whitelist; omit for all granted licenses
    enabled = true,
    blip = {
        sprite = 408, -- License/ID card icon
        color = 2, -- Green
        scale = 0.8,
        label = "Driving License Pickup",
    },
},
```

The `licenses` whitelist keeps this clerk to driving licences only. Omit it and the clerk hands over every licence the player has been granted.

The coordinates above place the clerk beside the default instructor position. If you move the instructor in <mark style="color:yellow;">/admindevhub → Driving School → Locations</mark>, move this clerk with it.
{% endstep %}

{% step %}
### Restart your server
{% endstep %}
{% endstepper %}

{% hint style="danger" %}
DO NOT CHANGE RESOURCE NAME
{% endhint %}

***

## <mark style="color:yellow;">**Checklist — connecting to devhub\_licenses**</mark>

Steps 8 to 10 are the whole integration. If a licence is not printing, check them in this order:

| # | What                                              | Where                                   |
| - | ------------------------------------------------- | --------------------------------------- |
| 1 | `items/devhub_licenses.sql` imported              | Your database                           |
| 2 | Three items registered, icons copied              | Your inventory resource                 |
| 3 | Both `Config.DataFields` entries added            | `devhub_licenses/configs/s.main.lua`    |
| 4 | Pickup point added                                | `devhub_licenses/configs/sh.main.lua`   |
| 5 | Template names match                              | `devhub_drivingschool/shared/sh.config.lua` |

***

## <mark style="color:yellow;">**Using a different licence system**</mark>

The school does not care what a licence *is* — it hands a name to devhub\_lib, which writes it wherever your server keeps licences. Only the names in `Config.License.templates` change:

* **devhub\_licenses** — the template name, as imported above.
* **ESX** — a `user_licenses` type.
* **QB / QBOX** — a metadata key.
* **vRP** — a permission name.

Steps 8 to 10 are devhub\_licenses only. On any other system, set the names in `shared/sh.config.lua` and you are done — the Categories line and the pickup clerk have no equivalent there.

***

{% content-ref url="../id-card-and-license/README.md" %}
[README.md](../id-card-and-license/README.md)
{% endcontent-ref %}

{% content-ref url="../scripts/devhub_lib-needed-for-each-script/" %}
[devhub\_lib-needed-for-each-script](../scripts/devhub_lib-needed-for-each-script/)
{% endcontent-ref %}
