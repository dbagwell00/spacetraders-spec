# SpaceTraders Client — The Charting Economy and the Expansion Lanes

**An amendment to `SPEC.md`, and a correction to one of its decisions.** Like
`SPEC-AGGRESIVE.md` this changes *policy*, not infrastructure: every ownership
rule, claim protocol, reservation door, rate limit and reconcile discipline in
`SPEC.md` stands unchanged and is load-bearing here.

**Status:** policy amendment, complete. Where this document and `SPEC.md`
disagree on a *policy* question, this document wins for the sections named in
§0.2. Where they appear to disagree on an *invariant*, `SPEC.md` wins and this
document has a defect.

**Sources.** This document is derived from the source code of the two agents
that beat us, read directly, plus measured outcomes from our own reset:

- `~/spacetraders-whater` — Excedrin's **WHATER**, #1 this reset (693.8M
  credits, 18,551 charts). Chiefly `docs/adaptive-global-expansion-strategy.md`
  and the `strategy_*_probe_*` / `warp_explorer_missions` modules.
- `~/spacetraders-whyando` — **WHYANDO**, #2 this reset (612.7M credits).
  Chiefly `src/agent_controller/fleet.rs`, `src/agent_controller/exploration.rs`,
  `src/ship_config.rs`.
- `~/spacetraders` — **TURKEYBOI1**, our own Python agent, 8th (94M), whose
  `events` table is the measurement in §1 and whose `PathfinderController`
  already implements much of §2.1.

---

## 0. What this changes and why

### 0.1 The thesis

> **The map is the product. Charting is an income line that pays for building
> it, and it out-earns our trade arm by nearly three orders of magnitude per
> API call.**

`SPEC.md` is a specification for trading well in one system. Everything in it
that touches the wider galaxy — the frontier controller, the reachability
walk, the probe census's frontier class — exists to serve trade in systems we
have *settled*. That is a coherent design, and it is not the design that wins.

Two agents win this game by two different routes, and **both begin with the
same move that we do not make at all**: fan a dedicated pool of hulls across
the jump-gate network as fast as the network can be revealed.

- **WHATER monetizes the map directly.** 18,551 charts this reset; its charting
  report (`charting_report.py`) accounts `chart_income` as a first-class line.
- **WHYANDO monetizes the map indirectly.** It is #2 on credits and **does not
  appear in the top five of the charts board at all**: it uses the network to
  find high-`P(T5)` systems and trades them out of the faction capital
  (`fleet.rs`, one `t5_trader` slot per reachable high-T5 system, capped at 25).

We do neither, and the reason is a decision recorded in `SPEC.md` §3.2b step 3:

> *"The dedicated **explorer (charter) role is retired**: charting is now a
> **reflex** — every ship charts whatever uncharted ground it already stands on
> — so the desired charter count is pinned to 0."*

The reflex was never implemented. `st_steps` has a `chart` step; **nothing in
the agent has ever emitted one**. So the arm was retired in favour of a
behaviour that does not exist, and the result is an agent that has submitted
**zero charts in its lifetime** while sitting on the same rate limit as the
agents above.

### 0.2 Sections of `SPEC.md` amended

| `SPEC.md` section | Amended by | Nature of the change |
|---|---|---|
| §3.2b Fleet, step 3 | §4 | The charter role is **un-retired**. Charting is a lane with an owner and a standing slot count, not a reflex nobody implemented. |
| §1.4 Component Inventory | §3 | New owner: the **Charting controller**. |
| §3.2d Probe census | §5.1 | The probe-per-market rule is scoped to the **starter system**. Galaxy-wide it is replaced by retention rules (§2.3). |
| §3.2e Frontier yield | §5.2 | Becomes measurable for the first time: it has always specified the already-charted response as its only input, and nothing ever produced one. |
| §3.3a Expansion | §5.3 | Procurement may buy at the frontier and at faction capitals, not only at home. |
| §3.2a Capital | §5.4 | The charting arm is funded as an income line, not as speculative overhead. |
| §3.2c Exploit | §5.5 | Settlement is admitted against what it displaces, including charting. |

`SPEC-AGGRESIVE.md` stands; this document supplies the *economic* reason its
land grab is correct, and replaces its implicit assumption that a probe's value
is market coverage.

### 0.3 What is explicitly NOT changed

- Every invariant listed in `SPEC-AGGRESIVE.md` §0.3, unchanged and restated by
  reference: claim protocol, generation guard, reservation door, the dual-bucket
  limiter, write-through persistence, reset boundaries, one convergent move per
  probe, defer-on-transit, full-path antimatter affordability, single ownership.
- **Jump gates ARE charted** *(corrected 2026-09-25)*. This said they never
  are -- "charting a gate publishes the graph to rivals for nothing" -- and
  every premise was false: a gate chart pays 10,000 credits live; a charted
  gate answers `/jump-gate` remotely forever, an uncharted one only with a
  ship present; 2,334 of 2,746 known gates were already charted by others
  (reset 2026-09-20); whater's relay counts charted gates, and TURKEYBOI1's
  own config calls its gate exclusion a reversible doctrine while its arrival
  reflex charts gates anyway. Exclusions are policy (§2.2); there is no gate
  invariant.
- The **rate limit is the binding resource**, not credits and not hulls. Every
  lane below is specified as a service level against it.

---

## 1. The measurement

This section is evidence, not policy. Every number is from our own cluster.

### 1.1 What a chart pays

`events` where `action='chart'`, reset `2026-09-13`, agent TURKEYBOI1:

```
attempts                 3,130
already charted          1,653   (52.8% — pays nothing)
mean credits / attempt      38,968
p50 / p90                    0 / 316,040
largest single chart      8,193,198
distinct systems               162
total                  121,970,569
```

A chart attempt is **one API call**. It pays nothing more than half the time,
and the mean across every attempt — successes and wasted calls together — is
**38,968 credits per call**.

Our Erlang agent's trade arm, from `/live/budget` on the same day:

```
arm_credits_per_call = { trade: 68.77 }
```

**A chart attempt is worth about 570 trade calls.** Both spend the same
2.44 req/s.

### 1.2 Corroboration across agents

Leaderboard, this reset, the five agents on the charts board:

| agent | charts | credits | credits / chart |
|---|---|---|---|
| WHATER | 18,551 | 693.8M | 37.4k |
| MOOSBEE | 13,249 | 439.8M | 33.2k |
| CHAMLIS | 8,457 | 420.9M | 49.8k |
| CHURT-LYNE | 6,940 | 233.3M | 33.6k |
| MAWHRIN-SKEL | 5,865 | 227.5M | 38.8k |
| **GOBBLICIDE (us)** | **0** | **2.1M** | — |

For every agent on that board, charts times the going rate accounts for
essentially the whole credit total. Two independent historical samples agree:
WHATER's own notes record 24,218 charts for 825.7M in the 2026-07-12 reset
(34,094 each); our Python agent's `config.py` records chamlis at 7,729 charts
for 382M (49.4k each).

**WHYANDO is the control.** It is second on credits with no place on the charts
board at all — proof that the map can be monetized by trade instead. It still
runs the same 20-hull charting pool to reveal the network; for it, the charts
are the means and the T5 systems are the end.

### 1.3 What the reference agents actually run

Read from their code, not inferred:

| | pool | size | purpose |
|---|---|---|---|
| WHATER | topology vanguard | ~50 probes | *"traverse every connected jump gate in roughly 16 hours"* |
| WHATER | chart swarm | any hull that can chart | converts revealed graph into chart income |
| WHATER | warp explorers | 20 converted refining freighters | disconnected space; 227.4M in one reset |
| WHYANDO | `jumpgate_probe/0..19` | flat 20 | *"fan probes out across the jump-gate network to map the web of connections as quickly as possible"* |
| WHYANDO | `t5_trader/0..N` | ≤25 | one per reachable `P(T5) ≥ 0.5` system, bought **in the faction capital** |
| TURKEYBOI1 | `PathfinderController` | floor 8 | *"the galaxy land-grab … a pool the settlement census cannot see and cannot ration"* |
| **ours** | — | **0** | — |

Note the second half of TURKEYBOI1's comment, which is the same defect this
project hit independently: *"an arm allowed to reach zero can never generate
the evidence that would grow it — that is exactly how the arm sat at 0 hulls
for 19h with 1,137 uncharted systems reachable."* Its answer is a **floor**,
and it cites WHYANDO's flat 20 as the same reasoning.

---

## 2. The lanes

Adapted from WHATER's *Concurrent Service Lanes*. The lanes run **concurrently
with protected service levels**. There is no global phase flag that turns a lane
off; funding shifts continuously. A lane that is starved must be visibly
starved (§7), never silently absent.

### 2.1 Lane A — the topology vanguard

**Objective:** reveal the connected gate graph as fast as it can be revealed.

- A **dedicated pool**, owned by one controller, that the settlement census
  **cannot see and cannot ration**. This is the load-bearing structural rule:
  our census is a coverage planner, and a coverage planner will always spend the
  marginal hull on a market it can already see.
- **Standing slot count with a floor.** The floor exists because the arm's own
  ceiling is computed from per-hull measurements that do not exist at zero
  hulls. `SPEC.md`'s API-capacity governor has this exact self-lock and it has
  already fired in production (a coverage plan trimmed to its floor while
  thirteen probes sat idle). A floor is the only defence that does not require
  the evidence it is meant to produce.
- **Durable, non-overlapping route segments**, not nearest-frontier picks.
  WHATER is explicit: *"One-off nearest-frontier selection is insufficient at
  galaxy scale."* Unvisited suffixes return to a shared queue when an
  assignment fails.
- A vanguard ship **is not absorbed by local backlog**. It enters a system,
  takes the intel that system owes (gate connections, shipyard and market
  presence, traits), and moves on.
- **A system is covered for expansion when it has been visited and its intel
  taken** — not when a probe is parked in it. See §2.3.

**Sizing.** Start at WHYANDO's flat 20 with a floor of 8. WHATER's ~50 is the
measured shape of a mature run; it is a target to sweep toward, not an opening
constant.

### 2.2 Lane B — the chart swarm

**Objective:** convert reachable uncharted ground into credits.

- **Charting is not a role, it is work any hull can do.** WHATER: *"This is a
  charting fleet, not literally a probe-only fleet … Any hull that can reach the
  next useful waypoint and call the chart or scout endpoint is expansion
  capacity."* Our command frigate and haulers are faster than probes.
- **The reflex is real and mandatory**: a ship standing on an uncharted
  waypoint charts it before it leaves. This is what `SPEC.md` §3.2b already
  claims and what was never built. It is nearly free — the ship is already
  there — and it is the cheapest credits in the game.
- **Chart priority**, in order:
  1. waypoints needed to continue topology discovery;
  2. probe-selling shipyards, faction headquarters, other useful yards;
  3. markets and high-confidence economic systems;
  4. dense local bundles that are efficient to finish;
  5. the remainder by expected reward against travel cost and the probability
     it is still uncharted on arrival.
- **Exclusions are policy, and none is an invariant.** Jump gates are charted
  like any other ground (§0.3, corrected 2026-09-25). Asteroid bases and engineered asteroids are
  deprioritized by TURKEYBOI1 as low payout — but our own data shows plain
  `ASTEROID` waypoints paying 852k–1.1M, so asteroids as a class must not be
  excluded. Measure before excluding.
- **A wasted call is expected and priced in.** 52.8% of attempts pay nothing.
  The mean already includes them. An implementation that tries to avoid every
  wasted call will avoid the income too.

### 2.3 Lane C — retention, which is not coverage

**Objective:** keep presence only where presence pays.

WHATER is explicit that the galaxy-scale version of our probe doctrine is
wrong: *"The strategy should not literally park a probe in every visited
system."* Durable presence is retained at:

- probe-selling shipyards (a parked buyer avoids a relocation every time
  procurement reopens);
- faction headquarters and other structural hull sources;
- high-quality trade systems;
- important branching gates;
- anywhere a ship materially improves market freshness or procurement
  visibility.

**Ship purpose and hull type stay separate concepts.** A surveyor or siphon
drone left over from the gate build is a perfectly good retention anchor, and
`SPEC.md`'s post-gate disposition should be per ship: retain locally if it can
do useful chart/scout/refresh work; use it as an anchor if that avoids a
purchase; export it only if the gate is reachable on its fuel; sell it only when
no local assignment exists.

### 2.4 Lane D — adaptive settlement

Settlement is admitted **when it beats the opportunity it displaces**, never at
a fixed hour or system count. The displaced opportunity now has a price: a
trade call earning 68.77 credits against a chart call earning 38,968 is not a
close question, and the exploit arm must be able to lose that comparison.

A high-confidence, unusually profitable system may receive a trader early; a
wave of mediocre settlements stays deferred while charting is productive.
WHYANDO's rule is the concrete form: **one trader per reachable `P(T5) ≥ 0.5`
system, capped at 25, bought in the faction capital.**

### 2.5 Lane E — warp explorers (deferred, and stated)

Warp-capable explorers cover systems **outside** the connected gate component.
WHATER ran 20 converted refining freighters for 227.4M in one reset, payback
26.9 hours including charts.

The conversion is a module swap, not a purchasable hull: a `SHIP_EXPLORER`'s
warp drive is installed on a `SHIP_REFINING_FREIGHTER`, which loses refining;
the donor explorer becomes an 800-fuel non-warp hull that is still good gate-
connected charting capacity.

**This lane is deferred**, and it is recorded here so the deferral is a decision
rather than an omission. It requires module-swap support the agent does not
have. Nothing else in this document depends on it.

### 2.6 Lane F — horizon damping

Long-payback purchases and long relocations taper as the reset approaches.
`SPEC.md`'s finite-horizon governor already does this; it must price a chart
hull by §1.1's realized rate rather than by a trade-revenue payback test that a
charting hull can never pass.

---

## 3. The Charting controller (new owner)

**Owns:** *"which uncharted ground is worth a call, and who is going to make
it."* Nothing else may decide to chart, and nothing else may decide not to.

**Inputs** (structural only, per agent): the waypoint store's charted flags; the
gate graph and reachability; the frontier-yield signal; the vanguard's route
claims; ship positions and idleness.

**Outputs:** chart targets per ship; the standing vanguard slot count; the
realized chart ledger (attempts, successes, already-charted, credits).

**Invariants:**

1. **A chart attempt is recorded, whatever it returns.** Attempt, success,
   already-charted and credits all land in the ledger. The frontier controller
   has specified the already-charted response as its only input since it was
   written and has never received one; this is the producer.
2. ~~**Never a jump gate** (§0.3).~~ Jump gates are charted (§0.3, corrected
   2026-09-25): the chart pays, and it keeps the gate's edges queryable.
3. **The reflex is not rationed by this controller.** A ship standing on
   uncharted ground charts it. Routing *toward* uncharted ground is this
   controller's decision; charting ground already underfoot is not a decision.
4. **Route segments are claimed, durable, and non-overlapping.** A failed
   assignment returns its unvisited suffix to the shared queue.
5. **The arm has a floor** (§2.1) and the floor is unconditional while any
   reachable uncharted ground remains.

**Error conditions:** a chart refusal that is not "already charted" is logged
and the target is dropped, never retried in-pass; a vanguard ship lost mid-route
returns its suffix; a frontier-yield failure **fails open** (charting continues)
— `SPEC.md` §3.2e already specifies that bootstrap default and it is correct.

---

## 4. Amendment to §3.2b Fleet — the charter role returns

Step 3 of the fleet reconcile is replaced. The desired charter count is **not**
pinned to 0; it is asked of the Charting controller, which is the single owner
of the arm's size. Promotion draws only from untargeted spare sentinels — never
from a probe actually covering a system — and a controller failure **holds the
roster** rather than defaulting to zero.

The retired-role revert in step 2 must not sweep charters.

**Why the reflex alone is insufficient**, stated plainly because this is the
decision being reversed: a reflex charts what a ship happens to walk past while
doing something else. Our fleet walks past the same 84 waypoints of one system
forever. A reflex over a stationary fleet produces one burst of income and then
nothing, which is indistinguishable in a dashboard from a reflex that was never
implemented — and is exactly what it would have produced here.

---

## 5. Amendments to other controllers

### 5.1 Probe census (§3.2d)

The **one-parked-probe-per-market** rule is the *starter-system* pattern. It is
correct there — the home system is where trade actually happens, and WHYANDO
runs the identical rule inside its own starting system (`ship_config.rs` puts a
probe on every market within 200 units of the origin).

Galaxy-wide it is wrong, and it is what makes our census unable to release a
hull: a home system with 26 markets claims 26 probes forever, and every
frontier system is compared against that claim and loses.

**Change:** the `exploited` class applies to the **home system and settled
systems only**. Elsewhere, presence is the retention rule of §2.3, and the
census's job is to keep the floor there rather than to fill a per-market quota.

### 5.2 Frontier yield (§3.2e)

Unchanged in mechanism and **finally measurable**: §3 supplies the
already-charted samples it was specified to consume. Its fail-open bootstrap
default is reaffirmed.

### 5.3 Expansion (§3.3a)

**Faction headquarters are structurally known procurement anchors.** Their
shipyards reliably sell `SHIP_PROBE`, `SHIP_EXPLORER` and
`SHIP_REFINING_FREIGHTER`. They are prospective targets by construction, not
discoveries to be stumbled on — though a known listing is not a fresh quote,
and the yard must still be reached and scouted before its offer is actionable.

**Buy near the work.** *"A cheap distant ship is not cheap after relocation API
calls, antimatter, and lost chart time."* WHYANDO buys its T5 traders in the
capital and parks a purchaser probe at that yard to price it.

**Every purchase carries a work packet**: role, source yard, destination claim,
expected useful start time, expected systems and charts, expected API calls,
expected antimatter and travel time, expected productive lifetime, and a release
condition. A purchase is justified when it *reduces the completion time of
valuable work* — not when a budget happens to be open.

**Buying slows** when the productive API queue stays saturated, when
ready-to-action delay rises without higher throughput, when idle capacity
accumulates, or when a recent purchase cohort did not improve throughput.

### 5.4 Capital (§3.2a)

The charting arm is funded as an **income line**. `SPEC-AGGRESIVE.md` §6 argues
a probe is a stock asset that cannot pass a revenue-payback test; for a charting
hull that argument is unnecessary — it has a measured revenue rate (§1.1) and
passes an ordinary payback test comfortably. A probe at 24,000 credits repays
itself in **one successful chart**.

### 5.5 Exploit (§3.2c)

Settlement competes against charting for the same API lane and must be able to
lose. The comparison is per-call and both sides are measured: this controller
already publishes its realized yield, and §3 now publishes the other side.

---

## 6. Retirement of the home fleet

WHYANDO retires its entire starting-area fleet — home economy, static probes,
logistics haulers, command ship — once the faction capital is reachable **and**
a capital trader is actually earning. The unassigned ships self-scrap.

The gating detail is worth copying exactly: retirement waits on an **earning**
trader, not merely on the purchaser probe existing, *"so retiring on it would
scrap the home fleet's money-makers during the gap before any trader is
earning."*

**This is specified but not scheduled.** It is the correct end state and it is
dangerous to implement before the lanes above work, because an agent that
retires its home fleet without a working replacement has no economy at all.

---

## 7. Telemetry

A lane that cannot be seen starving will starve. Published per pass:

- **charting**: attempts, successes, already-charted rate, credits, credits per
  attempt, charts per hour, distinct systems charted;
- **vanguard**: slots wanted / held / floored, route segments claimed /
  unclaimed / returned, systems first-visited per hour;
- **lane service**: API calls per lane against capacity, and the queue delay
  each lane sees;
- **comparison**: credits per API call, per lane, side by side. This single row
  is what makes §1.1 an ongoing fact rather than a one-off measurement.

The **indecision audit** of `SPEC-AGGRESIVE.md` §9 applies to every lane here:
"the planner did not think it was worth it yet" is a reportable defect, and the
binding constraint must be named from a closed list.

---

## 8. Acceptance criteria

1. The agent submits charts. Non-zero within an hour of a cold start.
2. Charting credits per API call are published beside trade's, per §7.
3. The vanguard holds its floor with reachable uncharted ground remaining, and
   the census cannot trim it.
4. A vanguard ship visits systems it does not settle, and does not park.
5. The already-charted rate reaches the frontier-yield controller.
6. No jump gate is ever charted.
7. A probe is bought at a yard that is not home when the work is not at home.
8. Chart attempts continue while trade continues; neither lane is a phase.

---

## 9. Open questions, deliberately not decided here

- **The vanguard's steady-state size.** 8 (floor) / 20 (WHYANDO) / ~50 (WHATER)
  are three measured points on somebody else's curve. Ours must be swept.
- **Whether asteroids belong in the swarm's priority order.** Our data says
  they pay; TURKEYBOI1 deprioritizes them. One of these is wrong for our fleet.
- **Warp explorers** (§2.5) and **home-fleet retirement** (§6): specified,
  not scheduled.
- **Whether charting or T5 trading is the better end state for us.** WHATER and
  WHYANDO prove both work. This document deliberately builds the shared
  prerequisite — the revealed network — and defers the choice.
