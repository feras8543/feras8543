const canvas = document.getElementById('tetris');
const ctx = canvas.getContext('2d');

const blockSize = 30;
const width = canvas.width / blockSize;
const height = canvas.height / blockSize;

const colors = [null, 'red', 'green', 'yellow', 'blue', 'purple', 'cyan'];

const shapes = [
  // T shape
  [
    [1, 1, 1],
    [0, 1, 0]
  ],
  // I shape
  [
    [1, 1, 1, 1]
  ],
  // O shape
  [
    [1, 1],
    [1, 1]
  ],
  // L shape
  [
    [1, 1, 1],
    [1, 0, 0]
  ],
  // J shape
  [
    [1, 1, 1],
    [0, 0, 1]
  ],
  // S shape
  [
    [0, 1, 1],
    [1, 1, 0]
  ],
  // Z shape
  [
    [1, 1, 0],
    [0, 1, 1]
  ]
];

let score = 0;
let level = 0;
let paused = true;

let board;
let shape;
let shapeX;
let shapeY;
let dx;
let dy;
let scoreInterval;

function init() {
  board = createBoard();
  shape = createShape();
  shapeX = width / 2 | 0;
  shapeY = 0;
  dx = 0;
  dy = -1;
  scoreInterval = 1000;
  score = 0;
  level = 0;
  paused = true;
  drawBoard();
  drawShape();
}

function createBoard() {
  const board = [];
  for (let i = 0; i < height; i++) {
    board[i] = [];
    for (let j = 0; j < width; j++) {
      board[i][j] = EMPTY;
    }
  }
  return board;
}

function createShape() {
  const shape = shapes[Math.floor(Math.random() * shapes.length)];
  const x = Math.floor(Math.random() * (width - shape[0].length));
  const y = Math.floor(Math.random() * (height - shape.length));
  return { shape, x, y };
}

function drawBoard() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  for (let i = 0; i < height; i++) {
    for (let j = 0; j < width; j++) {
      if (board[i][j] !== EMPTY) {
        ctx.fillStyle = colors[board[i][j]];
        ctx.fillRect(j * blockSize, i * blockSize, blockSize, blockSize);
      }
    }
  }
}

function drawShape() {
  ctx.fillStyle = colors[1];
  for (let i = 0; i < shape.shape.length; i++) {
    for (let j = 0; j < shape.shape[i].length; j++) {
      if (shape.shape[i][j] !== 0) {
        ctx.fillRect((shape.x + j) * blockSize, (shape.y + i) * blockSize, blockSize, blockSize);
      }
    }
  }
}

function moveDown() {
  if (canMove(0, 1)) {
    shapeY += dy;
    drawBoard();
    drawShape();
  } else {
    freezeShape();
    shape = createShape();
    if (!canMove(shape)) {
      gameOver();
    }
  }
}

function canMove(dx, dy) {
  const newX = shapeX + dx;
  const newY = shapeY + dy;
  for (let i = 0; i < shape.shape.length; i++) {
    for (let j = 0; j < shape.shape[i].length; j++) {
      if (shape.shape[i][j] !== 0) {
        if (newX + j < 0 || newX + j >= width || newY + i >= height || board[newY + i][newX + j] !== EMPTY) {
          return false;
        }
      }
    }
  }
  return true;
}

function freezeShape() {
  for (let i = 0; i < shape.shape.length; i++) {
    for (let j = 0; j < shape.shape[i].length; j++) {
      if (shape.shape[i][j] !== 0) {
        board[shape.y + i][shape.x + j] = shape.shape[i][j];
      }
    }
  }
  checkLines();
}

function checkLines() {
  let lines = 0;
  for (let i = height - 1; i >= 0; i--) {
    let isLine = true;
    for (let j = 0; j < width; j++) {
      if (board[i][j] === EMPTY) {
        isLine = false;
        break;
      }
    }
    if (isLine) {
      lines++;
      for (let k = i; k > 0; k--) {
        for (let j
