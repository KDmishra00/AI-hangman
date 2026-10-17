# questions — AI Hangman

 The one-line summary at the bottom is the version.

---

## A. Project and problem

**1. What is your project?**
AI Hangman — a browser-based Hangman game with a rule-based AI assistant. It keeps a dictionary
of words, filters that dictionary after every guess, suggests the most informative letter as a
hint, and shows how confident it is in the remaining candidates. It is a single HTML file using
plain HTML, CSS and JavaScript.

**2. What problem does it solve?**
Normal Hangman is blind guessing: the player has no idea how close the answer is or which letter
is worth trying, and a bad start cannot be recovered. This project adds an inference engine that
tracks every word still possible, explains its reasoning, and helps a stuck player without
playing the game for them.

**3. Why is it called AI? Where is the AI?**
Because the program represents knowledge (a word list), reasons over it (constraint filtering),
and makes a decision under uncertainty (which letter to reveal) using a heuristic. That is
classical symbolic AI, not machine learning. The AI lives in `candidateWords()`,
`bestHintLetter()` and `confidenceFor()`.

**4. Which AI techniques are used?**
- Knowledge representation: dictionary of words with categories.
- Constraint satisfaction / generate-and-test: keep only words consistent with the guesses.
- Heuristic search: rank unguessed letters by how many candidates they cover and pick the best.
- Uncertainty display: a confidence measure derived from the size of the candidate set.

**5. Is it machine learning?**
No. There is no training data, no model, no weights, no learning step. The behaviour is written
as rules and is fully deterministic. ML would learn a pattern from examples (for example, which
letters humans guess first); this project encodes the reasoning directly.

**6. Why did you not use a trained model or an LLM API?**
Three reasons: the project must run offline during a presentation, an LLM would add cost and
latency for a problem that a few rules solve exactly, and a rule-based engine is explainable —
I can show the examiner exactly why a letter was chosen. An LLM would also be less reliable at
guaranteeing a correct hint.

**7. What are the objectives?**
Playable Hangman; a dictionary-based knowledge base; candidate filtering; an informative hint
with a visible explanation; a confidence display; and a minimal implementation with no
dependencies or build step.

**8. What is the scope and what is out of scope?**
In scope: one player, one word at a time, six lives, three AI hints, local scoreboard, responsive
layout. Out of scope: accounts, multiplayer, server, online leaderboard, semantic hints,
localisation.

---

## B. The AI and algorithm

**9. Explain candidate filtering with an example.**
Suppose the hidden word is `SANDWICH` and the player has guessed `A`. Every dictionary word of
length 8 must have `A` in the same position; wrong letters are also excluded. That leaves one
candidate. If the player had guessed a wrong letter such as `Z`, every word containing `Z` is
removed instead. Each guess is a new constraint, so the candidate set can only shrink.

**10. How do you choose the hint?**
I look only at unguessed letters that occur in the hidden word, count how many remaining
candidates contain each one, and pick the letter with the highest count. Ties are broken
alphabetically. In a demo run the word was `SANDWICH` and the AI revealed `A`, reporting
"found in 4 of 6 candidate words"; afterwards only one candidate remained and confidence was 100%.

**11. Why is frequency a good heuristic?**
A letter that appears in most remaining candidates splits the candidate set most evenly. If a
letter is in 10 of 12 words, revealing it tells the player about most of the possibilities; a
letter in 1 of 12 barely helps. This is the intuition behind information gain.

**12. What is information gain or entropy, and could you use it?**
Entropy measures the uncertainty of a distribution: `H = −Σ p·log₂p`. Information gain is the
reduction in entropy after a guess. A stricter solver would compute, for each candidate letter,
the expected split of the candidate set and choose the letter with the highest gain, even if the
guess can be wrong. My frequency count is a cheap approximation of the same idea.

**13. How is the confidence value computed? Is it a probability?**
`confidence = 100 × (1 − (candidates − 1) / (pool − 1))`, where `pool` is the number of
dictionary words with the same length. It reaches 0% at the start and 100% when only one candidate
remains. It is a heuristic measure of how narrow the search is, not a calibrated probability.

**14. Can the AI ever reveal a wrong letter?**
No. The hint chooses only from letters that exist in the hidden word, so every hint is correct by
construction. That is a deliberate design choice: a hint should help the player, not gamble. A
real solver would sometimes guess a letter that might be wrong to gain information faster.

**15. Isn't that "cheating" because the AI knows the answer?**
Yes — the hint is oracle-assisted, and I present it as an assistant rather than an opponent. To
keep the game fair, the player gets only three hints. The candidate filtering and the confidence
meter, however, are honest inference: they use only the letters that have already been revealed.

**16. Why only three hints?**
With unlimited oracle hints the AI could reveal the whole word and the player would not be
playing. Three hints give the player meaningful help while leaving most of the word to be solved,
and the counter makes the limit visible.

**17. What is the time and space complexity?**
Filtering is `O(N × L)` per guess, where `N` is the dictionary size and `L` the word length.
Choosing a hint is `O(A × N × L)` in the worst case, where `A` is the alphabet size (26). With 53
words it is instantaneous. Space is `O(N × L)` for the dictionary plus `O(N)` for a candidate
list.

**18. What if the dictionary had one million words?**
I would index words by length, store an inverted index from each letter to the words containing
it, and keep candidates as a set that is intersected after each guess. For hints, precompute
letter counts per length or maintain per-letter counters, so a hint becomes close to `O(A)`
rather than a full scan.

**19. Is the AI guaranteed to find the word?**
The filtering guarantees the hidden word is always among the candidates, so the information is
always correct. Finding it in six lives depends on the player and on how many hints are used;
the AI recommends good letters but does not force a win.

**20. Data structures used?**
`Array` for the dictionary and the candidate list, `Set` for guessed letters (constant-time
membership checks and automatic de-duplication), objects for state, and a `Map`-like object for
persisted stats. The DOM is updated from the state after every move.

---

## C. Implementation

**21. Why a single HTML file and vanilla JavaScript?**
The project is deliberately minimal: no install, no build, no dependency risk, and it opens on
any laptop for the presentation. For a single-screen game the framework would add more
configuration than code. For a larger product I would move to React and a server.

**22. How is state managed?**
One `state` object holds the word, category, guessed letters, wrong count, hints left, hinted
letters and the game-over flag. Every action changes that object and then calls `render()`, which
rebuilds the word slots, keyboard classes, lives, hangman parts and AI panel from it. The UI is a
pure function of the state, which makes behaviour predictable and easy to test.

**23. How is a win or a loss detected?**
Win: every character of the word is in the guessed set
(`word.split("").every(ch => guessed.has(ch))`). Loss: `wrong >= 6` after a wrong guess. Both set
`over = true`, update the scoreboard and show the result overlay.

**24. How is the hangman drawn?**
As inline SVG: the gallows frame is always visible and six body parts (head, body, two arms, two
legs) start at `opacity: 0`. Each wrong guess adds a class that fades in the next part. SVG is
crisp at any size and each part is a separate element, so no image assets are needed.

**25. How does keyboard input work?**
The on-screen keys are buttons with a click listener. A `keydown` listener on `document` handles
physical typing with a regular expression for letters; when the result overlay is open, Enter
restarts instead. Disabled keys are ignored by the guard at the start of `guess()`.

**26. How do you prevent invalid or repeated guesses?**
Every guess passes through `guess()`, which returns immediately if the game is over or the letter
is already in the `guessed` set. Buttons are also marked `disabled` for used letters, so the
keyboard state and the logic agree.

**27. What is localStorage and why use it?**
A synchronous key-value store in the browser that survives refreshes and holds strings only.
I save one JSON object with wins, losses and streak. Cookies would be sent with every HTTP
request and are limited to about 4 KB; `sessionStorage` would disappear when the tab closes.
`localStorage` is perfect for a small offline scoreboard. It is per-origin, so the file must be
opened the same way each time (file or localhost) for the score to persist.

**28. How did you make the layout responsive?**
CSS Grid for the two-column desktop layout (`240px 1fr`), flexbox inside the panels, and a single
media query at 680 px that switches to one column, shrinks the gallows, and changes the keyboard
from 13 to 7 columns. The viewport meta tag makes mobile browsers use the device width.

**29. What about accessibility?**
Semantic elements (`header`, `main`, `section`, `button`), `aria-label` on the drawing and on
every key, a result dialog with `role="dialog"` and `aria-modal`, visible focus on the Play Again
button, and information never conveyed by colour alone (keys also become disabled and the lives
are counted in text).

**30. What was the trickiest bug and how did you fix it?**
The candidate filter and the hint had to agree. Early on it was possible for a hint to reveal a
letter that was not in the hidden word, or for a candidate containing a wrong letter to survive.
The fix was to make a single `candidateWords()` function the only source of truth, and to choose
hint letters only from the hidden word. I also had to reset the drawing and overlay explicitly in
`newGame()`, otherwise the old SVG parts stayed visible after restarting.

---

## D. Testing and design

**31. How did you test the game?**
Manually in the browser and with scripted checks through the exposed `window.aiHangman` API:
guessing correct and wrong letters, repeating a guess, exhausting the hint limit, winning,
losing, restarting, refreshing for persistence, typing on the physical keyboard and resizing the
viewport.

**32. Which test cases matter most?**
Correct guess fills all positions; wrong guess costs exactly one life and one drawing part; six
wrong guesses triggers the loss screen with the right word; guessing every letter triggers the
win screen and increments the score; a hint never breaks the word and cannot be used after three;
New Word resets word, drawing, hints, keyboard and overlay; stats survive a refresh; no console
errors.

**33. What would break first if the project grew?**
The linear dictionary scan and the string-based word list. With a real dictionary I would need
indexing and a data file loaded asynchronously; with an LLM hint I would need a backend, API keys
and error handling, because a key cannot be shipped in client-side JavaScript.

**34. If you had one more week, what would you add?**
Entropy-based scoring for hints, a bigger indexed dictionary, difficulty levels chosen from the
player's past performance, and semantic hints ("a red fruit") instead of letter hints. Those
would make the AI feel intelligent rather than only informative.

**35. What did you learn?**
How to turn a reasoning process into code: represent knowledge, restrict the hypothesis space,
rank actions with a heuristic, and expose the reasoning to the user. I also learned to keep game
state in one place so the UI stays consistent, and to design a hint system that helps without
removing the challenge.

---

## E. Quick-fire web fundamentals

**36. Frontend or backend?**
Frontend only. Everything runs in the browser; there is no server, database or API call.

**37. Roles of HTML, CSS and JavaScript?**
HTML is structure and content, CSS is presentation and layout, JavaScript is behaviour and state.

**38. What is the DOM?**
The Document Object Model — the browser's tree representation of the page. JavaScript reads and
modifies it (`querySelector`, `classList`, `textContent`) to update the game.

**39. What is an event listener?**
A function registered for an event on an element, such as `click` or `keydown`. The browser calls
it when the event happens; here it turns a click or a keypress into a guess.

**40. What is an API? Did you use one?**
An interface through which programs talk to each other. This project uses no external API; the
only interface it exposes is the small `window.aiHangman` object for debugging and testing.

**41. What is responsive design?**
Designing one page that adapts to different screen sizes using fluid units, flexible layouts
(Grid/Flexbox), media queries and the viewport meta tag.

**42. Why is the hint button disabled after three uses?**
Because the limit is enforced in two places: the button is disabled for the user and `aiHint()`
returns early when `hintsLeft <= 0`. Never rely on the UI alone for rules.

---

## Likely trick questions

**"You only filter a list — where is the intelligence?"**
Filtering is search over a hypothesis space. The engine maintains every explanation consistent
with the evidence and selects the action with the highest expected information. That is the same
loop used by diagnostic expert systems, just with words instead of faults.

**"What is your AI's accuracy?"**
The hint is 100% correct by construction, so accuracy is not the right metric. The useful metric
is how much it narrows the search — for example taking 14 candidates down to 1 with a single
hint. There is no trained model, so there is no test-set accuracy.

**"Why should this be considered AI and not just a game?"**
The game is the interface. The reasoning engine — knowledge base, constraint filtering, heuristic
letter choice and a confidence measure — is the AI component, and it is visible on screen so it
can be inspected and judged.

**"Can the AI lose?"**
The AI does not compete. It assists the player, and only if the player ignores it and uses all
six lives does the game end in a loss.

---

## One-line summary to memorise

"AI Hangman is a single-file web game whose assistant keeps every word consistent with the
player's guesses, recommends the letter that covers the most remaining candidates, and shows its
confidence — a small, explainable example of knowledge representation, constraint-based search
and heuristic decision-making."
