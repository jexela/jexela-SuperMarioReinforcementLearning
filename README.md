# Super Mario Reinforcement Learning with Double Q-Learning and Dueling DQN

### Introduction

This university project, created for the course "Bildverarbeitung, Maschinelles Lernen und Computer Vision," presents a comparative study on Reinforcement Learning (RL) applied to *Super Mario Bros*, utilizing **Double Q-Network (DQN)** and **Dueling Q-Network (DDQN)** agents. Inspired by Richard E. Bellman’s principle of decomposing complex problems into manageable subproblems, our approach leverages Q-learning techniques to assess each agent's performance on levels 1-1 and 1-4, using the [`nes-py`](https://github.com/Kautenja/nes-py) emulator.


- **Level 1-1** provides the standard vanilla environment.
- **Level 1-4**, on the other hand, introduces more challenging dynamics, demanding more intricate skills, but shorter episodes.

A key feature of this work is the use of a large replay buffer of 300,000 experiences, combined with an epsilon-greedy strategy (decaying epsilon from 1 to 0.02) to retain diverse game states and capture long-term dependencies.

---

## Training Progress:

### Double QN - Level [1,1]

<table>
  <tr>
    <td>After Episode 4000:</td>
    <td>After Episode 8000:</td>
    <td>After Episode 13500:</td>
  </tr>
  <tr>
    <td><img src="https://github.com/jexela/SuperMarioReinforcementLearning/raw/main/DoubleLevel[1,1]Data/mario_episode_doubledqn_4000_11.gif" width="250" /></td>
    <td><img src="https://github.com/jexela/SuperMarioReinforcementLearning/raw/main/Double Level [1,1] Data/mario_episode_doubledqn_8000_11.gif" width="250" /></td>
    <td><img src="https://github.com/jexela/SuperMarioReinforcementLearning/raw/main/Double Level [1,1] Data/mario_episode_doubledqn_13500_11.gif" width="250" /></td>
  </tr>
</table>

### Dueling DQN - Level [1,4]

<table>
  <tr>
    <td>After Episode 12500:</td>
  </tr>
  <tr>
    <td><img src="https://github.com/jexela/SuperMarioReinforcementLearning/raw/main/Double Level [1,4] Data/mario_episode.gif" width="250" /></td>
  </tr>
</table>

### Double QN - Level [1,4]

<table>
  <tr>
    <td>After Episode 26500:</td>
    <td>After Episode 28500:</td>
    <td>After Episode 29500:</td>
  </tr>
  <tr>
    <td><img src="https://github.com/jexela/SuperMarioReinforcementLearning/raw/main/Dueling Level [1,1] Data/mario_episode_doubledqn_26500_14_success.gif" width="250" /></td>
    <td><img src="https://github.com/jexela/SuperMarioReinforcementLearning/raw/main/Dueling Level [1,1] Data/mario_episode_doubledqn_28500_14_success.gif" width="250" /></td>
    <td><img src="https://github.com/jexela/SuperMarioReinforcementLearning/raw/main/Dueling Level [1,1] Data/mario_episode_doubledqn_29500_14.gif" width="250" /></td>
  </tr>
</table>

### Dueling DQN - Level [1,4]

<table>
  <tr>
    <td>After Episode 20000:</td>
    <td>After Episode 26000:</td>
    <td>After Episode 33500:</td>
    <td>After Episode 40000:</td>
  </tr>
  <tr>
    <td><img src="https://github.com/jexela/SuperMarioReinforcementLearning/raw/main/Dueling Level [1,4] Data/mario_episode_duelingdqn_20000_14.gif" width="250" /></td>
    <td><img src="https://github.com/jexela/SuperMarioReinforcementLearning/raw/main/Dueling Level [1,4] Data/mario_episode_duelingdqn_26000_14.gif" width="250" /></td>
    <td><img src="https://github.com/jexela/SuperMarioReinforcementLearning/raw/main/Dueling Level [1,4] Data/mario_episode_duelingdqn_33500_14.gif" width="250" /></td>
    <td><img src="https://github.com/jexela/SuperMarioReinforcementLearning/raw/main/Dueling Level [1,4] Data/mario_episode_duelingdqn_40000_14.gif" width="250" /></td>
  </tr>
</table>


