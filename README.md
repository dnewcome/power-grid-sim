# Transmission Grid Optimizer

An interactive, browser-based tool for simulating electrical transmission networks, analyzing power flows, and optimizing battery storage placement across a grid.

## What It Is

This is a single-page HTML5 application — no server, no install, no build step. Open the file in a browser and you have a working grid simulation environment. It's aimed at engineers, planners, or students who want to explore how transmission grids behave under different load conditions and where energy storage can do the most good.

The tool sits somewhere between a teaching aid and a lightweight planning tool. It won't replace professional power system software like PSS/E or PowerWorld, but it gives you fast, interactive feedback on questions like:

- Where are my bottlenecks at peak demand?
- What happens to utilization when solar output is high?
- If I have a fixed storage budget, where should I put it?

## Features

- **Four built-in grid topologies** — regional grid, metro area, radial distribution, and mesh network — plus a fully editable custom mode
- **Five operating scenarios** — peak demand, off-peak, high solar, high wind, and an extreme stress event
- **Interactive network visualization** with color-coded lines showing utilization in real time
- **Storage optimization** using a priority-based greedy allocator that distributes a kWh budget across substations to reduce congestion
- **Flow analysis charts** showing per-line utilization and generation vs. load balance
- **Save/load custom grids** as JSON files for reuse

## How to Run

Just open the HTML file in any modern browser:

```bash
firefox transmission_grid_optimizer.html
# or serve it locally if you prefer:
python3 -m http.server 8000
```

No dependencies to install. The tool loads vis-network and Plotly from CDN at runtime.

## How It Works

### Power Flow Model

The simulator uses a **simplified DC power flow** approach. For each node, it computes a net power value (generation minus load, adjusted for storage state). It then distributes flows across transmission lines using a proportional coupling model: when one node has surplus and a connected node has deficit, 40% of the available surplus routes across that line, capped at line capacity.

This is a deliberate simplification. A full AC load flow would track reactive power, voltage angles, and phase relationships — this model ignores all of that. For strategic planning purposes (finding approximate congestion hotspots, sizing storage investments) the DC approximation gives useful directional results with much less complexity.

Scenario multipliers adjust generation and load at simulation time:

| Scenario | Generation | Load | Solar | Wind |
|---|---|---|---|---|
| Peak | 90% | 120% | 30% | 50% |
| Off-peak | 100% | 60% | 0% | 70% |
| High Solar | — | — | 95% | 30% |
| High Wind | — | — | 20% | 90% |
| Extreme Event | 60% | 130% | — | — |

Congestion is flagged when a line exceeds 95% of its rated capacity.

### Storage Optimization

The optimizer runs a **greedy priority-based allocation**. For each substation, it calculates a congestion impact score based on how many overloaded lines it touches and their average utilization. Substations are then ranked by priority and allocated storage from the total budget in order, with a cap of 40% of the total budget going to any single location and a minimum threshold of 10 MWh per site.

After allocating storage, the simulation reruns and compares the before/after metrics: average utilization, number of congested lines, and overall grid efficiency score.

### Visualization

Transmission lines are colored by utilization:

- **Green** — below 50%
- **Yellow** — 50–80%
- **Orange** — 80–95%
- **Red** — over 95% (congested)

Line width scales with actual MW flow. Node colors indicate type: green for generation, red for load/data centers, yellow for substations with storage allocated.

### Custom Grid Builder

Selecting "Custom" topology enables an edit mode where you can:

- Add generation, load, or substation nodes by clicking the canvas or right-clicking
- Draw transmission lines between nodes and set their capacity
- Edit or delete any node or line
- Save the grid to a JSON file and reload it later

## Limitations and Known Simplifications

- **No reactive power or voltage modeling** — pure DC approximation only
- **No transmission losses** — line resistance and distance are not factored in
- **Instantaneous response** — no dynamic transients, oscillations, or settling time
- **Proportional flow distribution** — the 0.4 coupling coefficient is a heuristic, not derived from actual impedances
- **Storage modeled at constant 0.5C discharge rate** — does not account for state-of-charge dynamics or degradation
- **Greedy optimizer is not globally optimal** — a full mixed-integer program would find better allocations, especially under tight budgets

## Future Work

**More realistic power flow.** The biggest physical upgrade would be replacing the proportional heuristic with a proper DC load flow using the B-matrix (nodal susceptance). This would give impedance-weighted flows that respect Kirchhoff's laws and allow meaningful sensitivity analysis.

**AC power flow.** Adding reactive power and voltage magnitude tracking would open up voltage stability analysis and reactive compensation studies (capacitor banks, STATCOMs).

**Time-series simulation.** Rather than a single operating point per scenario, a 24-hour or seasonal simulation would let you track storage state-of-charge over time and evaluate cycling behavior and round-trip efficiency.

**N-1 contingency analysis.** Standard reliability practice is to check that the grid survives the loss of any single element. An automated N-1 sweep would identify which line or generator outages cause cascade failures.

**Storage dispatch optimization.** Replace the static 0.5C discharge assumption with an optimal dispatch that decides hour-by-hour when to charge and discharge each storage unit based on prices or congestion signals.

**Cost modeling.** Storage and transmission upgrade costs could be modeled more precisely, including capital cost per kWh, O&M, and expected lifetime, to compute actual levelized cost of avoided congestion.

**Multi-period investment planning.** Given a multi-year budget and forecast load growth, find the optimal sequence of storage and line upgrades over time.

**Import from standard formats.** Supporting CIM (Common Information Model) or MATPOWER case files would let users run the tool against real published grid datasets rather than hand-built topologies.

**Mobile layout.** The current UI requires roughly 1400px of width. A responsive layout would make it usable on tablets.
