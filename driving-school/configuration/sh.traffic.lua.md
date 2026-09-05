---
description: >-
  What every road exam shares — the traffic lights the exam controls and the
  rules that are true anywhere on a route.
---

# sh.traffic.lua

This file creates `Config.TrafficExams` and holds what every road exam has in common. The routes themselves are in the [class files](class-courses.md) — car, motorcycle and truck each have one.

***

## <mark style="color:yellow;">**How a road exam is built**</mark>

Each exam has two lists:

* **`route`** — ordered steps, driven in sequence.
* **`rules`** — always-on, true anywhere on the route. A collision has no location.

**Step types**

| Type         | What it is                                                              |
| ------------ | ----------------------------------------------------------------------- |
| `limit`      | A sign. Sets the posted limit from there until the next one.            |
| `stop`       | Come to a standstill inside the zone — a stop sign.                     |
| `light`      | A junction the exam controls.                                            |
| `checkpoint` | Drive through it. Nothing to fail.                                       |

***

## <mark style="color:yellow;">**Config.TrafficSpeed**</mark>

```lua
Config.TrafficSpeed = {}
```

* **Description**: Filled from the admin panel. A speed anywhere in a route is written `{ kph = 50, mph = 30 }` or as a plain number — **per unit rather than converted**, because 50 km/h converts to 31 and no road posts 31.
* **Fields** *(set in the panel under* <mark style="color:yellow;">Scoring</mark>*)*:
  * `units` — `kph` or `mph`. Which number is read from a step and shown on the HUD.
  * `sign` — the sign style. `auto` picks from the unit; use `eu` for the UK, which is the one case `auto` gets wrong.
* **Note**: `{limit}` in a step's label is replaced with the number actually posted.

***

## <mark style="color:yellow;">**Config.TrafficLights**</mark>

```lua
Config.TrafficLights = {
    redChance    = 0.4,
    radius       = 40.0,
    headRadius   = 5.0,
    refresh      = 500,
    releaseAfter = 5000,
    green = 0, red = 1, amber = 2,
    holdColour = 'red',
    reset = -1,
    models = { 'prop_traffic_01a', 'prop_traffic_01b', --[[ ... ]] },
}
```

* **Description**: A traffic light's state cannot be read from script, so the exam **owns** them — a `light` step forces the colour and therefore knows it. A junction whose prop it never found is not scored: a red nobody was shown is not a red they ran.
* **Fields**:
  * `redChance` *(0.0–1.0)* — how often a junction rolls red, once per light step per run. `1.0` pins it red, `0.0` pins it green. A step can carry its own to override this.
  * `radius` *(number)* — fallback sweep radius, used only when a `light` step lists no heads. Wide enough to reach the junction from the stop line — which on a short block can also reach the next one, so listing heads is always better.
  * `headRadius` *(number)* — search radius around each listed head. Keep it tight, or two points resolve to the same head.
  * `refresh` *(ms)* — the override is re-applied rather than set once, because the junction is often not streamed in when the step becomes current.
  * `releaseAfter` *(ms)* — how long a junction stays green after the **next** objective is completed. Measured from there so it never turns red in the student's mirror.
  * `green` / `red` / `amber` *(number)* — the override state values. <mark style="color:red;">These were measured in game; published lists disagree and are wrong about red.</mark> Test before changing.
  * `holdColour` — what a held junction shows: `red`, `amber` or `green`. Amber proves an override is landing, since no junction sits on amber by itself — useful when checking a new route.
  * `reset` *(number)* — puts the junction back on the game's own cycle. If one ever sticks after a run, try `5`.
  * `models` *(table)* — traffic light prop names. One object is found **per model per search point**, so a four-head junction needs four listed points. An unknown name costs nothing; a missing one is a junction the exam cannot touch.

{% hint style="warning" %}
**Known limitation.** The game's own AI ignores these overrides, so ambient traffic will drive through a red the exam is holding. There is nothing a script can do about it.
{% endhint %}

***

## <mark style="color:yellow;">**Config.TrafficRules**</mark>

```lua
Config.TrafficRules = {
    speedTolerance = { kph = 5, mph = 3 },
    speedRearm     = 10000,

    collisionSpeed = 2.0,

    pedScan   = 250,
    pedRadius = 8.0,

    stuntAirtime     = 700,
    stuntBurnout     = 900,
    stuntBurnoutSlow = 0.6,
    stuntSlip        = 3.0,
    stuntSpeed       = 5.0,
    stuntRearm       = 5000,

    settleTime = 0,
}
```

* **Description**: The always-on rules, true anywhere on any route. Each is one unambiguous reading.
* **Speed**:
  * `speedTolerance` — how far over the sign before it counts, per unit.
  * `speedRearm` *(ms)* — before the same offence can score again. Without it a motorway scores a fault every frame.
* **Collision**:
  * `collisionSpeed` *(m/s)* — how fast the student was going **along their own axis** for an impact to be theirs rather than something hitting them.
* **Pedestrians**:
  * `pedScan` *(ms)* / `pedRadius` *(m)* — how often the nearby ped pool is checked, and how far around the vehicle is worth checking.
* **Stunts** — three readings of "not being driven, being thrown around":
  * `stuntAirtime` *(ms)* off the ground = it was jumped.
  * `stuntBurnout` *(ms)* and `stuntBurnoutSlow` *(m/s)* — holding throttle and brake together reads as a burnout for as long as it is held, so what separates a burnout from braking is that braking sheds speed. `stuntBurnoutSlow` is how much speed it may lose in that window before the clock starts over. Lower is stricter.
  * `stuntSlip` *(m/s sideways)* = sliding, not steering. `stuntSpeed` is the floor below which slip is just parking.
  * `stuntRearm` *(ms)* — same reasoning as `speedRearm`.
* **Start**:
  * `settleTime` *(ms)* — the car is held still at the start while ambient population builds up in the fresh routing bucket. Raise it if the first stretch of your route feels like a ghost town.
