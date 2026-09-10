# Is there a structural change in India's bank credit? (2019-2026)

This is my time series project for MSc coursework. Everything is done in
Excel, not Python — I wanted to actually see and understand every formula
myself rather than run someone else's library function and trust the
output.

## Why I picked this

I wanted a real dataset, not a toy one, and something I could actually
explain if someone asked me about it. India's central bank (RBI) publishes
monthly data on how much credit banks give out, broken down by sector
(Agriculture, Industry, Services, Personal Loans etc.). I got curious
whether this has been growing "normally" the whole time, or whether
something actually changed along the way — like a shift in pace, or banks
lending very differently now compared to 2019.

"Structural change" sounds like a big scary econometrics term but it
really just means: did something durable shift, rather than the numbers
just naturally bouncing around? I'm looking at this in two ways:

1. Did the **overall trend** in total credit growth change at some point?
2. Did **which sectors** get the money change over time (industry vs.
   households, say)?

## The data

RBI Table No. 15 (Deployment of Gross Bank Credit by Major Sectors),
downloaded straight from the RBI website. Covers Jan 2019 to Jun 2026,
which comes out to 90 monthly-ish data points (RBI actually reports on
specific fortnight-end dates, not exact month-ends, so the spacing isn't
perfectly even — worth keeping in mind).

One thing I noticed reading RBI's own footnotes: the survey's bank
coverage changed twice in this window (around Oct 2019 and again around
Apr 2021, going from about 90% of banks reporting up to about 95%). That
matters a lot for this project, because it means some "changes" I find in
the data could just be RBI counting more banks, not banks actually
lending differently. I tried to flag this honestly rather than just
ignore it.

## What I actually did (methods)

I don't have statsmodels or any of the proper econometrics packages in
Excel, obviously, so everything below is built from basic Excel formulas
(SLOPE, INTERCEPT, STEYX, DEVSQ, OFFSET, COUNTIF, FDIST, TDIST). This
was honestly the most useful part of the whole project for me — having
to actually build the ADF test and Chow test formula-by-formula instead
of calling `adfuller()` made me understand what they're actually doing
in a way I don't think I would have otherwise.

**Growth Analysis sheet** — just plotting the level, log level, and
growth rates, before doing anything statistical. Always feels like a
good first step.

**Stationarity (ADF test)** — checks whether log(credit) is stationary.
Spoiler: it isn't, which makes sense since credit basically only grows.
Then I checked whether the growth *rate* is stationary — it is, which is
what you want if you're going to model growth rates instead of raw
levels.

**Chow test** — I picked 3 specific dates where I thought lending might
have shifted (COVID lockdown, the RBI coverage change I mentioned above,
and the 2023 HDFC-HDFC Bank merger which moved a massive mortgage book
onto a bank's books overnight) and tested each one properly.

**Breakpoint scan** — because picking my own dates felt a bit like
cheating (I already knew what "should" matter), I also built a version
that tries every possible split point in the data and finds whichever
one fits best on its own, with no assumption from me about the date.

**Sector composition** — separately from all the "break" stuff, I
checked whether the % of credit going to each major sector has been
drifting over time — this is more of a slow structural shift than a
sudden break.

## What I found

The short version: yes, both things seem to be genuinely true here.

For the trend: even without me telling Excel where to look, the
breakpoint scan landed on **December 2021** as the single best split
point, and growth roughly doubled after that (about 0.5%/month before,
up to about 1.15-1.2%/month after). All three of my hypothesis dates
(COVID, coverage change, HDFC merger) also came back as individually
significant when I tested them with the Chow test, so this doesn't look
like a fluke.

For the sector mix, this was the part I found more interesting honestly.
Industry's share of total credit has fallen fairly steadily — from about
29% in 2019 down to under 22% by 2026 — while Personal Loans went the
opposite way, from about 25% up to 33%. They actually cross over around
2021, so today more credit goes to personal loans than to industry,
which apparently wasn't true a few years ago. All four sectors I tested
(Agriculture, Industry, Services, Personal Loans) had statistically
significant trends, not just these two, so it's not just one category
moving.

## Things I'm still not sure about

- My 3 hypothesis dates are all within about 3 years of each other, so I
  don't think a single-break model can really separate which one
  actually caused the shift. Might genuinely be all three at once.
- The breakpoint scan's answer (Dec 2021) is suspiciously close to when
  RBI widened their survey coverage (Apr 2021) — so part of what looks
  like a real economic break might actually just be a reporting/coverage
  change. I don't think I can fully separate these two explanations with
  the data I have.
- I used the simple (non-augmented) Dickey-Fuller test, just one lag,
  because I couldn't get a multi-lag LINEST array formula to recalculate
  properly in Excel. Would want to redo this in Python with statsmodels
  at some point to see if it changes the conclusion.
- The HDFC merger is a genuine one-off event (a finance company's entire
  mortgage book got absorbed into a bank in one go), not organic lending
  growth, so the Personal Loans/Housing numbers right around 2023 should
  be read with that in mind rather than as pure "demand".
- 90 data points isn't huge for this kind of test. The critical values I
  used (Dickey-Fuller tables, etc.) are approximations meant for bigger
  samples, so I'd treat my exact p-values as roughly indicative rather
  than precise.

## Files

- `Bank_Credit_Structural_Change_Project.xlsx` — the actual project,
  everything is a live formula. Sheets: Summary, Raw Data, Growth
  Analysis, Stationarity ADF, Chow Test, Breakpoint Scan, Sector
  Composition.
- `credit_data_clean.csv` — the cleaned-up version of the raw RBI table
  (just the sectors I actually used, in a normal date x sector table
  instead of RBI's wide format).

## What I'd do differently next time

Probably test more than one break at once (I read this is called a
Bai-Perron test with multiple breaks, but couldn't figure out how to
build that in plain Excel formulas without VBA). Also would've liked to
actually forecast a few months forward using the post-break trend and
see how close it gets to the real numbers, but ran out of time for this
submission.
