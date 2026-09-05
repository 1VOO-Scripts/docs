---
description: >-
  sh.car.lua, sh.motorcycle.lua, sh.truck.lua, sh.boat.lua, sh.helicopter.lua
  and sh.plane.lua — the actual course layouts.
---

# sh.\<class>.lua

Six files, one per class, all the same shape. Each writes into the two tables the shared files created:

```lua
Config.Courses.car      = { --[[ the obstacle course ]] }
Config.TrafficExams.car = { --[[ the road exam, where the class has one ]] }
```

Only car, motorcycle and truck have a `Config.TrafficExams` entry — boats and aircraft list two stages, so they never reach one.

{% hint style="danger" %}
**Load order matters.** These files write *into* tables that `sh.courses.lua` and `sh.traffic.lua` create, which is why `fxmanifest.lua` lists `shared_scripts` explicitly instead of globbing. If a class's exams go missing, check that ordering first.
{% endhint %}

***

## <mark style="color:yellow;">**The anchor — how a course moves**</mark>

Every course has a `center`, and **everything else in the file is an offset in that point's frame**: props, boundary, spawn and every step. Move or rotate the anchor and the whole layout follows, intact.

```lua
center = vector4(-837.75, -1285.49, 5.0, 21.62),
```

* **Description**: The course anchor. Its `z` sits roughly one metre **above** the tarmac, which is why every cone in the file is at `-1.0`.
* **Note**: This line is the **starting** position. Moving the course in the admin panel under <mark style="color:yellow;">Locations</mark> saves an override that wins over it; resetting that field comes back to this line.
* **Note**: A course anchor applies to the **next** run. A student already driving finishes on the tarmac they started on.

{% hint style="success" %}
**Surveying.** With `Config.Debug` on, an admin can run `/coursemark <kind> <school>` in game and it prints a line already converted into the anchor's frame, ready to paste. Without the school argument you get plain map coordinates instead.
{% endhint %}

***

## <mark style="color:yellow;">**Obstacle course fields**</mark>

```lua
Config.Courses.car = {
    vehicle = 'blista',
    center  = vector4(-837.75, -1285.49, 5.0, 21.62),
    spawn   = vector4(-4.302, -13.053, -0.180, 2.990),
    blip    = true,

    rules = { collision = true, stunt = true },

    boundary = {
        vector2(-14.179, -17.809),
        vector2(-15.914, 22.679),
        vector2(15.037, 21.793),
        vector2(16.018, -16.933),
    },

    props = {
        { model = 'prop_roadcone02b', offset = vector4(-5.937, -10.899, -0.995, 94.069) },
    },

    steps = { --[[ ... ]] },
}
```

* `vehicle` *(string)* — the model the school provides. It is spawned clean, repaired and refuelled for every run, so one student's dents never carry into the next student's collision baseline.
* `center` *(vector4)* — the anchor. See above.
* `spawn` *(vector4)* — where the school vehicle appears, as an offset.
* `blip` *(boolean)* — a map marker on the next objective, without the road route `gps` adds. A cone lot has no roads to follow.
* `rules` *(table)* — which always-on rules apply. Absent or `false` means not scored. A course with **no** `rules` table gets `collision` by default, which is why the car course lists it explicitly — adding a table to switch `stunt` on would otherwise have switched `collision` off with it.
* `boundary` *(table of vector2)* — leaving this polygon is an instant fail. Height is irrelevant to a point-in-polygon test, so only `x` and `y` are needed.
* `props` *(table)* — the scenery, as offsets. A `model` listed in `Config.CourseConeModels` spawns dynamic and can be knocked over and scored; anything else spawns frozen.
* `steps` *(table)* — the ordered objectives. See below.
* `tolerance` *(table, optional)* — overrides any value from `Config.CourseTolerance` for this course only.
* `population` *(boolean, optional)* — ambient traffic and pedestrians inside the student's private bucket.
* `surface` *(string, optional)* — `water` or `air`. Puts markers and props on the surface rather than on the seabed or the ground.
* `propFaults` *(table, optional)* — which fault keys this course's props score, e.g. `{ nudge = 'buoyNudged', strike = 'buoyStruck' }` on the boat course, so a channel marker is not reported as a cone.

***

## <mark style="color:yellow;">**Steps**</mark>

```lua
steps = {
    { type = 'checkpoint', label = 'Pull forward out of the garage',
      coords = vector3(-4.680, -7.493, 0.000), radius = 2.0 },

    { type = 'gate', label = 'Slalom - gate 1 of 3',
      coords = vector4(-7.070, -2.017, 0.000, 301.830), radius = 2.0, noReverse = true },

    { type = 'stop', label = 'Come to a full stop', hold = 1000,
      coords = vector3(-0.970, 16.498, 0.000), radius = 3.5 },

    { type = 'park', mode = 'bay', label = 'Reverse into the bay', reverse = true,
      target = vector4(5.884, 9.038, -0.180, 91.450) },
}
```

* `type` *(string)* — see the [step types table](sh.courses.lua.md).
* `label` *(string)* — the instruction shown on the HUD.
* `coords` *(vector3 or vector4)* — an offset. A `vector4` draws a chevron pointing along its heading instead of a disc; the heading is not scored.
* `radius` *(number)* — how big the zone is.
* `hold` *(ms)* — on a `stop` step, how long the vehicle must stand still.
* `target` *(vector4)* — on a `park` step, the vehicle parked perfectly. The tolerance box is derived from it.
* `reverse` *(boolean)* — the park must be entered backwards. Failing to reverse in scores `notReversed`.
* `mode` *(string)* — `bay`, `parallel` or `garage`. Cosmetic labelling for the manoeuvre.
* `noReverse` *(boolean)* — reversing on this step scores `reversing`. Used on slalom gates.
* `fault` *(string, optional)* — override which fault key this step scores when failed. The boat course uses it to turn a `stop` into `missedCasualty`, because there is no stop line at sea.

**Aircraft and water steps** add a few more types: `hover` (hold station inside a box), `orbit` (circle a point at a held radius, with `band` for the allowed radius error and `arc` for how far round), and `land`. The helicopter and plane files are commented step by step.

***

## <mark style="color:yellow;">**Road exam fields**</mark>

```lua
Config.TrafficExams.car = {
    vehicle      = 'blista',
    spawn        = vector4(-749.95, -1288.65, 4.06, 298.85),
    flatDistance = true,
    defaultLimit = { kph = 80, mph = 50 },
    gps          = true,

    rules = {
        speeding   = true,
        collision  = true,
        stunt      = true,
        pedestrian = true,
    },

    attributeCollisions = true,

    route = { --[[ ... ]] },
}
```

* `spawn` *(vector4)* — on the road outside the school, nose pointing at the first step. Road exams use **map coordinates**, not offsets — there is no anchor to move.
* `flatDistance` *(boolean)* — zone checks ignore height, so a `z` typed slightly off the map cannot make a step unreachable. Overpasses are the cost; turn it off on a route that has one.
* `defaultLimit` — the limit in force from the spawn until the first `limit` sign.
* `gps` *(boolean)* — draws a GPS line to the next step. A world marker only appears inside streaming range, which on a road is long after the student had to pick a lane.
* `rules` — which always-on rules apply. Road exams normally want all four.
* `attributeCollisions` *(boolean)* — only count body damage when the student was moving into something. Being rear-ended at a light is not a fault.
* `population` *(boolean)* — ambient traffic in the student's private bucket. **On** for road exams: with nothing on the roads there is nothing to yield to, and the collision rule would only score walls.
* `route` *(table)* — the ordered steps. `limit`, `stop`, `light` and `checkpoint`.

{% hint style="info" %}
The shipped routes are laid out around the Los Santos school. Replace them freely — nothing but the shape of the table is load-bearing.
{% endhint %}

***

## <mark style="color:yellow;">**Adding a new class**</mark>

{% stepper %}
{% step %}
### Add it to Config.Schools

Give it an `id`, `label`, `class` and `icon` in [sh.config.lua](sh.config.lua.md). Add a `stages` list if it should not have all three.
{% endstep %}

{% step %}
### Create shared/sh.\<id>.lua

Copy the closest existing class file and change `Config.Courses.<id>`.
{% endstep %}

{% step %}
### Add it to fxmanifest.lua

Append it to `shared_scripts`, **after** `sh.courses.lua` and `sh.traffic.lua`.
{% endstep %}

{% step %}
### Give it a licence name

Add an entry to `Config.License.templates`, and to `Config.LicenceCategories.groups` if it should share a card.
{% endstep %}

{% step %}
### Restart and check the console

The school reports at boot if a class lists a stage it has no data for, so a half-built class says so instead of failing at the moment a student presses start.
{% endstep %}
{% endstepper %}

Tuition, allowance and time limit for the new class appear in the admin panel automatically — they are generated from `Config.Schools`.
