# AI Hangman

A one-screen Hangman game with an AI hint engine. No frameworks, no libraries, no build step —
the whole game is a single `index.html` file.

---

## Problem statement

Hangman is normally a game of blind guessing. The player gets no feedback about how close the
remaining possibilities are, no indication of which letter would be most useful next, and once a
few guesses go wrong there is no way to recover. Most simple Hangman implementations are also
static: they pick a word, draw the figure, and stop there.

This project builds an **AI-assisted Hangman** in which a small rule-based inference engine

1. stores a knowledge base of words and categories,
2. eliminates every word that contradicts the player's guesses (constraint-based reasoning),
3. recommends the most informative unguessed letter using a frequency heuristic, and
4. shows its own certainty through a live confidence meter.

The aim is to demonstrate the core ideas behind classical AI — knowledge representation, search
over a candidate space, and heuristic decision-making — in a small, fully explainable application
that can be presented in a few minutes.

---

## Objectives

- Build a complete, playable Hangman game in the browser using only HTML, CSS and JavaScript.
- Represent the game knowledge base as a dictionary of words with categories.
- Filter that dictionary after every guess so only consistent words remain.
- Suggest a letter that maximises the number of candidate words it covers.
- Visualise the AI's reasoning with candidate counts, a confidence bar and hint explanations.
- Keep the project minimal: one file, no installation, no network calls.

---

## Features

| Feature | Description |
| --- | --- |
| Random word selection | 53 words, 5–8 letters, four categories, no repeats back-to-back |
| Hangman drawing | SVG gallows with six body parts revealed one per wrong guess |
| On-screen keyboard | 26 letter keys with correct / wrong colour states |
| Physical keyboard | Type any letter to play; Enter restarts after a game ends |
| AI hint | Three hints per word, each revealing the most informative letter |
| AI confidence meter | Updates after every guess from the number of remaining candidates |
| Lives | Six wrong guesses allowed, shown as bars and as a counter |
| Scoreboard | Wins, losses and win streak saved in `localStorage` |
| Result screen | Shows the word, hints left and how far the AI had narrowed the search |
| Responsive layout | Two-column on desktop, stacked layout below 680 px |

---

## How the AI works

### 1. Knowledge base

Every word in `WORDS` carries its category. The engine only considers words with the same number
of letters as the hidden word, which is the first and cheapest filter.

### 2. Constraint filtering

After each guess, a candidate survives only if it agrees with everything the player knows:

```text
candidates = all words of the same length where:
    for every position i:
        if word[i] is already revealed, candidate[i] must be the same letter
    candidate must NOT contain any letter that was guessed and found to be wrong
```

This is classic constraint satisfaction: each guess shrinks the search space.

### 3. Hint heuristic

For a hint, the AI looks at the unguessed letters that are actually present in the hidden word and
picks the one that occurs in the most candidates:

```text
hint = the unguessed letter L in the word that maximises
       count of candidates containing L
```

A letter that appears in most remaining candidates removes the most uncertainty, so it is the most
useful letter to reveal. Ties are broken alphabetically for predictable behaviour.

### 4. Confidence

```text
confidence = 100 × (1 − (candidates − 1) / (pool − 1))
```

`pool` is the number of dictionary words with the same length. With every same-length word still
possible the meter reads 0%; when only one candidate remains it reads 100%. It is a heuristic
measure of how narrow the search has become, not a probability.

### 5. Guardrails

The hint only chooses letters from the hidden word, so **the AI can never reveal a wrong letter**.
That makes the hint "oracle-assisted", which is why the player gets only three per game. A hint
never costs a life, and the revealed letter is added to the guessed set as a correct letter
rather than counting as a wrong guess.

### Complexity

Let `N` be the dictionary size, `L` the word length and `A` the alphabet size.

- Candidate filtering: `O(N × L)` per guess (the dictionary stores all lengths, then length is checked).
- Choosing a hint: `O(A × N × L)` worst case. With `N = 53` this is effectively instant.

### Possible improvement

The filter checks position matches and wrong letters, but not letter counts. A candidate like
`BEVEL` can survive next to `PIXEL` after an `E` guess even though it contains an extra `E`.
Adding a multiset comparison would make the candidate set exact. This is noted in the limitations.

---

## Tech stack

| Layer | Choice | Why |
| --- | --- | --- |
| Structure | HTML5 | Semantic, accessible markup |
| Styling | CSS (custom properties, grid, flexbox, one media query) | No framework needed for a single screen |
| Logic | JavaScript (ES6) | DOM updates, Set/Array operations, no dependencies |
| Graphics | Inline SVG | Crisp hangman drawing, each part toggled with a class |
| Storage | `localStorage` | Tiny scoreboard, survives a refresh, no server |

---

## Run it

**Simplest:** double-click `index.html`.

**Recommended (keeps `localStorage` and the preview reliable):**

```bash
python3 -m http.server 4173
```

Then open <http://127.0.0.1:4173/index.html>.

No install, no build, no keys, works offline.

---

## Demo script (about 90 seconds)

| Time | Action | What to say |
| --- | --- | --- |
| 0:00 | Open the page | "Single HTML file. No frameworks, no build step — the AI lives in about forty lines of JavaScript." |
| 0:10 | Guess `E` | "Correct guesses fill every position. The AI panel counts how many dictionary words still fit." |
| 0:25 | Guess `Z` (or any letter not in the word) | "A wrong guess costs a life and draws a part. Six wrong guesses ends the game." |
| 0:35 | Click **AI Hint** | "The AI filters all words consistent with my guesses, then reveals the letter that appears in the most candidates. It also tells me how many candidates were covered." |
| 0:50 | Keep guessing to win | "The confidence meter rises as the candidate list shrinks; at one candidate it reads 100%." |
| 1:05 | Result screen → **Play Again** | "The result screen shows the word, the lives and hints left, and the AI's narrowed search." |
| 1:20 | Point at the header | "Wins, losses and streak persist between sessions in localStorage." |

Key line for the presentation: **the AI is an inference engine, not a trained model — it reasons
over a candidate word list using constraints and a frequency heuristic.**

---

## Project structure

```text
AI project/
├── index.html   # markup, styles, game logic and AI engine
├── README.md    # problem statement, design and demo notes
└── viva.md      # questions and answers for the viva
```

`window.aiHangman` exposes `guess`, `aiHint`, `newGame` and `getState()` so the game can be
inspected from the browser console, which is useful during a demo or for automated checks.

---

## Testing

Checked in a Chromium-based browser at 473 px and 1280 px widths.

| Test | Expected | Result |
| --- | --- | --- |
| Correct guess | Every matching blank fills, life count unchanged | Pass |
| Wrong guess | Life lost, one SVG part drawn, key turns red | Pass |
| Repeated guess | Ignored, no extra life lost | Pass |
| Six wrong guesses | Loss screen with the word and "0 lives left" | Pass |
| All letters guessed | Win screen, wins and streak increase | Pass |
| Hint | Reveals a correct letter, hint count drops, explanation shown | Pass |
| Hints exhausted | Button disabled after three hints | Pass |
| New Word | Word, drawing, keyboard and hints reset | Pass |
| Play Again | Overlay closes and a new word starts | Pass |
| Refresh page | Scoreboard restored from `localStorage` | Pass |
| Physical keyboard | Typing a letter guesses it | Pass |
| Narrow viewport | Layout stacks, keyboard wraps to 7 columns | Pass |
| Console | No errors or failed requests | Pass |

---

## Limitations

- Dictionary is small (53 words) and English-only; the AI is only as good as this list.
- The hint is oracle-assisted: it is guaranteed correct, so hints are capped at three.
- Letter multiplicities are not compared during filtering, so a few extra candidates can survive.
- Single player, no server, no accounts, no leaderboard.
- The AI cannot understand meaning — it cannot say "it is a red fruit"; it only reasons over letters.

## Future scope

- Semantic hints generated by an LLM ("a celestial body", "used in computing").
- Exact filtering with letter counts and a larger, indexed dictionary.
- Entropy-based hint selection instead of a simple frequency count.
- Difficulty levels that pick words the player is statistically likely to fail.
- Two-player mode or an AI that guesses while the human picks the word.
- Accounts, leaderboard and shareable results.
