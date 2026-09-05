---
description: Every line of text the script can say, in one file, shared by the Lua and the interface.
---

# sh.lang.lua

Around 260 lines of text — everything the player, the HUD and the admin panel can show. Change the value on the **right** of each `=`; leave the key on the left alone, because that is what the code asks for.

```lua
Config.Lang = {
    ['ped_target_label']  = "Start Driving School",
    ['blip_label']        = "Driving School",
    ['err_retry_in']      = "You can try again in %{seconds} seconds.",
}
```

***

## <mark style="color:yellow;">**One table, both sides**</mark>

The Lua reads this table directly. The interface asks for it **once as it loads**, so a translated line reaches the menu, the HUD and the admin panel without rebuilding anything.

{% hint style="success" %}
That matters: only the built interface ships, so there is no way for you to rebuild it. Translating this file is the supported way to change any text in the UI.
{% endhint %}

***

## <mark style="color:yellow;">**Placeholders**</mark>

```lua
['err_retry_in']       = "You can try again in %{seconds} seconds.",
['result_stage_failed'] = "You failed the %{stage} with %{points} fault points.",
```

* **Description**: `%{name}` markers are filled in at runtime — `%{school}` becomes `Car`, `%{seconds}` becomes `30`.
* **Rule**: Keep every marker exactly as it is spelled. One that is renamed or deleted leaves the sentence with a gap in it.
* **Word order is yours.** Each marker is filled **by name**, not by position, so both of these work:

```lua
"%{points} fault points on the %{stage}"
"on the %{stage}: %{points} fault points"
```

That is what makes the file translatable into a language whose grammar puts things in a different order.

***

## <mark style="color:yellow;">**How the file is grouped**</mark>

| Section              | What is in it                                                          |
| -------------------- | ---------------------------------------------------------------------- |
| In the world         | The ped interaction label, the map blip, the admin menu button.         |
| Menu                 | The school menu — headings, buttons, the instructor's welcome.          |
| Errors and refusals  | Every reason the school can turn a student away.                        |
| Stages               | The names of `theory`, `obstacle` and `traffic` inside a sentence.      |
| Results              | Pass, fail and unverified verdicts.                                     |
| Licence              | Issued, added to an existing card, collect at the desk, print failed.   |
| Theory exam          | The paper, the timer, the result breakdown.                             |
| HUD                  | The live course display — the objective, the fault counter, the timer.  |
| Admin panel          | Group names, field labels and the one-line help under each field. All the `tun_` keys. |

***

## <mark style="color:yellow;">**A missing key is visible, not fatal**</mark>

Ask for a key that is not in the table and you get `Translation for <key> not found` back rather than an error. If that string appears in game, the key was renamed or deleted — put it back.

{% hint style="warning" %}
Stage names are written for the **middle** of a sentence. The script raises the first letter itself where one starts a sentence, so write `obstacle course`, not `Obstacle course`.
{% endhint %}
