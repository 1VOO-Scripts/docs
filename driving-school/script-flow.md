---
description: >-
  What the script actually does — from a student walking up to the instructor to
  a licence in their pocket.
---

# 🌊 Script Flow

A driving school with six licence classes. A student pays tuition at the instructor, sits a written theory paper, drives a private obstacle course, then takes an exam on public roads — and collects a licence at the end.

***

## <mark style="color:yellow;">**The classes**</mark>

| Class      | Licence     | Stages                       | Requires        |
| ---------- | ----------- | ---------------------------- | --------------- |
| Car        | Class B     | Theory → Obstacle → Traffic  | —               |
| Motorcycle | Class A     | Theory → Obstacle → Traffic  | —               |
| Truck      | Class C     | Theory → Obstacle → Traffic  | A full car licence |
| Boat       | Marine      | Theory → Obstacle            | —               |
| Helicopter | Rotorcraft  | Theory → Obstacle            | —               |
| Plane      | Fixed wing  | Theory → Obstacle            | —               |

Boats and aircraft have no traffic stage — an exam in traffic at sea or in the air would only be the obstacle course again.

Stages unlock in order: each one opens when the one before it is passed. Which stages a class asks for is configurable, and any of them can be switched off entirely from the [admin panel](admin-panel.md).

***

## <mark style="color:yellow;">**Getting a licence**</mark>

{% stepper %}
{% step %}
### Find the instructor

A ped stands at the school with a map blip. Interact with them to open the menu.

The menu lists every class, what it costs, which stages it has and how far the student has got. A class locked behind another says so.
{% endstep %}

{% step %}
### Enrol and pay

Pick a class and choose cash or bank. Tuition is charged once, on enrolment.

Everything the menu showed is re-checked on the server before a penny is taken — the class exists, its prerequisite is met, it is not already paid for, and the chosen account actually covers it.
{% endstep %}

{% step %}
### Sit the theory paper

A multiple-choice paper drawn at random from that class's question bank, on a countdown.

* The questions arrive **without** the answers. Correct answers never leave the server.
* Starting the exam twice returns the **same** paper with the time remaining, rather than dealing a fresh one — so it cannot be re-rolled until a memorised set comes up.
* The clock is the server's. The paper auto-submits at zero.
* Walking out is recorded, and carries the same cooldown as failing.

Passing shows the score and unlocks the next stage. Failing shows which questions were missed, and starts a retry cooldown.
{% endstep %}

{% step %}
### Drive the obstacle course

The school provides the vehicle — spawned clean, repaired and refuelled, so one student's dents never carry into the next student's marking.

The run happens in the student's **own private world**, which means nobody can drive through their cones, two students can sit the test at the same place at the same time, and the lot stays free of ambient traffic.

A HUD shows the current objective, the fault counter against the allowance, and the time remaining. Faults flash up as they are scored, each one named.

The course is a sequence of objectives — drive through gates, come to a full stop, reverse into a bay, parallel park. Leaving the marked boundary, leaving the vehicle, running out of time or dying ends the run immediately.
{% endstep %}

{% step %}
### Take the exam in traffic

Only for car, motorcycle and truck. The same runner, driven on public roads with ambient traffic switched on.

The route is a sequence of speed-limit signs, stop signs, controlled junctions and checkpoints, with a GPS line to the next one. On top of that, four rules apply anywhere on the route: speeding, collisions, hitting a pedestrian, and losing control of the vehicle.

The exam **controls its own traffic lights**, so a red light it shows you is a red it knows about — and a junction it could not find is never scored.
{% endstep %}

{% step %}
### Collect the licence

Pass the last stage a class asks for and the licence is earned. By default the card is **collected at the desk** rather than appearing at the end of the course — the end of a run, sealed in a private world and sitting in a vehicle, is the worst possible moment to be photographed for an ID card.

The menu shows a **Claim** button on any finished class, and the result card offers it directly.
{% endstep %}
{% endstepper %}

***

## <mark style="color:yellow;">**Three cards, six classes**</mark>

Classes share a card:

| Card          | Classes                 |
| ------------- | ----------------------- |
| Driving Licence — ABC | Car, Motorcycle, Truck |
| Boat Licence  | Boat                    |
| Pilot Licence | Plane, Helicopter       |

Pass a second class in a group and there is no new card to print — the class is written **onto the card already in the player's pocket**, and its Categories line is brought up to date. A student who passes car and then motorcycle carries one licence that lists both.

{% hint style="info" %}
That rewriting is a devhub\_licenses feature. On ESX, QB/QBOX or vRP the licence is a name and a yes, with nothing on it to bring up to date — passing the second class simply confirms the entitlement they already had.
{% endhint %}

***

## <mark style="color:yellow;">**What the result card offers**</mark>

Every run ends on a card that says what happened and what to do next:

* **Failed** → retry, with the cooldown counting down on the button itself.
* **Passed, more stages left** → start the next stage.
* **Passed the last stage** → collect the licence.

Pressing any of them goes through exactly the same checks the menu buttons do — a student who has wandered off, hit a cooldown or died on the way is refused there and told why.

***

## <mark style="color:yellow;">**How marking is decided**</mark>

Driving is physics on the player's machine, so the client reports **which faults happened** and the server decides what they are worth — re-adding the points itself from its own config, applying a plausibility check on how long the run took, and recording every run either way.

A run that could not have been driven is stored as **unverified** rather than dropped, so a pattern of them is visible afterwards. The student is told their run could not be verified.

Everything else is decided on the server outright: enrolment, tuition, the theory grade, stage unlocking and the licence itself.

***

## <mark style="color:yellow;">**Records and retention**</mark>

Three things are stored: a progress row per character per class, a log of theory attempts, and a log of practical runs. The two attempt logs are what retry cooldowns are read from, and they double as the audit trail for the stages the server cannot referee itself.

They are pruned automatically — 30 days by default, set by `Config.AttemptRetentionDays`. Progress is never pruned.

***

## <mark style="color:yellow;">**Related pages**</mark>

{% content-ref url="admin-panel.md" %}
[admin-panel.md](admin-panel.md)
{% endcontent-ref %}

{% content-ref url="configuration/README.md" %}
[README.md](configuration/README.md)
{% endcontent-ref %}

{% content-ref url="exports-and-events.md" %}
[exports-and-events.md](exports-and-events.md)
{% endcontent-ref %}
