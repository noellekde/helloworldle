const FUNCTIONS = [
  { name: "append", language: "Go", category: "list", description: "Adds one value to the end of a collection.", length: 6 },
  { name: "async", language: "JavaScript", category: "control", description: "Runs work in the background without blocking the current flow.", length: 5 },
  { name: "await", language: "JavaScript", category: "control", description: "Pauses until an async task is finished.", length: 5 },
  { name: "ceil", language: "C#", category: "math", description: "Rounds a number upward to the next integer.", length: 4 },
  { name: "clone", language: "JavaScript", category: "object", description: "Duplicates data so the original stays unchanged.", length: 5 },
  { name: "count", language: "Python", category: "collection", description: "Counts how many values match a condition.", length: 5 },
  { name: "decode", language: "Python", category: "serialization", description: "Rebuilds data from a saved or encoded format.", length: 6 },
  { name: "encode", language: "Python", category: "serialization", description: "Converts data into a portable encoded format.", length: 6 },
  { name: "filter", language: "JavaScript", category: "collection", description: "Keeps only values that satisfy a rule.", length: 6 },
  { name: "floor", language: "C#", category: "math", description: "Rounds a value downward to the nearest integer.", length: 5 },
  { name: "format", language: "Python", category: "string", description: "Builds text using placeholders and values.", length: 6 },
  { name: "index", language: "Python", category: "lookup", description: "Finds the position of a value inside a sequence.", length: 5 },
  { name: "input", language: "Python", category: "user", description: "Reads data typed in by the user.", length: 5 },
  { name: "join", language: "JavaScript", category: "string", description: "Combines list items into one text string.", length: 4 },
  { name: "len", language: "Python", category: "collection", description: "Returns the number of items in a sequence.", length: 3 },
  { name: "map", language: "Python", category: "transform", description: "Applies a function to each item in a collection.", length: 3 },
  { name: "match", language: "JavaScript", category: "pattern", description: "Checks whether a string follows a certain pattern.", length: 5 },
  { name: "merge", language: "Go", category: "object", description: "Combines two collections into one unified result.", length: 5 },
  { name: "parse", language: "C#", category: "conversion", description: "Turns text into a structured value.", length: 5 },
  { name: "pop", language: "JavaScript", category: "array", description: "Removes the last item from a collection.", length: 3 },
  { name: "print", language: "Python", category: "output", description: "Shows text or values in the console.", length: 5 },
  { name: "push", language: "JavaScript", category: "array", description: "Adds a new item to the end of an array.", length: 4 },
  { name: "range", language: "Python", category: "iteration", description: "Creates a sequence of numbers in a span.", length: 5 },
  { name: "reverse", language: "Java", category: "sequence", description: "Flips the order of items in a list.", length: 7 },
  { name: "round", language: "Python", category: "math", description: "Adjusts a number to the nearest integer.", length: 5 },
  { name: "scan", language: "Go", category: "input", description: "Reads values from a stream or standard input.", length: 4 },
  { name: "search", language: "Go", category: "lookup", description: "Locates a value or pattern within data.", length: 6 },
  { name: "slice", language: "Python", category: "sequence", description: "Returns a portion of a string or list.", length: 5 },
  { name: "sort", language: "Java", category: "sequence", description: "Organizes values in ascending or descending order.", length: 4 },
  { name: "split", language: "Python", category: "string", description: "Breaks text into smaller pieces using a delimiter.", length: 5 },
  { name: "trim", language: "JavaScript", category: "string", description: "Removes blank space from the ends of text.", length: 4 }
];

const MAX_GUESSES = 6;
const HINT_ORDER = ["language", "category", "description", "length"];

let secret = null;
let guesses = [];
let revealedHints = 0;

const guessBoard = document.getElementById("guessBoard");
const guessForm = document.getElementById("guessForm");
const guessInput = document.getElementById("guessInput");
const messageEl = document.getElementById("message");
const attemptsLabel = document.getElementById("attemptsLabel");
const dailyStamp = document.getElementById("dailyStamp");
const functionList = document.getElementById("functionList");
const newGameButton = document.getElementById("newGameButton");

function hashString(str) {
  let hash = 0;
  for (let i = 0; i < str.length; i += 1) {
    hash = (hash << 5) - hash + str.charCodeAt(i);
    hash |= 0;
  }
  return Math.abs(hash);
}

function getTodayString() {
  return new Date().toISOString().slice(0, 10);
}

function pickSecret() {
  const today = getTodayString();
  const index = hashString(today) % FUNCTIONS.length;
  return FUNCTIONS[index];
}

function populateDatalist() {
  const options = FUNCTIONS.map((entry) => `<option value="${entry.name}">${entry.name}</option>`).join("");
  functionList.innerHTML = options;
}

function revealHint(key) {
  const hintCard = document.querySelector(`[data-hint="${key}"]`);
  if (!hintCard) return;

  const valueEl = hintCard.querySelector(".hint-value");
  const rawValue = secret[key];

  hintCard.classList.add("revealed");
  valueEl.textContent = key === "length" ? `${rawValue} letters` : rawValue;
}

function revealNextHint() {
  if (revealedHints >= HINT_ORDER.length) return;
  const key = HINT_ORDER[revealedHints];
  revealHint(key);
  revealedHints += 1;
}

function setStatus(text, type = "") {
  messageEl.textContent = text;
  messageEl.className = `message ${type}`.trim();
}

function evaluateGuess(secretWord, guessWord) {
  const result = Array(secretWord.length).fill("absent");
  const lettersLeft = {};

  for (let i = 0; i < secretWord.length; i += 1) {
    if (secretWord[i] === guessWord[i]) {
      result[i] = "correct";
    } else {
      lettersLeft[secretWord[i]] = (lettersLeft[secretWord[i]] || 0) + 1;
    }
  }

  for (let i = 0; i < guessWord.length; i += 1) {
    if (result[i] === "correct") continue;
    const letter = guessWord[i];
    if (lettersLeft[letter] > 0) {
      result[i] = "present";
      lettersLeft[letter] -= 1;
    }
  }

  return result;
}

function renderBoard() {
  guessBoard.innerHTML = "";

  for (const entry of guesses) {
    const row = document.createElement("div");
    row.className = "guess-row";

    for (let i = 0; i < entry.guess.length; i += 1) {
      const tile = document.createElement("div");
      tile.className = `tile ${entry.result[i]}`;
      tile.textContent = entry.guess[i];
      row.appendChild(tile);
    }

    guessBoard.appendChild(row);
  }

  attemptsLabel.textContent = `${guesses.length} / ${MAX_GUESSES} tries`;
}

function startNewGame() {
  secret = pickSecret();
  guesses = [];
  revealedHints = 0;
  guessInput.value = "";
  guessInput.focus();
  guessBoard.innerHTML = "";
  document.querySelectorAll(".hint-card").forEach((card) => {
    card.classList.remove("revealed");
    const valueEl = card.querySelector(".hint-value");
    valueEl.textContent = "Locked";
  });
  attemptsLabel.textContent = `0 / ${MAX_GUESSES} tries`;
  dailyStamp.textContent = `Daily: ${getTodayString()}`;
  setStatus("Make your first guess. Each wrong try reveals a new hint.");
}

function handleGuess(event) {
  event.preventDefault();

  if (!secret) return;

  const input = guessInput.value.trim().toLowerCase();
  if (!input) {
    setStatus("Enter a function name first.", "error");
    return;
  }

  const match = FUNCTIONS.find((entry) => entry.name.toLowerCase() === input);
  if (!match) {
    setStatus("That function isn’t in the challenge set. Try another one.", "error");
    return;
  }

  if (guesses.some((entry) => entry.guess === match.name)) {
    setStatus("You already guessed that one.", "error");
    return;
  }

  const result = evaluateGuess(secret.name, match.name);
  guesses.push({ guess: match.name, result });
  renderBoard();

  if (match.name === secret.name) {
    revealHint("language");
    revealHint("category");
    revealHint("description");
    revealHint("length");
    revealedHints = HINT_ORDER.length;
    setStatus(`You win! ${secret.name} was the secret function.`, "success");
    guessInput.value = "";
    return;
  }

  if (guesses.length >= MAX_GUESSES) {
    revealHint("language");
    revealHint("category");
    revealHint("description");
    revealHint("length");
    revealedHints = HINT_ORDER.length;
    setStatus(`Game over. The answer was ${secret.name}.`, "error");
    guessInput.value = "";
    return;
  }

  revealNextHint();
  setStatus(`Not quite. A new hint has been unlocked.`, "error");
  guessInput.value = "";
}

populateDatalist();
startNewGame();
guessForm.addEventListener("submit", handleGuess);
newGameButton.addEventListener("click", startNewGame);
