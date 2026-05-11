# jeryl007.github.io
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Chess Game</title>

  <style>
    *{
      margin:0;
      padding:0;
      box-sizing:border-box;
      font-family:Arial, sans-serif;
    }

    body{
      background:#1e293b;
      display:flex;
      justify-content:center;
      align-items:center;
      min-height:100vh;
      color:white;
      flex-direction:column;
    }

    h1{
      margin-bottom:20px;
      font-size:3rem;
    }

    .game-container{
      display:flex;
      flex-direction:column;
      align-items:center;
      gap:20px;
    }

    .status{
      font-size:1.2rem;
      font-weight:bold;
    }

    .board{
      width:640px;
      height:640px;
      display:grid;
      grid-template-columns:repeat(8,1fr);
      border:6px solid #111827;
    }

    .square{
      display:flex;
      justify-content:center;
      align-items:center;
      font-size:3rem;
      cursor:pointer;
      user-select:none;
    }

    .light{
      background:#f0d9b5;
    }

    .dark{
      background:#b58863;
    }

    .selected{
      outline:5px solid yellow;
    }

    .possible{
      box-shadow: inset 0 0 0 5px rgba(0,255,0,0.5);
    }

    button{
      padding:12px 24px;
      border:none;
      background:#22c55e;
      color:white;
      border-radius:8px;
      cursor:pointer;
      font-size:1rem;
      font-weight:bold;
      transition:0.2s;
    }

    button:hover{
      background:#16a34a;
    }

    @media(max-width:700px){
      .board{
        width:95vw;
        height:95vw;
      }

      .square{
        font-size:2rem;
      }
    }
  </style>
</head>
<body>

  <h1>♟ Chess Game</h1>

  <div class="game-container">
    <div class="status" id="status">
      White's Turn
    </div>

    <div class="board" id="board"></div>

    <button onclick="resetGame()">Restart Game</button>
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
    let selectedSquare = null;
    let currentPlayer = "white";

    function initialBoard() {
      return [
        ["r","n","b","q","k","b","n","r"],
        ["p","p","p","p","p","p","p","p"],
        ["","","","","","","",""],
        ["","","","","","","",""],
        ["","","","","","","",""],
        ["","","","","","","",""],
        ["P","P","P","P","P","P","P","P"],
        ["R","N","B","Q","K","B","N","R"]
      ];
    }

    function renderBoard() {
      boardElement.innerHTML = "";

      for(let row=0; row<8; row++){
        for(let col=0; col<8; col++){

          const square = document.createElement("div");

          square.classList.add("square");

          if((row + col) % 2 === 0){
            square.classList.add("light");
          } else {
            square.classList.add("dark");
          }

          square.dataset.row = row;
          square.dataset.col = col;

          const piece = board[row][col];

          if(piece){
            square.textContent = pieces[piece];
          }

          square.addEventListener("click", () => handleSquareClick(row, col));

          boardElement.appendChild(square);
        }
      }
    }

    function isWhite(piece){
      return piece === piece.toUpperCase();
    }

    function handleSquareClick(row, col){

      const piece = board[row][col];

      if(selectedSquare){

        const [selectedRow, selectedCol] = selectedSquare;
        const selectedPiece = board[selectedRow][selectedCol];

        if(isValidMove(selectedRow, selectedCol, row, col)){

          board[row][col] = selectedPiece;
          board[selectedRow][selectedCol] = "";

          currentPlayer = currentPlayer === "white" ? "black" : "white";

          statusElement.textContent =
            currentPlayer.charAt(0).toUpperCase() +
            currentPlayer.slice(1) +
            "'s Turn";
        }

        selectedSquare = null;
        renderBoard();
        return;
      }

      if(piece){

        const pieceColor = isWhite(piece) ? "white" : "black";

        if(pieceColor === currentPlayer){
          selectedSquare = [row, col];
          renderBoard();

          highlightSquare(row, col);
        }
      }
    }

    function highlightSquare(row, col){
      const squares = document.querySelectorAll(".square");

      squares.forEach(square => {
        if(
          parseInt(square.dataset.row) === row &&
          parseInt(square.dataset.col) === col
        ){
          square.classList.add("selected");
        }
      });
    }

    function isValidMove(fromRow, fromCol, toRow, toCol){

      const piece = board[fromRow][fromCol];
      const target = board[toRow][toCol];

      if(target){
        if(isWhite(piece) === isWhite(target)){
          return false;
        }
      }

      const rowDiff = toRow - fromRow;
      const colDiff = toCol - fromCol;

      switch(piece.toLowerCase()){

        case "p":
          return validatePawnMove(piece, fromRow, fromCol, toRow, toCol);

        case "r":
          return rowDiff === 0 || colDiff === 0;

        case "n":
          return (
            (Math.abs(rowDiff) === 2 && Math.abs(colDiff) === 1) ||
            (Math.abs(rowDiff) === 1 && Math.abs(colDiff) === 2)
          );

        case "b":
          return Math.abs(rowDiff) === Math.abs(colDiff);

        case "q":
          return (
            rowDiff === 0 ||
            colDiff === 0 ||
            Math.abs(rowDiff) === Math.abs(colDiff)
          );

        case "k":
          return (
            Math.abs(rowDiff) <= 1 &&
            Math.abs(colDiff) <= 1
          );

        default:
          return false;
      }
    }

    function validatePawnMove(piece, fromRow, fromCol, toRow, toCol){

      const direction = isWhite(piece) ? -1 : 1;

      const startRow = isWhite(piece) ? 6 : 1;

      const target = board[toRow][toCol];

      // Forward move
      if(
        fromCol === toCol &&
        !target
      ){

        if(toRow === fromRow + direction){
          return true;
        }

        if(
          fromRow === startRow &&
          toRow === fromRow + direction * 2 &&
          !board[fromRow + direction][fromCol]
        ){
          return true;
        }
      }

      // Capture
      if(
        Math.abs(toCol - fromCol) === 1 &&
        toRow === fromRow + direction &&
        target
      ){
        return true;
      }

      return false;
    }

    function resetGame(){
      board = initialBoard();
      selectedSquare = null;
      currentPlayer = "white";
      statusElement.textContent = "White's Turn";
      renderBoard();
    }

    resetGame();
  </script>

</body>
</html>
