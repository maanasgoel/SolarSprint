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

## Method

The operator solves a two-stage stochastic linear program every hour and carries out only the first
hour of the plan (model predictive control). The scenarios $`s`$ are 20 solar paths drawn from real
recent days. Time steps $`i`$ have length $`L_i`$: hourly for the first 36 hours, then coarser out to
14 days. The objective is the expected cost of running the site plus the value of the work refused:

```math
\min \;\; \frac{1}{|S|} \sum_{s} \left[ \, \sum_{i} L_i \big( (\pi_i + \phi)\, g_{is} + w\, d_{is} \big)
 + \lambda \sum_{m} \big( \Gamma_{ms} - \bar{\Gamma}_m \big) + \kappa \sum_{f} u_{fs} \right]
 + \sum_{j \in \mathrm{new}} \rho_j\, E_j\, (1 - a_j)
```

The first term is grid energy $`g`$ bought at the day-ahead price $`\pi`$ plus the utility's delivery
charge $`\phi`$, and battery wear $`w`$ on each unit discharged $`d`$. The second is the utility's
demand charge $`\lambda`$ on each month's highest grid draw $`\Gamma`$, above the peak already set
that month. The third is a large penalty $`\kappa`$ on accepted work $`u`$ that would finish late.
The last is the value $`\rho_j`$ lost by refusing the share $`1 - a_j`$ of a new job with energy
$`E_j`$. The plan must satisfy, in every scenario:

```math
\begin{array}{ll}
\displaystyle \sum_{f} x_{fis} + N_i + c_{is} = y_{is} + d_{is} + g_{is} & \quad \text{power used = solar + battery + grid} \\[8pt]
\displaystyle y_{is} \le \sigma_{is}, \quad 0 \le g_{is} \le \min(G, \Gamma_{ms}), \quad \sum_{f} x_{fis} + N_i \le C & \quad \text{solar, grid and site limits} \\[8pt]
\displaystyle e_{is} = e_{i-1,s} + L_i \big( \eta\, c_{is} - d_{is} / \eta \big), \quad 0 \le e_{is} \le \bar{E} & \quad \text{battery energy} \\[8pt]
\displaystyle \sum_{i' \le k} L_{i'}\, x_{f i' s} + u_{fs} \ge M_f(k) & \quad \text{flexible work stays on track for its deadlines}
\end{array}
```

Here $`x_f`$ is the power given to a group of flexible jobs, $`N`$ the power of urgent and standard
jobs, $`c`$ battery charging, $`y`$ the solar used out of the solar available $`\sigma`$, $`G`$ the
grid connection, $`C`$ the site's 20 MW, and $`M_f(k)`$ the least work a group must have completed by
step $`k`$ so that none of its jobs misses its deadline. Decisions for the current hour, and every
acceptance $`a_j`$, are the same in all scenarios, because they are made before the solar is known.
Each plan solves in about a second.

Rescheduling flexible computing toward cheap or clean power is an established idea, and others are
building computing at solar plants. What is new here is the operating problem itself: admission of
individual jobs in three service tiers at a site whose firm grid connection is smaller than its load,
under a real utility tariff with a monthly demand charge, evaluated on full years the method had not
seen and against hypotheses fixed in advance.

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
