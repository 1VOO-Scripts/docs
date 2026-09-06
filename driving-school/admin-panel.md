---
description: >-
  Where almost all the tuning happens — live, validated, and without touching a
  file or restarting the server.
---

# 🎛️ Admin Panel

Open it in game with:

```
/admindevhub  →  Driving School
```

The button only appears for players your framework treats as admins, and the server re-checks that on every open and every save — a hidden button is not a permission check.

{% hint style="success" %}
**Nothing is in force until you press Save.** The panel stages your edits and sends them as one batch, so a half-finished thought about the truck's allowance is never marking somebody's exam.
{% endhint %}

***

## <mark style="color:yellow;">**How it behaves**</mark>

* **Applies live.** Saving reaches every connected player immediately. No restart, no reconnect.
* **Runs in progress are not disturbed.** A run is marked under the rules it began with, so changing an allowance mid-exam applies to the next run rather than the one being driven. The same is true of a course anchor.
* **Only changed values are stored.** A setting you never touch keeps following the shipped default, so a later version with a better default still reaches you.
* **Reset really resets.** Putting a field back to its default **deletes** the stored row rather than saving today's default over it — otherwise the field would be silently pinned to an old number forever.
* **Everything is validated on the server.** Values are clamped to their allowed range whatever the interface sent, and what comes back is the value that actually landed.
* **Every save is logged** to the server console with the admin's identifier and each setting changed.

***

## <mark style="color:yellow;">**Scoring**</mark>

Fault allowances and the sanity checks on a reported run.

| Setting                | What it does                                                                                              |
| ---------------------- | --------------------------------------------------------------------------------------------------------- |
| Obstacle course allowance | Fault points that fail the run. Reaching it fails, so `12` allows 11. A class with its own allowance ignores this. |
| Traffic exam allowance | The same, for the drive in traffic. A red light costs 6 of it by default, a stop sign 4.                    |
| Retry cooldown         | Wait after a failed or abandoned run. The result card counts it down.                                       |
| Minimum run length     | A run reported faster than this cannot have been driven, and awards nothing.                                |
| Late submission grace  | Slack over the course time limit before a result is thrown out.                                             |
| Speed units            | Which of each sign's two numbers is posted. Ships as **mph**; changes the whole route at once.              |
| Speed sign style       | `Auto` picks US for mph and EU for km/h. Set EU explicitly for the UK.                                      |

***

## <mark style="color:yellow;">**Tolerances**</mark>

How exact a park has to be, and what counts as a hit.

| Setting                       | What it does                                                                     |
| ----------------------------- | -------------------------------------------------------------------------------- |
| Clean park — side to side     | Inside this of the perfect spot, the park scores nothing.                          |
| Clean park — fore and aft     | The same, along the length of the bay.                                             |
| Clean park — angle            | Degrees off square that still counts as straight.                                  |
| Sloppy park — side to side    | Outside the clean figure but inside this scores an inaccurate park. Beyond it, the bay is missed. |
| Sloppy park — fore and aft    | The same, lengthways.                                                              |
| Sloppy park — angle           | Degrees off square that still counts as parked at all.                             |
| Maximum shunts                | Back-and-forth corrections allowed before it scores excessive shunting.            |
| Collision threshold           | Body health lost in one hit before it counts as a collision. Lower is stricter.    |

***

## <mark style="color:yellow;">**Fault points**</mark>

What each mistake costs, one row per fault. The list is generated from `Config.CourseFaults`, so a fault you add to [sh.courses.lua](configuration/sh.courses.lua.md) appears here automatically.

{% hint style="info" %}
Faults that end a run outright are **not** listed. Whether a mistake ends a run is a design decision, not a number — set `major = true` in the config file for that.
{% endhint %}

***

## <mark style="color:yellow;">**Theory exam**</mark>

The written paper at the desk. <mark style="color:red;">Server side only — none of this reaches a client.</mark>

| Setting               | What it does                                                                                 |
| --------------------- | -------------------------------------------------------------------------------------------- |
| Questions per attempt | Drawn at random from the class bank. Keep it below the bank size or every paper is the same.   |
| Pass mark             | Share of answers that must be right. A class can set its own, under **Classes**.               |
| Time allowed          | The paper auto-submits at zero. The server keeps its own clock.                                |
| Submission grace      | How late an answer may arrive before it counts as a timeout. Covers latency, not tampering.    |
| Retry cooldown        | Wait after a failed paper. Stops a small bank being brute-forced. `0` disables it.             |

***

## <mark style="color:yellow;">**Licence**</mark>

What the school hands over when every stage is passed.

| Setting                   | What it does                                                                                         |
| ------------------------- | ------------------------------------------------------------------------------------------------------ |
| Issue licences            | Off, and the school still teaches and still records progress. It just does not end in a card.            |
| Approve without printing  | Grant the entitlement and hand over nothing — the card is collected from a clerk. <mark style="color:red;">devhub\_licenses only.</mark> |
| Licence validity          | How many days a card is valid for. `0` uses the template's own setting.                                  |

***

## <mark style="color:yellow;">**Locations**</mark>

Who the instructor is, where they stand, and where each cone course is laid out.

| Setting             | What it does                                                                                             |
| ------------------- | ---------------------------------------------------------------------------------------------------------- |
| Instructor          | Any ped model in the game. Checked as you type — a name GTA does not have would leave the school with nobody to talk to. |
| Instructor position | Where they stand.                                                                                            |
| *(class)* course    | Moves and turns the **whole** layout. The heading is which way the course faces, so a lot that runs the other way needs only this row. |

{% hint style="warning" %}
**Use my position.** Walk to the spot first, *then* open the panel and press it — the panel takes focus, so you cannot move once it is open.
{% endhint %}

The instructor changes the moment you save. A course anchor applies to the **next** run, not to one being driven.

***

## <mark style="color:yellow;">**Classes**</mark>

Generated per class from `Config.Schools`, so a class you add appears here on its own.

| Setting                          | What it does                                                                          |
| -------------------------------- | --------------------------------------------------------------------------------------- |
| *(class)* — *(exam)*             | Switch an exam off. It disappears from the menu, from the gating, and from what the licence requires. |
| *(class)* — theory pass mark     | Share of answers this class needs right. `0` uses the shared pass mark.                    |
| *(class)* — tuition              | Charged once, on enrolment.                                                               |
| *(class)* — *(stage)* allowance  | Fault points that fail that stage. `0` uses the shared allowance.                          |
| *(class)* — *(stage)* time limit | Seconds for the whole run. Running out is an instant fail.                                 |

{% hint style="danger" %}
**One exam has to stay on.** A class with nothing to pass would be bought and instantly complete, so the panel refuses the save that would switch off the last one — and tells you which class it refused, without losing the other edits in the same batch.
{% endhint %}

***

## <mark style="color:yellow;">**Related pages**</mark>

{% content-ref url="configuration/README.md" %}
[README.md](configuration/README.md)
{% endcontent-ref %}

{% content-ref url="script-flow.md" %}
[script-flow.md](script-flow.md)
{% endcontent-ref %}
