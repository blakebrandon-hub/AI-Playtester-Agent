# 🤖 AI Playtester Agent

A generic AI playtester template for text-based, AI-narrated games. 

This project provides an automated "Player Agent" that interacts with your existing AI game backend. It methodically explores your game world, tests mechanics, tracks state changes via structured tags, detects anomalies/bugs, and generates a structured QA playtest report.

## 🏗️ Architecture

The system consists of two main scripts that work alongside your existing game server:

1. **`playtester_agent.py`** (The Brain): A Flask server (`:7001`) that acts as the playtester. It uses an LLM to decide on actions, talks to your game backend, parses state tags, and logs bugs.
2. **`playtester_cli.py`** (The Viewer): A command-line interface that connects to the agent, allowing you to watch the AI play your game in real-time with beautifully formatted terminal output.

```text
[ Your Game Backend ]       [ playtester_agent.py ]       [ playtester_cli.py ]
   (e.g., Port 5000)  <--->    (Flask - Port 7001)  <--->     (Your Terminal)
    (The Narrator)               (The Playtester)               (The Spectator)
```

## 🚀 Getting Started

### 1. Prerequisites
You will need Python 3.8+ and the following dependencies:
```bash
pip install flask flask-cors requests anthropic openai google-genai
```

### 2. Environment Variables
The playtester requires an LLM to make decisions. Set **one** of the following environment variables:
* `ANTHROPIC_API_KEY` (Defaults to `claude-3-5-haiku`)
* `OPENAI_API_KEY` (Defaults to `gpt-4o-mini`)
* `GEMINI_API_KEY` (Defaults to `gemini-2.0-flash`)

*Example (Mac/Linux):*
```bash
export ANTHROPIC_API_KEY="your-api-key-here"
```

### 3. Customizing the Template (Crucial Step)
This code is a **template**. Before running it, you must open `playtester_agent.py` and search for the `TODO` comments to adapt it to your specific game's logic.

*   **`DEFAULT_GAME_STATE`**: Define the variables your game tracks (e.g., HP, inventory, location).
*   **`parse_tags()` & `apply_tags()`**: Write regex to catch structured tags emitted by your narrator (e.g., `[HP: -10]`) and apply them to the game state.
*   **Prompts**: Update `NARRATOR_SYSTEM_PROMPT` (the rules for your game) and `PLAYTESTER_SYSTEM_PROMPT` (the persona and goals of the tester).
*   **`detect_anomalies()`**: Write custom logic to flag bugs (e.g., *“Player moved locations but no `[LOC: ...]` tag was emitted”*).
*   **Backend URLs**: Ensure `GAME_API_URL` points to your actual game server.

---

## 🎮 Usage

You need three terminal windows open to run the full stack:

**Terminal 1: Your Game**
Start your existing game/narrator backend (expected on port `5000` by default).

**Terminal 2: The Agent Server**
Start the playtester agent:
```bash
python playtester_agent.py
```

**Terminal 3: The CLI Watcher**
Watch the agent play! You have several run modes:

```bash
# Auto-play for 20 turns (default)
python playtester_cli.py

# Auto-play for a specific number of turns with a delay
python playtester_cli.py --turns 50 --delay 3.0

# Manual step mode (press ENTER to advance turn-by-turn)
python playtester_cli.py --step

# Generate a QA report from the currently active session
python playtester_cli.py --report-only
```

### CLI Step Mode Controls
When running in `--step` mode, you can use the following commands:
* `[ENTER]` - Agent takes the next turn
* `s` - View current internal game state, bug count, and note count
* `r` - Generate and view the final QA playtest report
* `q` - Quit the CLI

---

## 📡 Agent API Endpoints

If you want to build your own web-frontend for the playtester instead of using the CLI, `playtester_agent.py` exposes the following REST API on `http://localhost:7001`:

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `POST` | `/api/playtester/start` | Initializes a fresh playtest session. |
| `POST` | `/api/playtester/step` | Prompts the AI to take exactly one action. |
| `POST` | `/api/playtester/run` | Auto-runs `N` steps. Accepts JSON `{"turns": 5, "delay": 1.5}`. |
| `GET`  | `/api/playtester/state` | Returns the current parsed game state, bugs, and logs. |
| `POST` | `/api/playtester/report`| LLM evaluates the session and generates a structured QA report. |
| `POST` | `/api/playtester/reset` | Clears the session data. |

## 📝 Example QA Report Output

After a session, generating a report (`r` in step mode, or `--report-only`) will yield something like this:

```text
════════════════════════════════════════════════════════════════════════
  PLAYTESTER REPORT
════════════════════════════════════════════════════════════════════════
  Turns Covered:  20
  Bugs Detected:  2
  Notes Recorded: 4
────────────────────────────────────────────────────────────────────────
1. **Overall Impression**: The game feels atmospheric and responsive.
2. **Mechanics**: Combat works well, but the inventory system failed to 
   register the "Rusty Key" when I picked it up on Turn 12.
3. **Bugs**: 
   - [Missing LOC Tag] Movement to "Dark Cave" emitted no location tag.
   - HP dropped below zero without triggering a death state.
4. **Recommendations**: Ensure the narrator strictly follows the tag format 
   when items are picked up.
```
