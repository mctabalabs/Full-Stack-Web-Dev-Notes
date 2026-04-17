# Lesson 1.5: Mini-Project – Guess the Number Game

## Learning Objectives
By the end of this project, you will be able to:
- Apply variables, control flow, loops, and user I/O to build a complete, interactive game.
- Generate random values and compare numbers using strict equality and inequalities.
- Build feedback mechanisms ("Too high" / "Too low").
- Track and count user attempts.
- Control the execution cycle of a game using a continuous loop and `break`/`continue` logic.

---

## 1. Project Brief
Build a console-based "Guess the Number" game. 
**How it works:** 
The computer picks a random integer between `1` and `100`. The player keeps entering guesses via prompts until they find the correct number. After each guess, the game tells the player whether their guess is too high, too low, or correct. As a bonus, it tracks how many attempts it took to guess the number.

---

## 2. Step-by-Step Guide

### 2.1. Create the File
In your `js-foundations` project folder, create a new file named `guess.js`. 
*(Alternatively, you can just add this to your existing `main.js` file.)*

```text
js-foundations/
├── index.html       # The structure of your site
├── main.js          # Previous lesson code
└── guess.js         # Your new game logic
```

Include it in your `index.html` by adding another `<script>` tag. Keep it below your other scripts just to be safe.

```html
<!-- Load the game! -->
<script src="guess.js"></script>
```

### 2.2. Generate the Target Number
To make the game fun, we need a random target number. We can generate this using built-in `Math` tools in JavaScript.

```javascript
// This generates a random whole number between 1 and 100
const target = Math.floor(Math.random() * 100) + 1;
```

> **How does this math work?**
> 1. `Math.random()` generates a decimal between `0` (inclusive) and `1` (exclusive), e.g., `0.456`.
> 2. `* 100` scales it up to a number between `0` and `99.999`.
> 3. `Math.floor()` chops off the decimals, leaving a whole integer from `0` to `99`.
> 4. `+ 1` shifts the range up by one, giving us a final range of `1` to `100`.

### 2.3. Initialize the Attempt Counter
We need to keep a running tally of the player's guesses.

```javascript
// We use 'let' because this value will change!
let attempts = 0;
```

### 2.4. Build the Main Game Loop
We want the game to run continuously until the user wins. We'll use a `while (true)` loop, which runs infinitely unless we manually tell it to `break`.

```javascript
while (true) {
  // 1. Prompt the user for input
  const input = prompt('Guess a number between 1 and 100:');
  
  // 2. Validate & parse string input to a Number type
  const guess = Number(input);
  attempts++; // Increment the attempt counter by 1!

  // 3. Check for invalid or empty input
  if (Number.isNaN(guess) || guess < 1 || guess > 100) {
    alert('Please enter a valid number between 1 and 100.');
    continue; // Skips the rest of the loop and prompts the user again immediately
  }

  // 4. Compare the user's guess against the target
  if (guess === target) {
    alert(`Correct! You guessed it in ${attempts} attempts.`);
    break; // Exit the loop because the game is won!
  } else if (guess < target) {
    alert('Too low! Try again.');
  } else {
    // If it's not strictly equal, and it's not too low, it MUST be too high.
    alert('Too high! Try again.');
  }
}
```

---

## 3. Detailed Concept Explanations
Let's review *why* we wrote the code the way we did.

### The Infinite Loop: `while (true)`
We chose `while (true)` because we have absolutely no idea how many guesses it will take the user to figure out the number. The loop runs indefinitely until the `guess === target` condition triggers the `break` statement.

### The `prompt()` & `Number()` Functions
- `prompt('Message')`: This is a built-in browser function that opens a dialog box, asking the user for text. **Important:** It always returns a *String*, even if the user types a number!
- `Number(input)`: This converts the string we got from `prompt()` back into a usable numeric value for comparisons later. 

### Input Validation
We use `Number.isNaN(guess)` to ensure they actually typed a number and not letters (like "apple"). We also enforce boundaries (`guess < 1 || guess > 100`).

Instead of exiting the game if they make a mistake, we log a warning (`alert()`) and use `continue`. This jumps them right back to the top of the loop so they can guess again. 

> **Pro Tip:** You could choose to *not* count invalid guesses against the player by moving `attempts++` *underneath* the input validation `if` statement!

---

## 4. Testing & Edge Cases
As a developer, you should always test how your program handles weird user behavior.
- **Cancel Prompt:** If the user clicks "Cancel" on the dialog, `input` becomes `null`. `Number(null)` becomes `0`, which triggers our validation logic. This is fine for now, but you could add specialized logic to quit the game if `input === null`.
- **Non-integer input:** If a user types `42.5`, it converts cleanly to `42.5`. This is technically valid, but you could force guesses to be whole integers using `Math.floor()` on the `guess`.
- **Repeated Guesses:** The game currently doesn't stop the user from guessing `50` three times in a row. A nice feature would be storing past guesses in an Array `[]` to check against!

---

## 5. Stretch Enhancements
Finished early? Try implementing one of these advanced features!

1. **Limit Attempts:** 
   Change the rules to have a "Game Over" after a maximum of `10` guesses.
2. **Hint System:** 
   After `5` incorrect guesses, pop up an alert revealing a hint (e.g., "Hint: The number is Even/Odd").
3. **HTML/UI Version:** 
   Move away from `prompt()` and `alert()`. Build a basic HTML form with an `<input>` field and a "Guess" button, then display the feedback directly on the webpage using DOM manipulation!
4. **High-Score Tracker:** 
   Store the fewest number of attempts. (Look into JavaScript's `localStorage` for saving high scores even after the page refreshes).

---

## 6. Deliverable & Submission
1. **The Code:** Submit your fully commented `guess.js` file.
2. **Readme:** Include a brief description in your `README.md` explaining how to run the game (e.g., "Open `index.html` in Chrome and begin entering numbers into the prompt!").
3. **Demo (Optional):** Record a quick 1-minute screencast of you playing and winning the game.

Once built and tested, share your code for review!