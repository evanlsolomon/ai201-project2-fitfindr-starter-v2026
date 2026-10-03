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
The search is deterministic, so the same query finds the same listings every
time. The slack is for the two model calls: one rate-limit pause or failed
request can cost a try without the loop being broken. Two misses would be a
bug, not bad luck.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
This path never calls the model (regex parse, deterministic search, an
`if not results` check), so nothing should vary. The cheapest listing is $12
and none are sized XS or XXS, so the test query can never match. A pass needs
the message to name the size or price and say what to change — "No results"
fails.

---

## 3. The item search found is the item `suggest_outfit` received

For a matching query, `session["selected_item"]["id"]` is the same as
`session["search_results"][0]["id"]` and the `id` of the item going into
`suggest_outfit` in the trace — 5 of 5 tries.

**Why this target:**
Passing the item along is plain Python with no model involved, so it should
never fail. A mismatch would mean a value skipped the session or got
overwritten.

---

## 4. The fit card gets the facts right, even though the words change

Run `'vintage graphic tee under $30'` 5 times. At least 4 of 5 fit cards:

- state the selected item's exact price (`$24` or `$24.00`; "under $25" doesn't count)
- name its platform
- are 2–4 sentences

Across the five, at least 3 cards open with a different first sentence.

**Why this target:**
At `TEMPERATURE = 0.9` the wording should change; the price and platform
shouldn't. 4 of 5 because the prompt can ask for those facts but can't force
them. If all five cards open the same way, the cache or temperature is stuck.

---

## 5. Search respects the size and price the user asked for

For each of these 5 queries, at least one result comes back and every result is
at or under the price and in a matching size — 5 of 5 queries.

1. `'vintage graphic tee under $30, size M'`
2. `'denim jacket size L under $60'`
3. `'jeans size W28'`
4. `'flannel under $25'`
5. `'top under $20, size S'`

A matching size means the size appears as its own token: `M` matches `M`,
`S/M`, `M/L` and `One Size`, but not `XL` or `US 7`.

**Why this target:**
No model is involved, so a filter that lets one wrong item through will do it
every time. Sizes in the data mix letters, waists (`W28`), shoes (`US 7`) and
`One Size`, so a plain substring check (`"s" in "us 7"` is true) is the likely
bug. Query 1 has a listing at exactly $30, so it also checks that "under $30"
includes $30.

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
