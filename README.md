# tictactoe-vibes
Vibe coded Tic Tac Toe Game with a brief write up of the AI Algorithm. The model used and the prompt to generate it.

# Tech Stack
HTML/JS
AI Model used: GLM-5 Turbo

# AI Difficulties Explained
The AI is implemented as three distinct strategies selected by difficulty, all contained in the getAIMove() function. Here's how each level works:

## Easy — Pure Random
```js

if (difficulty === 'easy')
  return available[Math.floor(Math.random() * available.length)];
```
Collects every empty cell into an array, then picks one at random. Zero intelligence — it doesn't even check if it's about to win or lose.

## Medium — Priority Heuristic
This one follows a ranked checklist, stopping at the first match:

```
Win if possible — iterate every empty cell, temporarily place O, check if that's a winning board state. If yes, play there.
Block if needed — same loop but pretending to be X. If placing X there would win, the AI must block that cell.
Take center — board[4] is the most strategically valuable square (part of 4 winning lines). Grab it if open.
Fallback random — if none of the above triggered, pick randomly.
```

```js

// Can AI win right now?
for (const i of available) {
  board[i] = 'O';
  if (checkWinner(board)?.winner === 'O') { board[i] = null; return i; }
  board[i] = null;
}
// Must block player's win?
for (const i of available) {
  board[i] = 'X';
  if (checkWinner(board)?.winner === 'X') { board[i] = null; return i; }
  board[i] = null;
}
// Center?
if (board[4] === null) return 4;
// Random
return available[Math.floor(Math.random() * available.length)];
```
This makes it feel competent but beatable — it never falls for a one-move win against itself, but it doesn't plan ahead beyond the current turn, so it can be lured into forks.

## Hard — Minimax with Alpha-Beta Pruning
This is the full game-tree search. The core idea:

### Scoring Terminal States
```text

AI wins  → +10 (penalized by depth so it prefers faster wins)
Player wins → -10 (penalized by depth so it delays losses)
Draw       → 0
```
The depth parameter ensures the AI prefers winning sooner rather than later, and losing later rather than sooner. A win at depth 0 scores 10, at depth 1 scores 9, etc.

### The Recursive Function
```js

function minimax(b, depth, isMax, alpha, beta) {
  // 1. Terminal check — is the game over?
  const result = checkWinner(b);
  if (result) {
    if (result.winner === 'O') return 10 - depth;
    if (result.winner === 'X') return depth - 10;
    return 0; // draw
  }

  const available = b.map((v,i) => v === null ? i : null).filter(v => v !== null);

  // 2. Maximizing turn (AI = 'O') — pick the highest score
  if (isMax) {
    let best = -Infinity;
    for (const i of available) {
      b[i] = 'O';                              // try a move
      best = Math.max(best, minimax(b, depth+1, false, alpha, beta));
      b[i] = null;                              // undo it (backtrack)
      alpha = Math.max(alpha, best);
      if (beta <= alpha) break;                 // prune this branch
    }
    return best;
  }

  // 3. Minimizing turn (Player = 'X') — pick the lowest score
  else {
    let best = Infinity;
    for (const i of available) {
      b[i] = 'X';
      best = Math.min(best, minimax(b, depth+1, true, alpha, beta));
      b[i] = null;
      beta = Math.min(beta, best);
      if (beta <= alpha) break;                 // prune this branch
    }
    return best;
  }
}
```

### Alpha-Beta Pruning Explained
Without pruning, minimax explores every possible game sequence. For Tic Tac Toe there are at most 9! = 362,880 leaf nodes — manageable, but pruning cuts this dramatically.

```
alpha — the best score the maximizing player (AI) can guarantee so far on this path
beta — the best score the minimizing player (human) can guarantee so far
```

When beta <= alpha, it means the minimizing player would never allow this branch to be reached (a better option already exists for them), so we skip the rest of the sibling nodes. This typically cuts the search tree by roughly half.

### The Top-Level Call
```js

let bestScore = -Infinity;
let bestMove = available[0];
for (const i of available) {
  board[i] = 'O';
  const score = minimax(board, 0, false, -Infinity, Infinity);
  board[i] = null;
  if (score > bestScore) {
    bestScore = score;
    bestMove = i;
  }
}
return bestMove;
```
The AI tries every empty cell, runs minimax on each resulting board (with the opponent moving next, hence isMax = false), and picks the move that yields the highest score.

## Why It's Unbeatable
Tic Tac Toe is a solved game — with perfect play from both sides, every game ends in a draw. Since Hard mode explores the complete game tree, it plays perfectly. The best a human can achieve against it is a draw, which requires the human to also play perfectly (or for the AI to not get the first move).
