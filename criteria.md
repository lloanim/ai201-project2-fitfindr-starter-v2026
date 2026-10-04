# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**
<!-- Why 4 of 5 and not 5 of 5? Something about your search, probably —
     "my search is a plain keyword match and some phrasings will miss" is a
     real answer. -->

I chose 4 of 5 because my search is exact keyword matching, so a different phrasing of the same request can miss, and the agent never gets past the first tool. The model can also occasionally stop early or skip a step even when search succeeds, so I do not expect a 5 of 5. 

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
<!-- Why is 5 of 5 reasonable here when criterion 1 isn't? What's different
     about this path? -->

Nothing on this path involves the model. Parsing is regex, `search_listing` filters deterministically, and the branch is a check. That if the result list is empty, set `session["error"]` to a fixed message and return before `suggest_outfit`. The same query always gives the same result, so any miss is a bug, not variance. Criterion 1 depends on model output that changes between runs, which is why it can't have a target of 5/5.

---

## 3. Something about state

<!-- YOU WRITE THIS ONE.

     How would you know that the item your search found is the same item the
     next tool received? Name something countable or observable.

     This is the criterion people find hardest, because state failure doesn't
     look like state failure — it looks like a tool problem. Something that
     compares session["selected_item"] against what actually reached
     suggest_outfit is the shape you're after. -->

Across 5 runs, compare `session["selected_item"][id]` after the search step against the `new_item["id"]` recorded in the trace's `suggest_outfit` step inputs. Target: 5/5

**Why this target:**

I expect 5 of 5 because this is a choice I made in my code, so there is no acceptable failure rate. This is not a judement from the LLM. If the IDs do differ then that means the outfit is built around an item the user never saw in the results. Otherwise, there is a bug in how state is passed between the tools. 

---

## 4. Something about the fit card

<!-- YOU WRITE THIS ONE.

     The fit card calls a model, so the same input can produce different words
     each time. That's not a bug — it's the nature of the tool. So what would
     make it acceptable?

     Think about what you'd actually be unhappy to see. A caption that never
     mentions the price? Two different items producing the same opening
     sentence? A card longer than a caption anyone would post? Any of those can
     be turned into a number. -->

For at least 4 of 5 tries, two items chosen from different categories in `listings.json` each generate a fit card, and the two cards do not have identical opening sentences. 

**Why this target:**

If two items in different category and style produce the same opening sentence, then that means the prompt is producing a template rather than reacting to the item. I chose 4 of 5 because the model is non-deterministic and short openers can repeat by chance. 

---

## 5. Your choice

<!-- YOU WRITE THIS ONE TOO.

     Pick something you actually care about getting right. Speed, the empty
     wardrobe path, what happens when the model can't be reached, whether the
     search respects a price ceiling — anything, as long as it names a number
     or an observable outcome. -->

For 5 of 5 tries, with an empty wardrobe, the top `search_listing` result is passed to the fit card. The card returns without error, it does not refer to items as if they own it, and it names at least one color or item type to pair with the searched item. 

**Why this target:**

Every new user would start with an empty wardrobe, so the result should be a general advisement but still helpful like color or item type. I do not allow misses because failing outright or referring to clothes the user doesn't own would make the app feel broken on the very first use. 

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
