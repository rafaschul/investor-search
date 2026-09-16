# Field tests

Five markets, five runs, five rewrites. **Every rule in `SKILL.md` that looks fussy is fussy
because a version without it was run and produced a wrong file.**

**No company is named in this document.** The tests made judgements about real firms — that
one sells services rather than investing, that another's citation could not be retrieved.
Those judgements were right for deciding what to build and wrong to publish under a
company's name. Every count, reason and failure below is unchanged; only the labels are
gone, and none of the lessons needed them.

**This is a written account, not a raw log.** The raw run transcripts are working documents
full of named firms and are not published.

| # | market | chosen because | defects found |
|---|---|---|---|
| 1 | Czech Republic | a market nobody had touched | 13 |
| 2 | United Arab Emirates | saturated with "family office services" sellers; English-speaking; outside Europe | the silent-merge class |
| 3 | Singapore | an entire industry selling family-office *incorporation*; place-name naming | 14 |
| 4 | Estonia | small enough that the stopping rule could actually fire | the exhaustion rule, falsified |
| 5 | Latvia | a run made to exercise the current rules end to end | 3 |

---

## Test 1 — Czech Republic

**13 firms over 11 rounds. Stopped on budget.** ~72 names offered, 6.5 a round.

### What it found, and what it cost

**There was no test for whether a firm was an investor at all.** Five of the six firms
verified through Czech-language queries were law firms, accountancies and consultancies
*selling* family-office services. Real names, live sites, real headquarters — they passed
every check the skill had. Then the type allowlist saw the words `family` and `office` and
typed them `family_office`, with a quotation in the original language attached.

> The file would have been beautiful. Every row sourced, dated and quoted — and a third of
> it lawyers. **Provenance is not truth**, and a row can be perfectly provenanced and still
> not be a family office.

This became **Job 0**, which now runs before everything else.

**"Could not fetch it" and "it does not exist" were the same rule.** Three of six rejections
were firms that almost certainly exist, killed by a robots.txt, a cache-only encyclopedia
page, and a fetch tool that refused a URL named inside another page. One of those — an
unambiguously real investment group — then *also* failed the headquarters check, because the
only page stating its city was the unfetchable one. Now split: 404 is fatal, everything else
keeps the row and says so.

**The fold's step order was backwards.** Punctuation was collapsed before legal forms were
stripped, turning the Czech `a.s.` into the two tokens `a s`, which could never match a list
holding `as`. **All four real duplicate pairs in the run failed on this.** Not a Czech
problem either — it breaks dotted `S.p.A.` and `B.V.`, forms already on the list. The worked
example that hid it happened to be undotted.

**The classification rule rewarded ignorance.** It blanked the type of firms classified
*correctly* from their own pages, while firms nobody could classify kept `unknown` — because
`unknown` is never contradicted. Now it only ever blanks an *inferred* type.

**Four type values could not hold the two largest firms in the country.** Both describe
themselves as investment groups, both are one family's money, both shipped as `unknown`.
`investment_group` was added.

### The language finding, both halves

```
English queries   12 offered ·  7 survived  (58%)
Czech queries     60 offered ·  6 survived  (10%)
```

Czech surfaced **five times the names**, including firms English never produced at all. It
also surfaced five times the advisers, because in Czech the phrase is largely law-firm
marketing. **The local language finds more names; it does not find more firms.**

### Verdict at the time

*"Not trustworthy in someone else's hands. The bar rejects for reasons it cannot
distinguish, and never checks the one thing that matters."*

---

## Test 2 — United Arab Emirates

**15 firms over 13 rounds. Stopped on budget.** 74 names offered.

### Job 0 validated, hard

**16 advisers rejected against 15 firms kept — 52% of everything resolved.** One round, using
as targeted a query for a real investor as could be written, returned **eight results, eight
advisers, zero investors** — company-formation agents and free-zone licensing consultancies
selling "family office setup" packages.

The rule turned out to work on a distinction one word wide. A genuine single family office
described itself as one that *"advises the investment vehicles associated with the … family"*
— its own principal. Read against a keyword list, "advises" is disqualifying and a real
investor is thrown away. **Watch the preposition** became a rule.

### The defect that mattered: the fold merged two different companies

The definition of "distinctive" was *"a token of 4+ characters not on any list."* That is a
string-length test, and a city name passes it.

```
<City> Capital Group      a private investment group
<City> Investment Office  a government agency
                ↓
   both fold to   <city>
```

`<City> + <generic financial noun>` is the **modal** naming convention across the Gulf, and
common in Asia and Latin America.

> The Czech defect was duplicate rows — visible, annoying, a reader fixes them. **This is one
> row for two companies, with both sets of sources pooled, looking exactly like a correct
> merge.** Nothing in the file marks it.

Two fixes followed. The narrow one: a place name is never distinctive, and when you cannot
tell whether a token is a place, treat it as a place. The structural one, which matters more:
**folding proposes, it never merges** — same key plus a shared domain merges; anything else is
flagged and both rows are kept.

### Everything else it broke

- **Nine of ten local legal forms did not fold.** Only the two containing the English `LLC`
  survived. The free-zone forms the entire local investment-vehicle industry is built on were
  absent from a European list.
- **The language rule inverted.** Arabic returned **1 name against English's 73**, and the
  one was an adviser. The rule had been stated as a law and was a finding about one country.
  It is now *"search the language business is actually conducted in"*, with a test.
- **A transliterated firm appeared under three spellings** and the typo rule split them,
  because vowel substitution is "not a typo". For transliteration it is the *primary* axis of
  variation.
- **A leading article is a dropped token, not a dropped character**, so no rule could reach it.
- **There was no answer for the operating conglomerate** — family-owned, enormous, not an
  adviser, and not deploying capital into anything it does not operate. **That is how Gulf
  family capital is held.** `operating_group` was added.
- **A clearly-described private equity firm shipped as `unknown`**, because neither `private`
  nor `equity` was an owned word. `private_equity` was added.
- **Six sovereign entities had no correct destination** — not family offices, not VCs, and
  emphatically not `unknown`, since their nature is perfectly well known.

### Verdict at the time

*"No — not outside Europe. Job 3's fold is the one part specified as an algorithm rather than
a judgement, and it is the part that failed hardest when moved to a new market."*

---

## Test 3 — Singapore

**33 firms over 10 search rounds plus 4 verification passes. Stopped on budget.**

### The fold rewrite vindicated

The tester implemented the fold from the skill's text and ran it over every name the run
produced, rather than reasoning about it.

**One fold key collided five ways, across four different domains** — a multi-family office,
two unrelated companies sharing a name, an advisory vehicle and a licensed fund manager. That
is the UAE failure at five times the scale. The previous version merges all five into one row
with five domains' sources pooled and nothing marking it. **This version flagged all five and
merged none.**

**7 flags on 33 rows — 21% — and every one a genuine question.** A high flag rate is the rule
working.

The place-name guard fired correctly but proved nothing: every wrong merge actually caught was
caught by the *domain* rule. And the guard turned out to be undecidable in a market full of
heritage brand names that are also district names — which produced the rule that **when you
cannot decide, it is a place, and you do not fold.**

### The defects it found

- **A spelled-out legal form was missing** — the commonest written form in that market —
  splitting a firm from itself. So was the market's own fund vehicle.
- **The fold decision table did not carry its own parent/child exception**, which sat sixty
  lines away under a different heading. *"Follow the table literally and you merge them. I
  nearly did."* The exception now sits inside the table.
- **Citations packed into a CSV cell.** Every in-cell format tried broke: one separator is
  ordinary inside URLs and shreds them silently; the other made the file **174 physical lines
  for 34 records**, so every line-based tool reported 173 firms. Citations became their own
  file with a join key and a rung column.
- **Ten of thirty-three rows were `unknown`** because mezzanine, secondaries, credit, timber
  and real estate are owned by no type. A strategy is not an entity type; those words now
  belong to `private_equity`, and the strategy belongs in `sectors`.
- **Sovereign subsidiaries had no rule** — and in that market their commercial arms are a
  large, legitimately targetable population. They are now kept, with the parent recorded.
- **Angel networks and membership bodies** fit neither "invests" nor "advises". `syndicate`
  was added.
- **The working-language test is circular** if you sample only firms you already hold, since
  those came from English directories. A registry or a local news source has to be checked too.

### And the finding that changed the shape of the whole thing

> *"~40–45% of the skill is rule **provenance**, not rules. Every defect I found is a rule
> that exists but is unreachable at the moment of decision."* … *"The operative rules are
> interleaved with their justifications, and at 45KB the justifications win."* … *"Czech found
> 13, UAE found the silent-merge class, Singapore found 14 — and most are placement and
> list-completeness failures, not reasoning failures. That's a formatting plateau."*

The skill was split into three files the same day: rules, lists, evidence.

---

## Test 4 — Estonia · the most important result

The first three runs all stopped on **budget**. Which meant the one sentence this skill exists
to be able to say — *"I know this search is finished"* — **had never once been said.** Estonia
was picked because it is small, has a fully public registry, and plausibly has few enough
firms that six dry rounds is reachable. Budget: 30 rounds.

### The stopping rule fired. It was wrong.

**It declared exhaustion at round 23, on a dry streak of rounds 18–23.**

**It was falsified at round 26 of the same run.** A plain English query that had not been tried
produced two firms that had not appeared once in 25 prior rounds. Round 28 added a third. A
fourth row was promoted from a bare directory line to a sourced firm after the declaration.

**Four of the run's 29 rows arrived after the rule said the market was finished.** Nothing
about Estonia changed between round 23 and round 26. A query shape changed.

### Three structural causes, now three gates

1. **The queue was invisible to the counter.** At round 8 the financial regulator's own
   register handed over **85 fund managers**. Ten closed that round; the other 75 became a
   queue which, by the rule's explicit text, *can never reset the streak*. **The rule fired
   while holding 75 regulator-confirmed names it had told itself not to count.**
   → *Gate 1: nothing resolvable may be left pending.*

2. **Six dry rounds were six web searches — one surface failing six times, not six surfaces
   failing.** The registry surface yielded 85 names on a single fetch and an association list
   28, *after* web search had gone dry. **A surface never tried cannot be dry.**
   → *Gate 2: the streak must span at least three kinds of source.*

3. **One garbled fetch caused the streak.** At round 2 an article listing family offices in
   the region returned cipher text. The garble rule protected the streak
   *counter* but not the *evidence*, so the page was never recovered. The same content, found
   on a mirror at round 26, held two of the three firms that broke the exhaustion.
   → *Gate 3: no round in the streak may rest on a page that failed to load.*

And a fourth cause no gate can fix: **the streak hinged on a single judgement.** One Job 0
rejection — an energy developer that a directory had listed as an investor — is what made one
round dry. Trust the directory and the streak breaks at five and never fires.

### What it costs the claim

> **"Six queries I chose stopped working" is not "this market is exhausted."**
> It is the strongest sentence in the product, and it was resting on the weakest evidence in it.

The skill may now claim exhaustion only **on the surfaces it names**, and it must name them —
so a reader who knows a further surface can say so.

**The three gates have not themselves been field-tested.** They were written after this run.
*(A fourth — Gate 0, requiring that the record exist on disk at all — was added later, for a
reason unrelated to this test.)*
Every run in this package stopped on budget. What the skill reliably produces today is a
*defensible partial list*; the completeness claim is a design with an argument behind it, not
a measured result. That is stated in `SKILL.md`, in `README.md` and here, on purpose.

### Other defects from the same run

- **Legal forms written in front of the name.** Baltic and Nordic registries write `AS X`
  while the firm writes `X`, and the same register writes `X Capital AS`. An end-anchored
  strip folds one and not the other. **Five real pairs failed.** No list edit fixes it — the
  bug was the anchor, not the entries. Forms are now stripped from both ends.
- **Rebrands are invisible to every rule there is.** A renamed firm shares no characters with
  its old name; no normalisation, typo rule or distinctiveness test can reach it. **Three turned up in one small market.** One was
  recorded with a citation behind it; the others were named in the run but not sourced well
  enough to record, which is the empty-field rule doing its job on our own run. Only the domain and the sentence (*"formerly"*) find them,
  and the old name must go into the exclude list or it returns every round as a name never
  seen.
- **A type abbreviation was missing from the tail list**, splitting a firm written one way
  from the same firm written the other.
- **No reject code for "not an investment entity at all"** — distinct from a real adviser
  correctly excluded, and the distinction is how you find out a directory is unreliable.
- **Individual angels had no code path.** One row means one firm; 26 named angels with no
  vehicle had to be filed under a code for something else. They are now parked under their own
  reason, which is a *park*, not a rejection — they are legitimate targets.
- **Ten of the run's 29 rows were `unknown`**, the strategy-word problem again in a different market.

### What the split did, and did not, achieve

`lists.md` was **extended with local legal forms before round 1** — the first time in four
tests that instruction was acted on, which is the split working. But it produced a **false
sense of completion**: the market felt localised, and the fold failed anyway, because the
defect was in the step that reads the list rather than in the list.

`why.md` was consulted **zero times** across 28 rounds. That is the design succeeding.

### The output design held

`sources.csv` as a separate file: **a clean join in both directions, zero orphans, zero
uncited firms, and physical lines equal to records in every file.** (The run produced 29 rows
and 136 citations; one row did not clear the bar.) The rung
column immediately surfaced the rows resting on a commercial aggregator alone — which is what
Job 2 asks for and what no packed-cell format could express.

---

## Test 5 — Latvia

**The first run made to exercise the current rules end to end.**
Twenty-round budget, stopped on budget, reported PARTIAL. Five firms verified, one adviser
rejected, thirty-three names left unresolved.

Three findings, all new, none of which the first four tests could have produced.

### 1 — Job 0 barely fired, and the reason was the skill's own discipline

An association membership page gave **31 named organisations in a single round**. The
association's own description of its membership covers both fund managers and the legal and
financial advisers who serve them, without saying which member is which.

Several of the 31 are large professional-services firms. A human reading the list would place
them without hesitation. **The skill may not act on that**, because Job 4's rule is
that a type is read from evidence and never from a name — and the same logic governs Job 0.
Deciding would have meant one fetch per firm.

The run spent its budget searching for further names instead.

> **Result: 1 rejection recorded, 16 firms left as `unresolvedJob0`.** The adviser test — the
> skill's headline capability, measured at 5-of-6 and 52% in earlier markets — effectively did
> not run, on a list whose composition made advisers likely.

The rule is right. **The ordering was wrong**, and nothing in the skill said so: a list of N
named firms is N decisions already waiting, and resolving them is cheaper per decision than
finding the next name. That is now a rule in Job 2.

### 2 — A manager is not its funds, and nothing said which one is the row

Three of the five verified firms are fund managers. One runs **three separately named funds**;
its own site names all three. Nothing in the skill said whether the row is the manager or the
fund, so three rows were available where one organisation exists — in a file whose entire
premise is that one row means one organisation.

A fourth case made it sharper: a **locally registered fund whose management company is
headquartered in another country**. Registration says one market, control says another, and
the two answers put the row in different countries' files.

Now stated: the row is the entity that decides where the money goes, the manager; and where
the manager is out of country, so is the row — said out loud, because it is arguable.

### 3 — Two headquarters, one column

One fund describes itself as based in two cities, in two countries, with neither subordinate.
`headquarters` holds one value and `headquarters_partial` only distinguishes country-only.

The first city was recorded. **Nothing in the file showed that a choice had been made** — which
is the same defect as a guessed cell, one level up: not a wrong value, but an invisible one.
The rule now records the first-listed city and requires the second to be named in the report.

### What worked

- **The fixed path, the index and the early write all behaved as designed** — this was their
  first live use.
- **`sourceUnchecked` fired correctly for the first time in five tests.** One firm's site
  refused the fetch via `robots.txt`; the skill kept the name pending rather than rejecting it,
  which is exactly what that code exists for.
- **The language precondition held.** Two rounds in Latvian offered two names and produced
  none; English produced everything. The same inversion as the UAE run, caught by the same test.
- **Every core field carries a citation, `investor_evidence` included** — the Job 0 verdict is
  now evidenced in the file rather than asserted.

### What it does not show

This run **never reached the exhaustion rule**, so the three gates remain unmeasured after
five tests. It also never used the company-registry surface, so Gate 2's three-of-five
surface requirement was not exercised. And a five-firm run cannot demonstrate the fold: no two
names in it collided.

---

## No sample output ships

Every one of these runs produced real files, and earlier versions of this package shipped one
of them. That stopped, for a reason worth recording here rather than hiding.

A shipped sample is a set of claims about itself — how many rows, how many citations, what was
removed and why — and every one of those claims has to stay true as the rules move. It did not.
Successive audits found a person's name inside a citation URL, a column used for something it
was not for, a rebrand the rules require to be recorded and the file did not record, citation
counts describing a file that had already been replaced, and finally a sample that broke a rule
**the same run had just produced**.

The rules are what this package is. The data was a liability with no corresponding benefit that
running it would not give you faster.

## Where that leaves it

**Sound, and measured across five markets:** the adviser test, the empty-field rule, the
provenance labels, the refusal to merge without corroboration, the separate citations file,
and the requirement that a partial run be reported differently from a finished one.

**Designed, argued, and not yet measured:** the three gates on the stopping rule — still
true after five tests, because no run has reached exhaustion. *(A fourth gate, Gate 0,
requires the record to exist on disk; it is a precondition on evidence, not a heuristic.)*

**Fixed after test 5, therefore measured once and not yet re-tested:** resolving a named list
before searching wider, the manager-not-the-fund rule, and the two-headquarters rule.

**Known to be market-specific and needing extension:** every list in `references/lists.md`. A
market whose legal forms are missing produces silent duplicates — not an error, just a wrong
file. That is the first thing to do before working a market not already covered.
