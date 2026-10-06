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
This is an important factor to ensure we can successfully fetch items from the wardrobe. While 5 out of 5 would be ideal, 4 out of 5 (an 80% success rate) is a practically achievable target. Since the initial search relies on keyword matching, it might occasionally miss items if the exact phrasing is not used.
<!-- Why 4 of 5 and not 5 of 5? Something about your search, probably —
     "my search is a plain keyword match and some phrasings will miss" is a
     real answer. -->

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
It is important for the agent to stop if no matching results are found; otherwise, we cannot distinguish between an item actually retrieved from the wardrobe and one the agent made up. Therefore, it is crucial to have a strict 5 out of 5 success rate here.
<!-- Why is 5 of 5 reasonable here when criterion 1 isn't? What's different
     about this path? -->

---

## 3. Something about state


<!-- YOU WRITE THIS ONE.

     How would you know that the item your search found is the same item the
     next tool received? Name something countable or observable.

     This is the criterion people find hardest, because state failure doesn't
     look like state failure — it looks like a tool problem. Something that
     compares session["selected_item"] against what actually reached
     suggest_outfit is the shape you're after. -->
In 5 out of 5 tries, the exact item ID returned by the initial search must be the exact same item ID passed into the suggest_outfit tool.


**Why this target:**

Because the first search call is what provides the item id and there is no reason for the next tool to not receive the same item id.

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
In 3 out of 5 tries, the generated fit card must include the item's price.


**Why this target:**
If a user likes everything about an outfit but finally checks the price and it's out of their budget, it results in a very disappointing experience and wasted time. Providing the price upfront in the fit card helps the customer decide whether to proceed. The target is 3 out of 5 because the model might occasionally format the card differently and miss the explicit price label, but it should be present most of the time.


---

## 5. Your choice
When the model cannot be reached, the agent returns a specific error message stating the model is unavailable, in 5 out of 5 tries.
<!-- YOU WRITE THIS ONE TOO.

     Pick something you actually care about getting right. Speed, the empty
     wardrobe path, what happens when the model can't be reached, whether the
     search respects a price ceiling — anything, as long as it names a number
     or an observable outcome. -->



**Why this target:**
This is a gatekeeper criterion that requires a strict 5 out of 5 success rate. If the model is unavailable, the agent must return a clear error message. This helps us distinguish between actual retrieved wardrobe data and fallback behavior. While the model can suggest styles based on its general knowledge, the user must explicitly know that the suggestion is made up and not actually from the wardrobe.


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
