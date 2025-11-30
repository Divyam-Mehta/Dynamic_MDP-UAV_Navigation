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

## 🚀 Action Space

> **The Action Space consists of 5 distinct actions – {F, L, R, U, D}**

| **Symbol** | **Name**           | **Description (Intended Motion)**          |
|------------|-------------------|---------------------------------------------|
| **F**      | Forward           | Move along **+X** direction                 |
| **L**      | Left-Forward      | Move along **+X** and **+Y** directions     |
| **R**      | Right-Forward     | Move along **+X** and **−Y** directions     |
| **U**      | Lift Up-Forward   | Move along **+X** and **+Z** directions     |
| **D**      | Lift Down-Forward | Move along **+X** and **−Z** directions     |

---

### 🔀 Transition Space

The UAV operates under **stochastic dynamics**.  
For every state **s = (x, y, z)** and action **a**, the transition model produces **three possible successor states**:

- **Primary successor** `s₁` with probability **0.98**
- **Secondary successor** `s₂` with probability **0.01**
- **Tertiary successor** `s₃` with probability **0.01**

Let:

- `X+` = x + 1  
- `Y+` = y + 1  
- `Y−` = y − 1  
- `Z+` = z + 1  
- `Z−` = z − 1  

> **Note:** Boundary conditions clamp values to the grid limits.

| **Action** | **Primary Successor s₁ (≈ 98%)** | **Secondary s₂ (≈ 1%)** | **Tertiary s₃ (≈ 1%)** |
|-----------|----------------------------------|-------------------------|------------------------|
| **F**     | (X+, y, z)                        | (X+, Y+, z)             | (X+, Y−, z)            |
| **L**     | (X+, Y+, z)                       | (X+, y, z)              | (X+, y, z)             |
| **R**     | (X+, Y−, z)                       | (X+, y, z)              | (X+, y, z)             |
| **U**     | (X+, y, Z+)                       | (X+, y, z)              | (X+, y, z)             |
| **D**     | (X+, y, Z−)                       | (X+, y, z)              | (X+, y, z)             |

Boundary Rule

If the UAV reaches any grid limit - **x = 19**, **y = 0**, **y = 19**, **z = 0**, or **z = 4**, the corresponding coordinate is **clamped**, meaning:

> The UAV **remains at the edge** in that dimension instead of moving outside the defined 3D space.
