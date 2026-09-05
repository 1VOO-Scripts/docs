---
description: >-
  The integration contract — one server export, and three open functions the
  school calls at the moments worth reacting to.
---

# ⚙️ Exports & Events

| Surface               | Where it lives              | What it is                                          |
| --------------------- | --------------------------- | --------------------------------------------------- |
| Client exports        | —                           | None. The client decides nothing.                   |
| Server exports        | Callable from any resource  | `getCategories` — which classes a player has earned. |
| Client open functions | —                           | None.                                               |
| Server open functions | `server/s.hooks.lua`        | `CanStartExam`, `OnExamFinished`, `OnLicenseIssued`  |

{% hint style="warning" %}
There are deliberately **no client exports**. Everything the school decides — enrolment, tuition, grading, stage unlocking, the licence — is decided on the server, and the client only paints what it is sent. Anything you might want to ask a client, ask the server instead.
{% endhint %}

***

## <mark style="color:yellow;">**Server Exports**</mark>

| Export          | Returns            | Purpose                                                |
| --------------- | ------------------ | ------------------------------------------------------ |
| `getCategories` | `string, table`    | Which classes in a group this player has actually earned. |

### getCategories

```lua
local line, list = exports['devhub_drivingschool']:getCategories(source, group)
```

* **What it does**: Answers which classes inside one card's group a player has **finished** — every stage that licence asks for, passed. It returns both a ready-made line for printing and the list it was built from.
* **When to use**: Anywhere you want to print or check what a licence actually covers — a card's Categories field, an MDT lookup, a police check, a job that only accepts truck drivers.
* **Parameters**:
  * `source` *(number, required)* — the player's server ID. If that player is not loaded, you get the empty line and an empty list.
  * `group` *(string, required)* — a group name from `Config.LicenceCategories.groups` (`"car"`, `"boat"`, `"plane"`), **or** any class `id`, which resolves to the group containing it. So `"motorcycle"` and `"car"` both answer about the ABC card.
* **Returns**: `string, table`
  * The **line** — the earned classes joined by `Config.LicenceCategories.separator`, e.g. `"Car, Motorcycle"`. A group with nothing earned in it returns `Config.LicenceCategories.empty` (an empty string by default).
  * The **list** — the same classes as a plain array, `{ "Car", "Motorcycle" }`, so you can lay them out yourself rather than parsing the sentence back apart. Empty table when nothing is earned.
  * A group name that does not exist, a player who is not loaded, or the school still starting up all answer the empty line and an empty table — never `nil`, never an error.

{% hint style="info" %}
**Earned, not held.** This asks whether the course is finished, not whether the card is in their pocket. That is deliberate — it is called *while a card is being printed*, so asking about the card would answer about the previous one.
{% endhint %}

```lua
-- Print the classes on an ID card
local line = exports['devhub_drivingschool']:getCategories(source, 'car')
-- "Car, Truck"

-- Gate a trucking job on the class rather than the card
local _, classes = exports['devhub_drivingschool']:getCategories(source, 'car')
for _, name in ipairs(classes) do
    if name == 'Truck' then return true end
end
return false
```

The names come back as they are written in `Config.LicenceCategories.names`, falling back to each class's `label`. If you renamed them to `A` / `B` / `C`, compare against those.

***

## <mark style="color:yellow;">**Server Open Functions**</mark>

Three functions in `server/s.hooks.lua`. They ship doing nothing — fill them in, leave them, or **delete the file entirely**; the school behaves the same either way.

{% hint style="success" %}
This file is **not sealed**, so it stays yours across an update. Keep your own rules in here rather than in the files around it, which are sealed.
{% endhint %}

| Hook              | Fires when                                            | Blocking |
| ----------------- | ----------------------------------------------------- | -------- |
| `CanStartExam`    | A student is about to be handed any exam               | ✅        |
| `OnExamFinished`  | An exam has been graded                                | ❌        |
| `OnLicenseIssued` | The player has just gained a licence                   | ❌        |

**Shared parameters**

* `source` *(number)* — the player's server ID.
* `school` *(string)* — a class id: `car`, `motorcycle`, `truck`, `boat`, `helicopter`, `plane`.
* `exam` *(string)* — `theory`, `obstacle` or `traffic`.

{% hint style="danger" %}
A mistake in this file is named in the server console rather than taken out on the player — the school carries on as though the function had not been written. That means a broken hook is **not** a closed school, but it is also not a working rule. Watch the console after editing.
{% endhint %}

***

### CanStartExam

```lua
function CanStartExam(source, school, exam)
    return true
end
```

* **Fires when**: A student presses start on any exam, theory or practical.
* **Blocking**: ✅ — **only `false` refuses.** Anything else, including forgetting to return at all, lets the exam start. A mistake here should not close the school.
* **Parameters**: `source`, `school`, `exam`.
* **Returns**: `boolean, string|nil` — return `false` to refuse, optionally with a reason **the player is shown**. Without a reason they get a general refusal, which leaves them guessing at a button that appeared to do nothing.
* **Note**: Every rule the school has of its own has already passed by the time this is asked — enrolled, tuition paid, the previous stage cleared, off cooldown, standing at the school. This is only for the condition the school has no way of knowing about.

```lua
function CanStartExam(source, school, exam)
    if exam == 'traffic' and not Core.HasLicense(source, 'insurance_certificate') then
        return false, 'The examiner wants to see your insurance first.'
    end

    -- Weekends only, for the aircraft classes
    if school == 'plane' or school == 'helicopter' then
        local day = tonumber(os.date('%w'))
        if day ~= 0 and day ~= 6 then
            return false, 'Flight examinations run at weekends only.'
        end
    end

    return true
end
```

***

### OnExamFinished

```lua
function OnExamFinished(source, school, exam, passed)
end
```

* **Fires when**: An exam has been graded, with the verdict that went into the record.
* **Blocking**: ❌ — the return value is ignored.
* **Parameters**: `source`, `school`, `exam`, plus:
  * `passed` *(boolean)* — `true` only where it counted as a pass.
* **Note**: Driving away from a course arrives here as a **fail**. A paper walked out of, or a run whose player disconnected, is logged as abandoned but never graded — so it does not arrive here at all.
* **Note**: The stage, and any licence it completes, are written **straight after** this. If what you want to know is whether that was the last stage, wait to be told by `OnLicenseIssued` rather than reading the progress table from here.

```lua
function OnExamFinished(source, school, exam, passed)
    exports['my_stats']:record(source, 'drivingschool', {
        school = school, exam = exam, passed = passed,
    })

    if not passed and exam == 'traffic' then
        exports['my_dispatch']:notify('dmv', 'A road test just ended badly.')
    end
end
```

***

### OnLicenseIssued

```lua
function OnLicenseIssued(source, school)
end
```

* **Fires when**: The player has just gained the licence for `school` — whether that printed them a card or wrote the class onto one they already carry. Both are a licence they did not hold a moment ago.
* **Blocking**: ❌ — the return value is ignored.
* **Parameters**: `source`, `school`.
* **Note**: Collecting a card at the desk arrives here too. Earning a licence and being handed it are not the same moment, and this is the second one.
* **Note**: It fires **once** per licence. The two ways a licence can arrive — a fresh card, or a class added to an existing one — are mutually exclusive, so there is no double call to guard against.

```lua
function OnLicenseIssued(source, school)
    Core.Notify(source, 'Drive safely.', 5000, 'success')

    if school == 'truck' then
        exports['my_jobs']:unlock(source, 'trucker')
    end
end
```

***

## <mark style="color:yellow;">**Integration recipes**</mark>

### Only let insured drivers sit the road test

```lua
-- server/s.hooks.lua
function CanStartExam(source, school, exam)
    if exam ~= 'traffic' then return true end

    local hasIt = exports['devhub_licenses']:playerHasLicense(source, 'insurance_certificate')
    if not hasIt then
        return false, 'You need valid insurance before the road test.'
    end

    return true
end
```

### Refuse a truck licence to anyone without a clean record

```lua
-- server/s.hooks.lua
function CanStartExam(source, school, exam)
    if school ~= 'truck' then return true end

    local points = exports['my_dmv']:penaltyPoints(source)
    if points >= 6 then
        return false, ('Too many penalty points (%d). Clear them first.'):format(points)
    end

    return true
end
```

### Print the earned classes on your own ID card

```lua
-- anywhere server side
local line = exports['devhub_drivingschool']:getCategories(source, 'car')
myCard.categories = line ~= '' and line or 'None'
```

### Keep an external DMV database in step

```lua
-- server/s.hooks.lua
function OnLicenseIssued(source, school)
    local identifier = Core.GetIdentifier(source)
    local _, classes = exports['devhub_drivingschool']:getCategories(source, school)

    exports['my_dmv']:sync(identifier, {
        school  = school,
        classes = classes,
        issued  = os.time(),
    })
end
```

***

## <mark style="color:yellow;">**Related pages**</mark>

{% content-ref url="configuration/sh.config.lua.md" %}
[sh.config.lua.md](configuration/sh.config.lua.md)
{% endcontent-ref %}

{% content-ref url="script-flow.md" %}
[script-flow.md](script-flow.md)
{% endcontent-ref %}
