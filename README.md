# jeryl007.github.io
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Chess Game vs AI</title>

  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: Arial, sans-serif;
    }

    body {
      background: #1e293b;
      color: white;
      min-height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      padding: 20px;
    }

    h1 {
      margin-bottom: 20px;
    }

    .game-container {
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    .board {
      display: grid;
      grid-template-columns: repeat(8, 80px);
      grid-template-rows: repeat(8, 80px);
      border: 5px solid #0f172a;
    }

    .cell {
      width: 80px;
      height: 80px;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 42px;
      cursor: pointer;
      user-select: none;
    }

    .white {
      background: #f0d9b5;
    }

    .black {
      background: #b58863;
    }

    .selected {
      outline: 4px solid yellow;
    }

    .status {
      margin-top: 20px;
      font-size: 1.2rem;
    }

    .controls {
      margin-top: 20px;
      display: flex;
      gap: 10px;
    }

    button {
      padding: 10px 18px;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      background: #22c55e;
      color: white;
      font-size: 1rem;
    }

    button:hover {
      background: #16a34a;
    }

    @media (max-width: 700px) {
      .board {
        grid-template-columns: repeat(8, 45px);
        grid-template-rows: repeat(8, 45px);
      }

      .cell {
        width: 45px;
        height: 45px;
        font-size: 24px;
      }
    }
  </style>
</head>
<body>

<h1>Chess vs AI</h1>

<div class="game-container">
  <div class="board" id="board"></div>

  <div class="status" id="status">
    Your Turn (White)
  </div>

  <div class="controls">
    <button onclick="resetGame()">Restart Game</button>
  </div>
</div>

<script>
  const boardElement = document.getElementById("board");
  const statusElement = document.getElementById("status");

  const pieces = {
    r: "♜",
    n: "♞",
    b: "♝",
    q: "♛",
    k: "♚",
    p: "♟",
    R: "♖",
    N: "♘",
    B: "♗",
    Q: "♕",
    K: "♔",
    P: "♙"
  };

  let board = [];
  let selected = null;
  let currentPlayer = "white";
  let gameOver = false;

  function initBoard() {
    board = [
      ["r","n","b","q","k","b","n","r"],
      ["p","p","p","p","p","p","p","p"],
      ["","","","","","","",""],
      ["","","","","","","",""],
      ["","","","","","","",""],
      ["","","","","","","",""],
      ["P","P","P","P","P","P","P","P"],
      ["R","N","B","Q","K","B","N","R"]
    ];

    selected = null;
    currentPlayer = "white";
    gameOver = false;

    renderBoard();
    statusElement.textContent = "Your Turn (White)";
  }

  function renderBoard() {
    boardElement.innerHTML = "";

    for (let row = 0; row < 8; row++) {
      for (let col = 0; col < 8; col++) {
        const cell = document.createElement("div");

        cell.classList.add("cell");
        cell.classList.add((row + col) % 2 === 0 ? "white" : "black");

        if (
          selected &&
          selected.row === row &&
          selected.col === col
        ) {
          cell.classList.add("selected");
        }

        const piece = board[row][col];
        cell.textContent = pieces[piece] || "";

        cell.addEventListener("click", () => handleCellClick(row, col));

        boardElement.appendChild(cell);
      }
    }
  }

  function handleCellClick(row, col) {
    if (gameOver || currentPlayer !== "white") return;

    const piece = board[row][col];

    if (selected) {
      movePiece(selected.row, selected.col, row, col);

      selected = null;
      renderBoard();
    } else {
      if (piece && isWhite(piece)) {
        selected = { row, col };
        renderBoard();
      }
    }
  }

  function movePiece(fromRow, fromCol, toRow, toCol) {
    const piece = board[fromRow][fromCol];
    const target = board[toRow][toCol];

    if (!piece) return;

    // Prevent taking own piece
    if (target) {
      if (isWhite(piece) === isWhite(target)) {
        return;
      }
    }

    // Basic movement validation
    if (!isValidMove(piece, fromRow, fromCol, toRow, toCol)) {
      return;
    }

    // End game if king captured
    if (target === "k") {
      board[toRow][toCol] = piece;
      board[fromRow][fromCol] = "";
      renderBoard();
      statusElement.textContent = "🎉 You Win!";
      gameOver = true;
      return;
    }

    if (target === "K") {
      board[toRow][toCol] = piece;
      board[fromRow][fromCol] = "";
      renderBoard();
      statusElement.textContent = "💀 AI Wins!";
      gameOver = true;
      return;
    }

    board[toRow][toCol] = piece;
    board[fromRow][fromCol] = "";

    currentPlayer = currentPlayer === "white" ? "black" : "white";

    statusElement.textContent =
      currentPlayer === "white"
        ? "Your Turn (White)"
        : "AI Thinking...";

    renderBoard();

    if (currentPlayer === "black") {
      setTimeout(aiMove, 500);
    }
  }

  function aiMove() {
    if (gameOver) return;

    const moves = [];

    for (let r = 0; r < 8; r++) {
      for (let c = 0; c < 8; c++) {
        const piece = board[r][c];

        if (piece && isBlack(piece)) {
          for (let tr = 0; tr < 8; tr++) {
            for (let tc = 0; tc < 8; tc++) {
              if (isValidMove(piece, r, c, tr, tc)) {
                const target = board[tr][tc];

                if (!target || isWhite(target)) {
                  moves.push({
                    fromRow: r,
                    fromCol: c,
                    toRow: tr,
                    toCol: tc
                  });
                }
              }
            }
          }
        }
      }
    }

    if (moves.length === 0) {
      statusElement.textContent = "Draw!";
      gameOver = true;
      return;
    }

    const move = moves[Math.floor(Math.random() * moves.length)];

    movePiece(
      move.fromRow,
      move.fromCol,
      move.toRow,
      move.toCol
    );
  }

  function isWhite(piece) {
    return piece === piece.toUpperCase();
  }

  function isBlack(piece) {
    return piece === piece.toLowerCase();
  }

  function isValidMove(piece, fr, fc, tr, tc) {
    if (fr === tr && fc === tc) return false;

    const dx = tc - fc;
    const dy = tr - fr;

    switch (piece.toLowerCase()) {
      case "p":
        if (piece === "P") {
          if (dx === 0 && dy === -1 && !board[tr][tc]) return true;
          if (Math.abs(dx) === 1 && dy === -1 && board[tr][tc]) return true;
        } else {
          if (dx === 0 && dy === 1 && !board[tr][tc]) return true;
          if (Math.abs(dx) === 1 && dy === 1 && board[tr][tc]) return true;
        }
        break;

      case "r":
        if (dx === 0 || dy === 0) return clearPath(fr, fc, tr, tc);
        break;

      case "b":
        if (Math.abs(dx) === Math.abs(dy))
          return clearPath(fr, fc, tr, tc);
        break;

      case "q":
        if (
          dx === 0 ||
          dy === 0 ||
          Math.abs(dx) === Math.abs(dy)
        ) {
          return clearPath(fr, fc, tr, tc);
        }
        break;

      case "n":
        if (
          (Math.abs(dx) === 2 && Math.abs(dy) === 1) ||
          (Math.abs(dx) === 1 && Math.abs(dy) === 2)
        ) {
          return true;
        }
        break;

      case "k":
        if (Math.abs(dx) <= 1 && Math.abs(dy) <= 1)
          return true;
        break;
    }

    return false;
  }

  function clearPath(fr, fc, tr, tc) {
    const stepRow = Math.sign(tr - fr);
    const stepCol = Math.sign(tc - fc);

    let r = fr + stepRow;
    let c = fc + stepCol;

    while (r !== tr || c !== tc) {
      if (board[r][c] !== "") return false;

      r += stepRow;
      c += stepCol;
    }

    return true;
  }

  function resetGame() {
    initBoard();
  }

  initBoard();
</script>

</body>
</html>
