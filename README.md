# Dynamic MDP for UAV Bird-Strike Avoidance

This repository implements a **dynamic Markov Decision Process (MDP)** to compute collision-avoiding trajectories for a **Unmanned Aerial Vehicle (UAV)** flying through a 3D airspace populated with multiple birds.

The UAV flies through a **20 × 20 × 5 discrete grid**, while a flock of birds moves with continuous velocities. At each time step, the algorithm:

1. Predicts the future positions of all birds (with observation noise),
2. Builds a **time-dependent MDP**,
3. Runs **policy iteration** to obtain an optimal policy for that time slice,
4. Executes the **most probable transition** for the UAV,
5. Propagates the real bird states forward with the continuous transition model.

The goal is to **reach the far boundary of the grid without colliding with any birds**, while minimizing control effort.

---

## 💡 Key Ideas

- **Dynamic MDP**: The environment is non-stationary. Bird positions change with time, so the reward function (and effective transition probabilities) depend on the current time step.
- **Discrete UAV states, continuous bird motion**:
  - UAV states are discrete grid cells `(x, y, z)`.
  - Bird states are represented by both continuous locations and discretized (floored) grid cells.
- **Bird prediction model**:
  - Uses noisy observations of bird locations and velocities to **predict future bird occupancy**.
- **Risk-aware reward function**:
  - Large negative reward for collisions.
  - Positive reward for reaching the goal boundary.
  - Shaping penalties for unnecessary movements to encourage efficient, smooth paths.
- **Stochastic UAV transitions**:
  - Each action leads to three possible successor states:
    - `new_state_1` with probability 0.98 (most likely),
    - `new_state_2` and `new_state_3` with probability 0.01 each.
- **Policy iteration loop**:
  - Alternates between policy evaluation and policy improvement until convergence at each time step.

---

## 🧱 Environment & State Space

### Grid World

