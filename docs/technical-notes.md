# SolarSprint: technical notes

Last updated November 17, 2026. This file holds the detail behind the project summary in the
[README](../README.md). Every number comes from our own simulation.

## The site

| Part | Size |
|---|---|
| Computing | 20 MW |
| Grid connection | 10 MW in the main case. We also test 0, 5, 17.5 and 20 MW. |
| Battery | 10 MW / 40 MWh in the main case. We also test none and 20 MW / 80 MWh. |
| Solar | The statewide pattern of curtailed solar, up to 30 MW at the site. |

## The three service tiers

| Tier | The promise | Typical work | Share of energy |
|---|---|---|---:|
| Urgent | Starts when it arrives, or is refused | Interactive use, live inference | 10% |
| Standard | Starts within 24 hours, then runs without interruption | Batch inference, evaluation | 10% |
| Flexible | Finishes within 7 days of the earliest it could, and can pause | Model training | 80% |

The 24-hour window matches batch AI products sold today at half price. The 7-day window is our
assumption, and we vary it. In the simulation the tier is set by the type of job. Customers do not
yet choose.

## The optimizer

Every hour the optimizer writes a plan for the next 14 days. The plan covers which jobs to accept,
when each one runs, when the battery charges and discharges, and how much grid power to buy.
Tomorrow's solar is unknown, so the plan is checked against 20 possible solar futures taken from real
recent days. Only the first hour of the plan is carried out, and an hour later the optimizer plans
again.

The optimizer minimizes the electricity bill, battery wear and the value lost by refusing work. The
bill has three parts: energy at the day-ahead price, the utility's delivery charge, and the utility's
charge on each month's highest grid draw. Accepted work must stay on track for its deadline in every
one of the 20 futures. The plan is a linear program and solves in about a second.

## Results in detail

"Work finished" is the energy of jobs finished by their deadline, divided by the energy of all jobs
offered. Every operator faces the same site, jobs, solar and prices.

### Held-out years, 2024 and 2025

The table shows the mean of five different workloads per year.

| Operator | Work finished, 2024 | Work finished, 2025 | Accepted work on time |
|---|---:|---:|---:|
| Optimizer | 91.3% | 91.8% | 100% |
| Best rule-based operator | 86.1% | 85.0% | 87% |
| The report's tier design, as we implemented it | 36.1% | 37.1% | 36 to 37% |

- The optimizer beats the best rule by 5.2 points in 2024 and 6.9 points in 2025, in all five
  workloads of both years.
- Electricity cost is a tie at about $45 to $47 per finished MWh. We had predicted that the optimizer
  would also be cheaper, and that prediction failed.
- Most of the gain comes from the right to refuse work at arrival. An optimizer that must accept
  everything finishes 87 to 88%.
- With a 17.5 MW grid connection the optimizer finishes 99.7 to 99.8%, against 91.6 to 92.6% for the
  rule. It also holds the monthly peak at about 15.7 MW, where the rule sits at 17.5 MW.

These figures are a first read of the main comparison, which is 100 year-long runs. A further 158
runs that vary grid size, battery size, tier mix and other settings finish around November 20.

### Tuning year, 2023

| Operator | Work finished | Accepted work on time | Electricity cost per finished MWh |
|---|---:|---:|---:|
| Optimizer | 91.9% | 100% | $46.35 |
| Best rule-based operator | 87.1% | 88.7% | $47.69 |
| The report's tier design, as we implemented it | 36.7% | 36.7% | $49.07 |

The rule's cost includes a flaw we found later in how it handles negative prices. With that
corrected it costs $46.11, so cost is about equal in this year too.

All three operators serve urgent and standard work in full. The gap is in the flexible tier, where
the optimizer finishes about 90% of the work, the rule 84% and the report's design 22%.

### The value of the grid connection

- Free energy is worth a few million dollars a year at this site. The computing it powers would rent
  for several hundred million dollars a year.
- A 10 MW connection alone could carry at most 63% of the work offered. The solar matters because it
  supplies megawatts that the connection lacks.
- At a small connection, one more megawatt of firm grid is worth about $100 per kW-month to the site
  after all electricity charges. The utility charges $17.19 for it. The value falls to about zero
  between 17.5 and 20 MW.
- A 10 MW / 40 MWh battery replaces only about 1 MW of connection.

The last two points come from the rule-based operator on 2024 and are being repeated with the
optimizer.

## Notes on the numbers

- The 2024 and 2025 figures are the held-out results. The 2023 figures (91.9%, 87.1%, 36.7%) are from
  the tuning year.
- The optimizer finishes about 2.5 times the work of the report's tier design. That comparison uses
  our own implementation of one paragraph of the report. The margin over our own best rule, 5 to 7
  points with no late work, is the fairer measure of what the optimizer adds.
- The optimizer is not cheaper per unit of work. Cost is a tie.
- Money figures leave out the cost of the GPUs.
- The October figures listed under "Earlier results" come from earlier experiments with the
  rule-based operator and are no longer the main result.

## Test design

- The hypotheses, settings and code were fixed on November 10, before any 2024 or 2025 run. There are
  ten hypotheses and 258 year-long runs. Failed tests are reported alongside passed ones.
- Settings were tuned on 2023 only. Each held-out year uses five workloads, and operators are compared
  on the same jobs.
- Inputs are Alibaba GPU cluster job traces, an Azure LLM inference trace, CAISO curtailment and
  day-ahead prices, and PG&E's B-20 tariff with its monthly demand charge.
- The simulator checks the power balance in every simulated hour and stops if it fails. The code base
  has 78 test files.
- Three of the later runs stopped on a rounding error that put one hour's load above the site's
  20 MW. The remaining runs use a correction. A finished run repeated with the correction gave an
  identical result, so no finished result changes.

## Limitations

- The work is a simulation. There is no hardware and there are no customers.
- The solar supply is an assumption: the site receives the statewide pattern of curtailed solar,
  free, up to 30 MW. A real site would depend on one plant and one contract.
- Tiers are assigned by job type. Tier prices and customer choice are the next part of the work.
- A half-size grid connection is only a sensible choice where full-size connections are scarce or
  slow to obtain.
- Hardware cost is not included in the money figures.
- Grid energy is billed at the ZP26 hub price. A PG&E customer would settle at a price about $8 to
  $11 per MWh higher, and a re-billing is planned.
- The main job trace is from 2020, before large language models.
- The simulation uses hourly steps and day-ahead prices only.

## Relation to other work

Moving flexible computing to hours with clean or cheap power is an established idea (GreenSlot, 2011;
GreenSwitch, 2013). Companies are building computing at solar plants today (Rune, Soluna), and
software for flexible data centers exists (Emerald AI). We do not claim those ideas.

Our contribution is an operating policy for a site whose firm grid connection is smaller than its
load. It handles three service tiers and the admission of individual jobs under a real tariff with a
monthly demand charge, and it is tested on full held-out years. The related question we study is what
a megawatt of firm grid connection is worth compared with curtailed solar and a battery.

## Earlier results, October 2026

These results used the rule-based operator, before the optimizer existed.

- On six held-out weeks of Alibaba GPU jobs, the rule finished 95.9% of the work against 31.6% for
  the report's tier design. Those weeks were about 85% flexible work, which is why the figures differ
  from the full-year ones above.
- The rule earned about $10.3M a year more electricity margin than the report's design, at prices we
  assumed.
- Planning against several solar scenarios raised finished work to 97.9%, at a higher electricity
  cost.
- With the real PG&E tariff and the rule-based operator, the best site had a 17.5 MW grid connection
  and no battery. It still needed prices about 1.2 to 1.3 times our base menu to cover its loans.
- No three-price menu we tested kept customers honest about their deadlines, kept them as customers
  and covered the site's costs. No on-time guarantee we tested was better than best effort.
