# SolarSprint: Scheduling AI Computing on California's Wasted Solar Power

## The problem

California throws away ("curtails") solar power it can't move, about 3.4 TWh in 2024, mostly because of local transmission congestion. Prof. Lee's report *Curtail to Compute* (Next 10, March 2026) proposes a 20 MW data center near Fresno that runs AI computing on that wasted power, backed by a battery and a limited grid connection.

## What we're building

Scheduling software for that site's operator. Customers buy compute in three service tiers: urgent (start now), standard (within 24 hours), and flexible (within 7 days). Every hour, the scheduler decides which jobs run and whether their power comes from curtailed solar, the battery, or the grid. When a job arrives, it accepts it, refuses it, or offers a later, cheaper deadline.

## Results so far (simulated)

- **Real AI workloads:** on held-out Alibaba GPU job traces across four seasons, our three-tier operator completed **95.9% of the work vs. 31.6%** for the report's three-tier rule, with the same solar, battery, and grid. The whole gap comes from the flexible tier, which our operator keeps on time by buying grid power when needed.
- **Economics:** about **$10.3M per year** more electricity margin than the report's rule, and the most financeable policy we tested, though it still needs higher prices than the base menu to cover the site's loans (GPU costs excluded).
- **Uncertain solar:** hedging across solar forecast scenarios raised completed work to **97.9%**, at a higher electricity cost.
- **Site sizing:** under our assumptions, a 20 MW grid connection with no battery was the best setup in every fresh test. That result depends heavily on the grid fee, and we're re-checking it with the real utility tariff.

Every number is from our own simulation and is reproducible with one command in the project repo. These are research results, not measurements from an operating site.

## What's next

- Re-run site sizing with the real PG&E demand charges
- Pricing and service rules that keep customers honest about deadlines and cover the site's costs
- On-time guarantees that stay reliable on new workloads
- A senior thesis built on the strongest result

The code lives in a private repository. Happy to walk through the work on request. Advised by Prof. Benjamin Lee at Penn, and Prof. Le Xie's lab at Harvard. Work in progress.
