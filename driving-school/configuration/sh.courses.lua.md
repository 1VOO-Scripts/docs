---
description: >-
  What every obstacle course shares — the fault list, the markers, and the
  measurements behind a verdict.
---

# sh.courses.lua

This file creates `Config.Courses` and holds everything the six courses have in common. The courses themselves are in the [class files](class-courses.md).

{% hint style="info" %}
What a fault **costs**, and how exact a park has to be, are in the admin panel under <mark style="color:yellow;">Fault points</mark> and <mark style="color:yellow;">Tolerances</mark>. This file decides which faults **exist** and how they are measured.
{% endhint %}

***

## <mark style="color:yellow;">**Step types**</mark>

A course is an ordered list of typed steps. These are the four types:

| Type         | What the student does                | Fields                        |
| ------------ | ------------------------------------ | ----------------------------- |
| `checkpoint` | Drive through a sphere               | `coords`, `radius`            |
| `gate`       | The same, labelled as a slalom       | `coords`, `radius`            |
| `stop`       | Come to a standstill inside a sphere | `coords`, `radius`, `hold`    |
| `park`       | Finish inside a tolerance box        | `target` (vector4), `reverse` |

A `vector4` `coords` marks the step with a chevron pointing along its heading instead of a disc — visual only, the heading is not scored. `target` on a park step is the vehicle parked perfectly; the tolerance box is derived from it.

***

## <mark style="color:yellow;">**Config.CourseGroundSnap**</mark>

```lua
Config.CourseGroundSnap = true
```

* **Description**: Drops props onto the ground beneath them instead of trusting the stored height offset. A single prop opts out with `snap = false`.
* **Example**: Leave it on unless a course sits on a surface the ground probe misreads, such as a pier deck.

***

## <mark style="color:yellow;">**Config.CourseConeModels**</mark>

```lua
Config.CourseConeModels = {
    ['prop_roadcone01a'] = true,
    ['prop_mp_cone_01']  = true,
    ['prop_dock_bouy_3'] = true,
}
```

* **Description**: Props that spawn **unfrozen**, so being knocked can be measured and scored. Anything not listed spawns frozen and is scored through body damage instead.
* **Note**: On a `surface = 'water'` course, a listed prop is also held on the surface instead of sinking.
* **Example**: Add your own cone or buoy model here to have it scored the same way.

***

## <mark style="color:yellow;">**Config.CourseFaults**</mark>

```lua
Config.CourseFaults = {
    cone          = { label = 'Knocked a cone' },
    ranRedLight   = { label = 'Ran a red light' },
    outOfBounds   = { major = true, label = 'Left the course' },
}
```

* **Description**: Every mistake the school can score, for **both** practical exams. A minor fault adds points; reaching the allowance fails the run. A `major` fault ends it immediately and carries no points.
* **Fields**:
  * `label` *(string)* — what the student sees on the HUD and in the result.
  * `major` *(boolean, optional)* — ends the run outright.
* **Note**: What each minor fault **costs** is in the admin panel. Majors are not listed there — whether a mistake ends a run is a design decision, not a number.
* **Example**: A real test fails you for running a red. To match that, add `major = true` to `ranRedLight` and remove its points in the panel.

**The shipped faults**

| Group           | Keys                                                                                |
| --------------- | ----------------------------------------------------------------------------------- |
| Obstacle course | `cone`, `coneFlattened`, `collision`, `noStop`, `notReversed`, `sloppyPark`, `shunting`, `stalled` |
| Rotary wing     | `hoverLost`, `slowStop`, `hardLanding`, `notLevel`                                   |
| Fixed wing      | `steepBank`, `reversing`                                                             |
| Water           | `buoyNudged`, `buoyStruck`, `missedStop`, `missedCasualty`                            |
| Traffic exam    | `speeding`, `ranStopSign`, `ranRedLight`, `stunt`                                    |
| Major           | `outOfBounds`, `leftVehicle`, `timeout`, `hitPedestrian`, `died`                     |

{% hint style="warning" %}
Every fault is one unambiguous reading — a number against a posted limit, a measured displacement, a contact. Nothing that only *feels* wrong is scored, because a fault a student can argue with is worse than one that goes unscored. Keep that rule if you add your own.
{% endhint %}

***

## <mark style="color:yellow;">**Config.CourseTolerance**</mark>

```lua
Config.CourseTolerance = {
    stationarySpeed = 0.30, -- m/s counted as "stopped"
    parkSettleTime  = 1500, -- ms held still before a park is judged
    reverseSpeed    = 0.50, -- backwards m/s counted as "reversing"

    coneNudge       = 0.20, -- m moved = clipped it
    coneFlatten     = 0.80, -- m moved = ran it over
    coneTipAngle    = 40.0, -- degrees of pitch/roll = knocked flat

    hoverStill      = 2.50, -- m/s that still counts as holding station
    hoverBand       = 3.00, -- m of height either side of a hover box
    orbitBand       = 12.00,-- m of radius error while circling a point

    bankLimit       = 70.0, -- degrees of roll before it is being thrown about
    hardLanding     = 4.00, -- m/s at contact
    levelTolerance  = 12.0, -- degrees of roll or pitch on the skids
}
```

* **Description**: The measurements behind each fault — how often things are checked and how far something must move to count. A per-course `tolerance = {}` block overrides any of these for that course only.
* **Note**: The values that decide how exact a **park** is, and what counts as a **hit**, are deliberately not here — they are in the admin panel under <mark style="color:yellow;">Tolerances</mark>, because they are the ones you retune while watching a student drive.
* **Groups**:
  * **Movement** — `stationarySpeed`, `parkSettleTime`, `reverseSpeed`.
  * **Floating props** — `floatSync`, `floatRadius`, `floatSlack`, `floatWaves`, `floatDraft`. Only a rescue for a prop the engine fails to float; normally they do nothing. Widen `floatSlack` if buoys twitch, narrow it if one sinks.
  * **Cones** — `coneSettle`, `conePoll`, `coneNudge`, `coneFlatten`, `coneTipAngle`.
  * **Rotary wing** — `hoverStill`, `hoverBand`, `hoverHold`, `orbitBand`, `quickStopWindow`, `landingWindow`, `hardLanding`, `levelTolerance`.
  * **Roll** — `bankLimit` and `bankRelease`. `bankRelease` is hysteresis, so one long steep turn is one fault rather than one per frame.
  * **Reverse** — `reverseCreep`, `reverseAllowed`, `reverseRoll`. Two allowances, because rolling back and choosing reverse are not the same thing.

***

## <mark style="color:yellow;">**Markers**</mark>

Four tables decide how a course is drawn. All of them take `r`, `g`, `b`, `a` colour values.

```lua
Config.CourseMarker = { type = 2, scale = 0.65, maxSize = 3.0, height = 0.30 }
Config.CourseCircle = { type = 1, scale = 0.8,  maxSize = 4.5, thickness = 0.45 }
Config.CourseOrbit  = { leadArc = 70.0, lead = 8, sizeScale = 0.0245, type = 28 }
Config.CourseBox    = { nose = 0.9, noseTilt = 180.0, noseYaw = 0.0 }
```

* **`Config.CourseMarker`** — the chevron drawn on steps whose coords carry a heading. `scale` multiplies the step radius, `maxSize` caps it so a wide traffic zone does not paint the road.
* **`Config.CourseCircle`** — the disc drawn on steps with no heading. `thickness` makes it a low column, because a flat disc disappears at speed.
* **`Config.CourseOrbit`** — the path flown **around** a point. A full ring is a hairline at distance, so what you actually fly is a run of markers on the line ahead: `leadArc` degrees of it, `lead` markers deep. The gold ones count laps, the red ones mean you are outside the band.
* **`Config.CourseBox`** — the outline around a parking bay or landing pad, plus the oversized arrow that stands above it pointing down.
* **`Config.MarkerPlanes`** — which way a marker lies: `vertical` is a hoop on its edge you fly **through**, `horizontal` lies flat. Set `plane` on a step, or on a course's `marker` / `circle` for all of them; the step wins.

{% hint style="info" %}
The aim values are `scale * stepHeading + offset` in degrees, and were established in game with the `/gateaim` dev command. If a chevron points the wrong way after you change one, that command is how to get it back.
{% endhint %}
