### Kepler-K1 vs Kepler-K2

Kepler-K1 could land the booster, but it didn't land it *cleanly*. Near the ground it would cut the engine, relight it, and float above the deck for a second or two while it hunted for the centre of the pad. It worked, but no real rocket would fly like that, and every moment spent hovering is fuel thrown away.

Kepler-K2 fixes that. It flies a true hoverslam: one ignition, one continuous burn, and a smooth slowdown all the way to the deck, with no throttle cuts and no loitering. Here's what changed, and why.

#### Core physics and control strategy

| Feature | **Kepler-K1** | **Kepler-K2** |
| :--- | :--- | :--- |
| **Throttle behaviour** | **Discontinuous / chattering.** Cut the engine (`throttle = 0`) whenever $V_z$ rose above target, then re-ignited. | **Continuous single burn.** Once lit, the engine never cuts off; it modulates smoothly within $[40\%, 100\%]$. |
| **Terminal descent profile** | Tended to stutter near the ground ($z < 10\text{ m}$), floating before settling. | **Continuous hoverslam.** A true SpaceX-style suicide burn following a continuous parabolic glideslope. |
| **Aloft lateral guidance** | Softer guidance gain (`pos_gain = 0.26`), so it carried horizontal offset deep into the final descent. | Aggressive aloft guidance (`pos_gain = 0.30`). Drift is nulled high up, so the booster enters its vertical flare already centred over the pad. |
| **Hover duration ($z \le 6\text{ m}$)** | Lingered near the deck for $\sim 1.5\text{ s}$ to $3.0\text{ s}$. | **Zero hover ($\sim 0.30\text{ s}$).** Touchdown happens on first contact, with only landing-leg compression. |
| **Fuel conservation** | Conserved $\sim 68\%$–$69\%$ of propellant. | **Conserves $\sim 72.2\%$ ($5,776.6\text{ kg}$ remaining)**, thanks to eliminating throttle transients and loitering. |

The two changes that matter most work together. Because K2 cleans up its sideways drift while it's still high up, it doesn't need to search for the pad at the bottom. And because it doesn't need to search, it never has to stop and hover.

#### How the training changed

Both versions learn by imitation, using DAgger: the agent flies, an expert flight computer shows what it *should* have done in the situations it actually got into, and those corrections are fed back into training. What changed between K1 and K2 is who the expert was and what it was shown.

1. **A better teacher.**
   - **K1** learned from the original flight computer, and picked up its habits, including the chattering throttle near the ground.
   - **K2** learned through **Multi-Condition DAgger**, from an upgraded flight computer that flies a continuous hoverslam. A cleaner expert meant a cleaner student.

2. **Harder, more varied flights.**
   - **K1** trained mostly on baseline drops with mild perturbations.
   - **K2** trained across an adversarial matrix of drops from $120\text{ m}$, $250\text{ m}$, $400\text{ m}$, $600\text{ m}$, and $800\text{ m}$, in Dryden continuous wind turbulence, with random initial pitch and yaw offsets.

3. **Targeted practice on its weak spots.**
   K2 added a **selective aggregation loop**. Any benchmark flight that landed more than $8\text{ m}$ off centre, or hovered for more than $0.05\text{ s}$, was flown again with expert corrections, and those corrections were injected into the training batches. In other words, K2 spent extra time practising exactly the landings it was getting wrong.

#### What a landing looks like

**Kepler-K1**

Drops rapidly $\rightarrow$ cuts throttle mid-descent when it senses high negative acceleration $\rightarrow$ re-ignites near the deck $\rightarrow$ wobbles and hovers for a few seconds searching for the pad centre $\rightarrow$ touches down.

**Kepler-K2**

Drops in freefall while TVC aligns the gimbal $\rightarrow$ single-point ignition $\rightarrow$ continuous, monotonic deceleration straight toward $(0, 0)$ $\rightarrow$ legs deploy at $35\text{ m}$ $\rightarrow$ touches down at $-1.8\text{ m/s}$, with zero throttle cuts and no hovering.

If you have the sandbox running, the difference is easy to see for yourself: watch the last ten metres of each.
