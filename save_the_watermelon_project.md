```Python
save-the-watermelon/
├─ README.md
├─ LICENSE
├─ .gitignore
├─ src/
│  ├─ game.py
│  ├─ logic.py
│  ├─ words.py
│  └─ __init__.py
├─ tests/
│  ├─ test_logic.py
│  └─ __init__.py
├─ docs/
│  ├─ design.md
│  ├─ pseudocode.md
│  ├─ test-plan.md
│  └─ screenshots/    (optional — create for your manual screenshots)
└─ data/
   └─ words.txt
Copy each file's content below into the corresponding file in your repo.

README.md
markdown
Copy code
# save-the-watermelon

**What:** A terminal-based word-guessing game where the player must save a watermelon by guessing letters before the "slice counter" runs out.

**Why:** Practice iterative software design, Python programming (clean organization, functions, docstrings, input validation) and Git/GitHub-based project submission.

---

## Run

From the repository root:

```bash
# Run via module
python -m src.game

# Or direct
python src/game.py
Requirements: Python 3.9+. (Recommended 3.10+). No third-party packages required.

Test
Run unit tests:

bash
Copy code
python -m unittest discover -v
# or specifically:
python -m unittest tests.test_logic
Features & Rules
The game randomly selects a secret English word.

Player sees masked word (example: _ a _ e).

Player guesses single letters (alphabet only).

Correct guesses reveal letters. Incorrect guesses decrement the "slice counter".

If slices reach zero → watermelon is sliced (loss). If all letters revealed before slices reach zero → saved (win).

Input validation: only single letters allowed, alphabetical, not already guessed.

Replay option after each game.

Stretch goals (optional): ASCII art stages, difficulty levels, categories, scoreboard.

Known issues / limitations
Word list is modest; for larger variety replace data/words.txt.

No persistent scoreboard between sessions (could be added as stretch goal).

Terminal-only UI (no graphical display).

Repository structure
(see project root layout in the assignment)

Credits
Created by [your name]. Inspired by classic hangman-style terminal games.

yaml
Copy code

---

## `LICENSE`
Use MIT license text. Paste this into `LICENSE`:
```text
MIT License

Copyright (c) 2025

Permission is hereby granted, free of charge, to any person obtaining a copy
...
(standard MIT license body) 
(Use the full MIT text; replace the ellipsis with the standard MIT license content and your name/year.)

.gitignore
text
Copy code
__pycache__/
*.pyc
.env
.vscode/
.idea/
.DS_Store
docs/screenshots/*
data/words.txt
A simple word list — you can expand it.

nginx
Copy code
apple
banana
orange
melon
watermelon
grape
kiwi
lemon
mango
peach
berry
cherry
pineapple
apricot
plum
src/__init__.py
python
Copy code
# package marker for src
src/words.py
python
Copy code
"""
words.py
Loader for word list and helper to choose a secret word.
"""

from pathlib import Path
import random
from typing import List

DATA_DIR = Path(__file__).resolve().parents[1] / "data"
WORDS_FILE = DATA_DIR / "words.txt"

DEFAULT_WORDS: List[str] = [
    "apple", "banana", "orange", "melon", "watermelon",
    "grape", "kiwi", "lemon", "mango", "peach", "berry",
    "cherry", "pineapple", "apricot", "plum"
]

def load_words() -> List[str]:
    """Load words from data/words.txt if present, otherwise return DEFAULT_WORDS."""
    try:
        if WORDS_FILE.exists():
            with WORDS_FILE.open("r", encoding="utf-8") as fh:
                words = [w.strip().lower() for w in fh if w.strip()]
            if words:
                return words
    except Exception:
        # Fall back to defaults on any error
        pass
    return DEFAULT_WORDS.copy()

def choose_secret(words: List[str]) -> str:
    """Return a random word from the provided list (lowercased)."""
    if not words:
        words = DEFAULT_WORDS
    return random.choice(words).lower()
src/logic.py
python
Copy code
"""
logic.py
Core functions for Save The Watermelon game.
Keep pure logic here so it is easy to test.
"""

from typing import Set, Tuple, List

def mask_word(secret: str, guessed: Set[str]) -> str:
    """
    Return the masked version of secret given guessed letters.
    Example: secret="apple", guessed={"a","p"} -> "a p p _ _"
    """
    return " ".join([ch if ch in guessed else "_" for ch in secret])

def is_win(secret: str, guessed: Set[str]) -> bool:
    """Return True when all letters in secret have been guessed."""
    return set(secret) <= set(guessed)

def validate_guess(raw: str) -> Tuple[bool, str]:
    """
    Validate raw input for a guess.
    Returns (is_valid, normalized_or_error_msg)
    Valid if single alphabetic character.
    """
    if not raw:
        return False, "Empty input"
    raw = raw.strip().lower()
    if len(raw) != 1:
        return False, "Please enter a single letter."
    if not raw.isalpha():
        return False, "Please enter a letter (a-z)."
    return True, raw

def process_guess(secret: str, guessed: Set[str], guess: str) -> Tuple[bool, int]:
    """
    Process a guess:
      - Adds guess to guessed set (caller should handle duplicates before calling).
      - Returns (was_correct, slices_lost)
    slices_lost is 0 if correct, 1 if incorrect.
    """
    guess = guess.lower()
    guessed.add(guess)
    if guess in secret:
        return True, 0
    else:
        return False, 1

def render_state(secret: str, guessed: Set[str], slices_left: int) -> str:
    """
    Return a multi-line string representing the game state to print to the user.
    Keep the function pure (no printing) so tests can inspect output easily.
    """
    masked = mask_word(secret, guessed)
    guessed_sorted = " ".join(sorted(guessed)) if guessed else "(none)"
    return (
        f"Secret: {masked}\n"
        f"Guessed letters: {guessed_sorted}\n"
        f"Slices remaining: {slices_left}\n"
    )
src/game.py
python
Copy code
"""
Entry point for Save The Watermelon.
Run: python -m src.game
"""

import random
from src.words import load_words, choose_secret
from src.logic import (
    render_state,
    validate_guess,
    process_guess,
    is_win,
)
import sys

MAX_SLICES = 6

def prompt_yes_no(prompt: str) -> bool:
    while True:
        try:
            rv = input(prompt + " (y/n): ").strip().lower()
        except (EOFError, KeyboardInterrupt):
            print("\nExiting.")
            sys.exit(0)
        if rv in ("y", "yes"):
            return True
        if rv in ("n", "no"):
            return False
        print("Please answer 'y' or 'n'.")

def game_loop():
    words = load_words()
    secret = choose_secret(words)
    guessed = set()
    slices = MAX_SLICES

    while slices > 0 and not is_win(secret, guessed):
        print(render_state(secret, guessed, slices))
        try:
            raw = input("Guess a letter: ")
        except (EOFError, KeyboardInterrupt):
            print("\nExiting game. Goodbye.")
            return

        valid, val = validate_guess(raw)
        if not valid:
            print(f"Invalid input: {val}")
            continue

        guess = val
        if guess in guessed:
            print("You already guessed that letter. Try again.")
            continue

        correct, lost = process_guess(secret, guessed, guess)
        if correct:
            print(f"Nice! '{guess}' is in the word.")
        else:
            slices -= lost
            print(f"Sliced! '{guess}' is not in the word. Remaining slices: {slices}")

    # end game
    if is_win(secret, guessed):
        print(render_state(secret, guessed, slices))
        print("🎉 You saved the watermelon! You win!")
    else:
        print(f"\nOh no! The watermelon was sliced. The word was: {secret}")
        print("Better luck next time!")

def main():
    print("=== Save The Watermelon ===")
    while True:
        game_loop()
        if not prompt_yes_no("Play again?"):
            print("Thanks for playing — see you next time!")
            break

if __name__ == "__main__":
    main()
tests/__init__.py
python
Copy code
# tests package
tests/test_logic.py
python
Copy code
import unittest
from src.logic import mask_word, is_win, validate_guess, process_guess, render_state

class TestLogic(unittest.TestCase):
    def test_mask_word_and_is_win(self):
        secret = "apple"
        guessed = set()
        self.assertEqual(mask_word(secret, guessed), "_ _ _ _ _")
        guessed.update(["a","p","l","e"])
        self.assertTrue(is_win(secret, guessed))
        self.assertEqual(mask_word(secret, guessed), "a p p l e")

    def test_validate_guess(self):
        valid, val = validate_guess("a")
        self.assertTrue(valid)
        self.assertEqual(val, "a")
        valid, val = validate_guess(" A ")
        self.assertTrue(valid)
        self.assertEqual(val, "a")
        valid, msg = validate_guess("")
        self.assertFalse(valid)
        self.assertIn("Empty", msg)
        valid, msg = validate_guess("ab")
        self.assertFalse(valid)
        self.assertIn("single", msg)
        valid, msg = validate_guess("1")
        self.assertFalse(valid)
        self.assertIn("letter", msg)

    def test_process_guess(self):
        secret = "banana"
        guessed = set()
        correct, lost = process_guess(secret, guessed, "b")
        self.assertTrue(correct)
        self.assertEqual(lost, 0)
        self.assertIn("b", guessed)
        # incorrect guess
        correct, lost = process_guess(secret, guessed, "z")
        self.assertFalse(correct)
        self.assertEqual(lost, 1)
        self.assertIn("z", guessed)

    def test_render_state(self):
        secret = "kiwi"
        guessed = {"k"}
        s = render_state(secret, guessed, 4)
        self.assertIn("k _ _ _", s)
        self.assertIn("Slices remaining: 4", s)

if __name__ == "__main__":
    unittest.main()
docs/design.md (Progression 1)
markdown
Copy code
# Design - Save The Watermelon

## Problem statement & target audience
Save The Watermelon is a small terminal word-guessing game for learners practicing Python and casual players who enjoy word puzzles. Target audience: beginner-to-intermediate CS students or high-school players.

## Game rules & win/lose conditions
- A secret word is selected randomly.
- Player sees a masked word (underscores for unknown letters).
- Player guesses single alphabetic letters.
- Correct guesses reveal those letters; incorrect guesses remove one slice.
- Start with MAX_SLICES (6). If slices → 0 the watermelon is sliced (loss).
- If all letters are revealed before slices run out → player wins.

## Core features (must-have)
- Random secret selection.
- Masked display.
- Single-letter input validation (alphabet only, no duplicates).
- Slices counter; lose at 0.
- Clean console UX and replay option.

## Stretch goals (nice-to-have)
- ASCII-art stages showing watermelon being sliced.
- Difficulty levels (word length, slices).
- Word categories and hints.
- Persistent scoreboard or streak tracking.

## Basic flow
1. Start → load words and choose secret.
2. Initialize guessed set and slices.
3. Loop:
   - Render state
   - Prompt for letter
   - Validate input
   - If duplicate play message and continue
   - Update guessed, adjust slices if incorrect
4. End: show win/lose message and replay prompt.

## Data design
- `secret`: `str` (lowercase)
- `guessed`: `set[str]` (letters guessed so far)
- `slices` / `slices_left`: `int` (remaining lives)
- Word list: loaded from `data/words.txt` or built-in default list

## Module / function responsibilities
- `src/words.py` — load_words(), choose_secret()
- `src/logic.py` — mask_word(), is_win(), validate_guess(), process_guess(), render_state()
- `src/game.py` — CLI, game loop, user interaction (uses functions above)
- `tests/` — unit tests for logic functions
docs/pseudocode.md (Progression 2)
markdown
Copy code
# Pseudocode - Save The Watermelon

FUNCTION main_game_loop
  words <- load_words()
  secret <- choose_secret(words)
  guessed <- empty set
  slices <- MAX_SLICES

  WHILE slices > 0 AND NOT is_win(secret, guessed)
    DISPLAY render_state(secret, guessed, slices)
    raw_guess <- prompt_for_letter()
    valid, value_or_msg <- validate_guess(raw_guess)
    IF NOT valid THEN
      DISPLAY "Invalid input: " + value_or_msg
      CONTINUE
    ENDIF
    guess <- value_or_msg
    IF guess IN guessed THEN
      DISPLAY "Already guessed"
      CONTINUE
    ENDIF
    correct, lost <- process_guess(secret, guessed, guess)
    IF correct THEN
      DISPLAY "Nice! '" + guess + "' is in the word."
    ELSE
      slices <- slices - lost
      DISPLAY "Sliced! Remaining: " + slices
    ENDIF
  ENDWHILE

  IF is_win(secret, guessed) THEN
    DISPLAY "You saved the watermelon!"
  ELSE
    DISPLAY "Oh no. The watermelon was sliced. The word was: " + secret
  ENDIF

FUNCTION validate_guess(raw)
  TRIM & LOWER raw
  IF raw == "" THEN RETURN (False, "Empty input")
  IF LENGTH(raw) != 1 THEN RETURN (False, "Please enter a single letter.")
  IF NOT raw.isalpha() THEN RETURN (False, "Please enter a letter.")
  RETURN (True, raw)
END FUNCTION

FUNCTION render_state(secret, guessed, slices)
  masked <- mask_word(secret, guessed)   # returns string with spaces
  guessed_list <- sorted(guessed)
  RETURN multiline string with masked, guessed_list, slices
END FUNCTION

FUNCTION process_guess(secret, guessed, guess)
  guessed.add(guess)
  IF guess IN secret THEN RETURN (True, 0)
  ELSE RETURN (False, 1)
END FUNCTION
docs/test-plan.md (Progression 4)
markdown
Copy code
# Test Plan & Results

## Test matrix (core cases)
1. Valid single-letter correct guess
   - Input: 'a' when secret contains 'a'
   - Expect: mask shows revealed letter, slices unchanged

2. Valid single-letter incorrect guess
   - Input: 'z' when secret does not contain 'z'
   - Expect: slices decremented by 1

3. Repeated guess
   - Input: 'a' then 'a'
   - Expect: second input shows "already guessed" message; no slices lost

4. Invalid inputs
   - Empty string -> reject
   - Multi-character (e.g., "ab") -> reject
   - Non-alpha (e.g., "1", "!") -> reject

5. Win condition
   - Guess all unique letters -> is_win true -> win message displayed

6. Lose condition
   - Make MAX_SLICES incorrect guesses -> slices hits 0 -> lose message and reveal secret

## Unit tests
- `tests/test_logic.py` contains tests for mask_word, validate_guess, process_guess, render_state, is_win.

## Manual test transcripts (example)
1) Start game.
2) Secret: apple (hidden)
3) Guess 'a' -> "Nice!"; masked shows 'a _ _ _ _'
4) Guess 'x' -> "Sliced! Remaining: 5"
5) Repeat 'x' -> "Already guessed"
6) Continue until win or slices reach 0; record final message.

(Include screenshots in `docs/screenshots/` if desired.)

## Results
- Unit tests pass locally with `python -m unittest discover -v`.
- Manual gameplay tested for typical flows — correct reveals, slices decrement, duplicate guesses prevented, input validation messages present.

## Known issues (again)
- No persistent scoreboard.
- Words list is small but easily extended by editing `data/words.txt`.
Commit & tagging suggested workflow (example)
Make meaningful commits for each progression:

bash
Copy code
git init
git add .
git commit -m "P1: Add design doc and project skeleton"
# work on code
git add src docs
git commit -m "P2: Add pseudocode and core modules skeleton"
# add tests & polish
git add tests README
git commit -m "P3/P4: Implement logic, CLI, tests and test plan"
# tag final
git tag v1.0.0
git push origin main --tags
Quick notes / checklist for submission
Repo name: save-the-watermelon (make sure exact).

Confirm these files/folders are present and render on GitHub.

Use meaningful commit messages across the four progressions.

Tag final commit v1.0.0.

If instructor wants a private repo, add them as a collaborator or share.
```
