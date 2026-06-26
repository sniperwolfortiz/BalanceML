# BalanceML — On-Device Reinforcement Learning Minigame

A complete Android app (Java, no external ML libraries) where a **Deep Q-Network (DQN)** 
agent learns entirely on-device to keep a box balanced on top of a moving platform.

---

## 🎮 What It Does

- A **cyan platform** moves up and down in a Lissajous sine pattern
- An **orange box** must stay on top of it
- A **DQN agent** controls the box (UP / STAY / DOWN), learning purely from reward signals
- Training happens **live on your phone** — you watch the agent go from random flailing to 
  precise tracking in real-time

---

## 🧠 ML Architecture

### Algorithm: Deep Q-Network (DQN)
| Component | Detail |
|---|---|
| Network | 5 → 64 → 64 → 3 (ReLU hidden, linear output) |
| State space | boxY, targetY, delta, boxVelocity, targetVelocity (normalised) |
| Action space | UP / STAY / DOWN |
| Training | Experience replay (10k buffer), batch=64, γ=0.95 |
| Exploration | ε-greedy, ε: 1.0 → 0.05 (decay 0.9995/step) |
| Target net | Hard sync every 200 steps |
| Optimiser | SGD with lr=1e-3 (vanilla backprop, no external libs) |

All ML code is **pure Java** — no TensorFlow Lite, no ONNX, no dependencies.

---

## 📁 Project Structure

```
BalanceML/
├── app/src/main/java/com/balanceml/
│   ├── ml/
│   │   ├── NeuralNetwork.java   — Forward pass + backprop from scratch
│   │   ├── ReplayBuffer.java    — Circular experience replay
│   │   └── DQNAgent.java        — ε-greedy policy, training loop, target net
│   ├── game/
│   │   ├── GameEngine.java      — Physics sim (pure Java, no Android deps)
│   │   └── GameView.java        — Custom View: game loop + rendering
│   └── ui/
│       └── MainActivity.java
└── app/src/main/res/
    └── layout/activity_main.xml
```

---

## 🚀 Building & Running

### Requirements
- Android Studio Hedgehog or later
- Android SDK 26+
- A physical Android device or emulator (API 26+)

### Steps
1. Open **Android Studio**
2. **File → Open** → select the `BalanceML/` folder
3. Wait for Gradle sync (no external dependencies to download)
4. Click **Run ▶** on a connected device

### Minimum device spec
- Any Android 8.0+ phone (2017+)
- ~10MB RAM for the model (tiny)
- Works on low-end devices — no GPU required

---

## 📊 What You See (HUD)

| Element | Meaning |
|---|---|
| `EP N` | Episode number (each episode = 8 seconds of game time) |
| `STEPS N` | Total training steps taken |
| `ε=0.xxx` | Current exploration rate (starts at 1.0, decays toward 0.05) |
| `SCORE` | Score this episode (reward accumulation) |
| `BEST` | Best score across all episodes |
| `BALANCE` bar | How well the box is sitting on the platform right now |
| Q-value bars | The agent's estimated value for each action |

---

## 🔧 Tuning Parameters

Edit constants in the source files:

**`DQNAgent.java`**
```java
GAMMA       = 0.95f   // discount factor
LR          = 1e-3f   // learning rate
EPS_DECAY   = 0.9995f // exploration decay per step
BATCH_SIZE  = 64      // replay batch size
TARGET_SYNC = 200     // steps between target network sync
```

**`GameEngine.java`**
```java
BOX_SPEED    = 220f  // max box speed (px/s)
TARGET_AMP   = 220f  // target oscillation amplitude
TARGET_SPEED = 1.4f  // target oscillation frequency
```

---

## 📈 Expected Learning Curve

| Phase | Episodes | Behaviour |
|---|---|---|
| Random | 0–50 | Box flies around randomly |
| Early learning | 50–200 | Box starts tracking roughly |
| Improving | 200–500 | Box follows with some lag |
| Competent | 500+ | Box stays on target most of the time |
| Expert | 1000+ | Tight tracking, even with increased difficulty |

---

## 🔌 Integrating Into Your Game

`GameEngine` has **zero Android dependencies** — copy it into your game and call:

```java
GameEngine engine = new GameEngine();
DQNAgent agent = new DQNAgent();

// Each frame:
float[] state = engine.getState();
int action = agent.selectAction(state);    // agent picks move
engine.tick(deltaTime, action);             // physics step
float reward = yourRewardFunction();
agent.step(state, action, reward, engine.getState(), episodeDone);
```

---

## 📄 License
MIT — free to use, modify, and embed in your own game.
