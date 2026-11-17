# SolarSprint

**Operating an AI data center on California's curtailed solar power.**

## The problem

California curtailed about 3.4 TWh of solar generation in 2024 because the grid could not deliver it
to customers. AI computing could use that energy, but AI hardware is too expensive to sit idle
whenever the surplus stops, and grid connections large enough to back it up are scarce and slow to
obtain.

## Our approach

We study the site proposed in the report *Curtail to Compute* (Next 10, 2026): a 20 MW AI data center
near Fresno that runs on curtailed solar, a 10 MW / 40 MWh battery and a grid connection of only
10 MW, half the size of its load. Customers buy computing in three service tiers according to how
long they can wait: immediately, within a day, or within a week.

We built the software that operates this site. Every hour, an optimizer plans the next 14 days
against 20 possible solar forecasts. It decides which jobs to accept, when each job runs, and whether
its power comes from solar, the battery or the grid. It accepts only the work it can finish on time.

## Results so far

We test the optimizer in full-year simulations that use real AI job traces, CAISO market prices and
the PG&E tariff. Its settings were chosen on 2023 and then frozen. The results below are for 2024 and
2025, two years it had not seen.

| Operator | Work finished, 2024 | Work finished, 2025 | Accepted work finished on time |
|---|---:|---:|---:|
| Our optimizer | 91.3% | 91.8% | 100% |
| Our best rule-based operator | 86.1% | 85.0% | 87% |
| The tier design sketched in the report | 36.1% | 37.1% | 36 to 37% |

Three conclusions follow.

1. A site like this cannot be run on simple rules. The report's tier design never buys grid power for
   flexible jobs, so most of the work stalls when solar runs short.
2. Planning ahead, and declining work that cannot be finished, is what delivers reliable service. The
   optimizer finishes 5 to 7 points more work than our best rule-based operator, and nothing it
   accepts is late. Its electricity cost per unit of work is about the same.
3. The scarce resource is the grid connection, not the energy. Electricity is about 1% of the value
   of the computing it powers, while one more megawatt of firm grid connection is worth several times
   what the utility charges for it.

These are simulation results, not measurements from an operating site. The comparison with the
report's tier design uses our own implementation of it, since the report did not simulate its tiers.

## Next steps

We are finishing the remaining test runs, which vary the size of the grid connection and the battery.
After that we will price the three service tiers, model how customers choose among them, and study
how the site could sell its flexibility back to the grid. The work is being written up as a research
paper.

## About

SolarSprint is a University of Pennsylvania senior design project led by Maanas Goel, carried out
with Prof. Le Xie's lab at Harvard and advised by Prof. Benjamin Lee at Penn. The code is in a private
repository, and we are happy to walk through the work on request. Methods, test design, limitations
and earlier results are described in the [technical notes](docs/technical-notes.md).
