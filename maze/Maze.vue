<template>
  <div class="content">
    <div class="canvas_wrap" ref="canvas_wrap">
      <canvas id="mainForm" :style="`transform: translate(-50%,-50%) scale(${(widthOuter - 2) / heightInner})`"
        ref="mainForm"></canvas>
    </div>
    <div class="settings_container">
      <div class="settings">
        <div class="switchLang" v-on:click="switchLang()">
          <img src="@/assets/switchLang.svg" alt="" />
        </div>
        <p>{{ (lang === 'en' ? 'Steps remain: ' : '剩余步数：') + steps }}</p>
      </div>
      <div class="controls">
        <div id="up" class="up move"></div>
        <div id="right" class="right move"></div>
        <div id="down" class="down move"></div>
        <div id="left" class="left move"></div>
      </div>
    </div>
    <div class="Q_A" v-show="steps < 1">
      <div class="question">
        <p>{{ lang === 'en' ? Q.Q : Q.Qzn }}</p>
      </div>
      <div class="answer">
        <div class="answer_item">
          <button ref="A" :id="Q.A" v-on:click="CheckAnswer(A, Q)">
            <span class="DOT">A</span>{{ lang === 'en' ? Q.A : Q.Azn }}
          </button>
        </div>
        <div class="answer_item">
          <button ref="B" :id="Q.B" v-on:click="CheckAnswer(B, Q)">
            <span class="DOT">B</span>{{ lang === 'en' ? Q.B : Q.Bzn }}
          </button>
        </div>
        <div class="answer_item">
          <button ref="C" :id="Q.C" v-on:click="CheckAnswer(C, Q)">
            <span class="DOT">C</span>{{ lang === 'en' ? Q.C : Q.Czn }}
          </button>
        </div>
        <div class="answer_item">
          <button ref="D" :id="Q.D" v-on:click="CheckAnswer(D, Q)">
            <span class="DOT">D</span>{{ lang === 'en' ? Q.D : Q.Dzn }}
          </button>
        </div>
      </div>
    </div>
    <div class="finished" v-show="hasFinished">
      <button class="close" v-on:click="() => {
          hasFinished = false
          player.reset()
          maze.generate()
        }
        ">
        <span class="first"></span><span class="second"></span>
      </button>
      <h1>Congratulations</h1>
      <div class="content_wrap row">
        <div class="left col-12 col-md-18 offset-md-2">
          <img src="https://static.igem.wiki/teams/4591/wiki/wiki/maze/skating2.png" alt="" />
        </div>
        <div class="right col-12 col-md-24">
          <p>
            {{
              lang === 'en'
              ? "You've been awarded the Medal of"
              : String.raw`恭喜你被评为:`
            }}<br /><b>{{
  lang === 'en'
  ? "green expert!"
  : String.raw`环保小能手!`
}}</b>
          </p>
        </div>
      </div>
    </div>
    <footer>
      this page is based on
      <a href="https://github.com/JakeMoe/htmlMaze">htmlMaze</a>
    </footer>
  </div>
</template>

<script setup>
import { onMounted } from 'vue'
import { ref } from 'vue'
import { useElementSize } from '@vueuse/core'
const canvas_wrap = ref()
const mainForm = ref()
const { width: widthOuter } = useElementSize(canvas_wrap)
const { height: heightInner } = useElementSize(mainForm)
import { QUESTIONS } from './questions'
import playerLogo from '@/assets/logo.svg'
let bottle = 'https://static.igem.wiki/teams/4591/wiki/wiki/home/plastic-3.svg'
const A = ref()
const B = ref()
const C = ref()
const D = ref()

const lang = ref('en')
const switchLang = () => {
  if (lang.value === 'en') {
    lang.value = 'zh'
  } else {
    lang.value = 'en'
  }
}

const hasFinished = ref(false)

const steps = ref(5)
let ctx
let canvas
let maze
let mazeHeight
let mazeWidth
let player

// shuffle
let Questions = QUESTIONS.sort(() => Math.random() - 0.5)

function createArrayIterator(arr) {
  let index = 0
  return {
    next: function () {
      if (index < arr.length) {
        return { done: false, value: arr[index++] }
      } else {
        index = 0
        return { done: true, value: arr[index++] }
      }
    }
  }
}
const QuestionIterator = createArrayIterator(Questions)

const Q = ref(QuestionIterator.next().value)

const nextQuestion = () => {
  Q.value = QuestionIterator.next().value
}
// nextQuestion()

function CheckAnswer(element, Q) {
  if (
    element.style.backgroundColor !== '' ||
    document.getElementById(Q.correct).style.backgroundColor !== ''
  ) {
    return
  }
  if (element.id == Q.correct) {
    element.style.backgroundColor = 'green'
    setTimeout(() => {
      element.style.backgroundColor = ''
      steps.value = steps.value + 10
      nextQuestion()
    }, 1700)
  } else {
    element.style.backgroundColor = 'red'
    document.getElementById(Q.correct).style.backgroundColor = 'green'
    setTimeout(() => {
      element.style.backgroundColor = ''
      document.getElementById(Q.correct).style.backgroundColor = ''
      nextQuestion()
    }, 2700)
  }
}

function addStep(num) {
  console.log(num)
  steps.value = steps.value + num
}

class Player {
  constructor() {
    this.reset()
  }

  reset() {
    this.col = 0
    this.row = 0
  }
}

class MazeCell {
  constructor(col, row) {
    // console.log(col, row)
    this.col = col
    this.row = row

    this.eastWall = true
    this.northWall = true
    this.southWall = true
    this.westWall = true

    this.visited = false
  }
}

class Maze {
  constructor(cols, rows, cellSize) {
    this.backgroundColor = '#ffffff'
    this.cols = cols
    this.endColor = '#88FF88'
    this.mazeColor = '#000000'
    this.playerColor = '#880088'
    this.rows = rows
    this.cellSize = cellSize

    this.cells = []

    this.generate()
  }

  generate() {
    mazeHeight = this.rows * this.cellSize
    mazeWidth = this.cols * this.cellSize

    canvas.height = mazeHeight
    canvas.width = mazeWidth
    canvas.style.height = mazeHeight
    canvas.style.width = mazeWidth

    for (let col = 0; col < this.cols; col++) {
      this.cells[col] = []
      for (let row = 0; row < this.rows; row++) {
        this.cells[col][row] = new MazeCell(col, row)
      }
    }

    let rndCol = Math.floor(Math.random() * this.cols)
    let rndRow = Math.floor(Math.random() * this.rows)

    let stack = []
    stack.push(this.cells[rndCol][rndRow])

    let currCell
    let dir
    let foundNeighbor
    let nextCell

    while (this.hasUnvisited(this.cells)) {
      currCell = stack[stack.length - 1]
      currCell.visited = true
      if (this.hasUnvisitedNeighbor(currCell)) {
        nextCell = null
        foundNeighbor = false
        do {
          dir = Math.floor(Math.random() * 4)
          switch (dir) {
            case 0:
              if (
                currCell.col !== this.cols - 1 &&
                !this.cells[currCell.col + 1][currCell.row].visited
              ) {
                currCell.eastWall = false
                nextCell = this.cells[currCell.col + 1][currCell.row]
                nextCell.westWall = false
                foundNeighbor = true
              }
              break
            case 1:
              if (currCell.row !== 0 && !this.cells[currCell.col][currCell.row - 1].visited) {
                currCell.northWall = false
                nextCell = this.cells[currCell.col][currCell.row - 1]
                nextCell.southWall = false
                foundNeighbor = true
              }
              break
            case 2:
              if (
                currCell.row !== this.rows - 1 &&
                !this.cells[currCell.col][currCell.row + 1].visited
              ) {
                currCell.southWall = false
                nextCell = this.cells[currCell.col][currCell.row + 1]
                nextCell.northWall = false
                foundNeighbor = true
              }
              break
            case 3:
              if (currCell.col !== 0 && !this.cells[currCell.col - 1][currCell.row].visited) {
                currCell.westWall = false
                nextCell = this.cells[currCell.col - 1][currCell.row]
                nextCell.eastWall = false
                foundNeighbor = true
              }
              break
          }
          if (foundNeighbor) {
            stack.push(nextCell)
          }
        } while (!foundNeighbor)
      } else {
        currCell = stack.pop()
      }
    }

    this.redraw()
  }

  hasUnvisited() {
    for (let col = 0; col < this.cols; col++) {
      for (let row = 0; row < this.rows; row++) {
        if (!this.cells[col][row].visited) {
          return true
        }
      }
    }
    return false
  }

  hasUnvisitedNeighbor(mazeCell) {
    return (
      (mazeCell.col !== 0 && !this.cells[mazeCell.col - 1][mazeCell.row].visited) ||
      (mazeCell.col !== this.cols - 1 && !this.cells[mazeCell.col + 1][mazeCell.row].visited) ||
      (mazeCell.row !== 0 && !this.cells[mazeCell.col][mazeCell.row - 1].visited) ||
      (mazeCell.row !== this.rows - 1 && !this.cells[mazeCell.col][mazeCell.row + 1].visited)
    )
  }

  redraw() {
    ctx.fillStyle = this.backgroundColor
    ctx.fillRect(0, 0, mazeHeight, mazeWidth)

    const imgBottle = new Image()
    imgBottle.src = bottle
    imgBottle.onload = () => {
      ctx.drawImage(
        imgBottle,
        (this.cols - 1) * this.cellSize + 2,
        (this.rows - 1) * this.cellSize + 2,
        this.cellSize - 4,
        this.cellSize - 4
      )
    }

    ctx.strokeRect(0, 0, mazeHeight, mazeWidth)
    ctx.strokeStyle = this.mazeColor

    for (let col = 0; col < this.cols; col++) {
      for (let row = 0; row < this.rows; row++) {
        if (this.cells[col][row].eastWall) {
          ctx.beginPath()
          ctx.moveTo((col + 1) * this.cellSize, row * this.cellSize)
          ctx.lineTo((col + 1) * this.cellSize, (row + 1) * this.cellSize)
          ctx.stroke()
        }
        if (this.cells[col][row].northWall) {
          ctx.beginPath()
          ctx.moveTo(col * this.cellSize, row * this.cellSize)
          ctx.lineTo((col + 1) * this.cellSize, row * this.cellSize)
          ctx.stroke()
        }
        if (this.cells[col][row].southWall) {
          ctx.beginPath()
          ctx.moveTo(col * this.cellSize, (row + 1) * this.cellSize)
          ctx.lineTo((col + 1) * this.cellSize, (row + 1) * this.cellSize)
          ctx.stroke()
        }
        if (this.cells[col][row].westWall) {
          ctx.beginPath()
          ctx.moveTo(col * this.cellSize, row * this.cellSize)
          ctx.lineTo(col * this.cellSize, (row + 1) * this.cellSize)
          ctx.stroke()
        }
      }
    }

    const imgPlayer = new Image()
    imgPlayer.src = playerLogo
    imgPlayer.onload = () => {
      ctx.drawImage(
        imgPlayer,
        player.col * this.cellSize + 2,
        player.row * this.cellSize + 2,
        this.cellSize - 4,
        this.cellSize - 4
      )
    }

    if (player.col == this.cols - 1 && player.row == this.rows - 1) {
      finished()
    }
  }
}

function finished() {
  steps.value = 10
  hasFinished.value = true
}

function onClick(event) {
  player.reset()
  maze.generate()
}

function onControlClick(event) {
  if (steps.value < 1) {
    return
  }
  switch (event.target.id) {
    case 'left':
      if (!maze.cells[player.col][player.row].westWall) {
        steps.value--
        player.col -= 1
        maze.redraw()
      }
      break
    case 'right':
      if (!maze.cells[player.col][player.row].eastWall) {
        steps.value--
        player.col += 1
        maze.redraw()
      }
      break
    case 'down':
      if (!maze.cells[player.col][player.row].southWall) {
        steps.value--
        player.row += 1
        maze.redraw()
      }
      break
    case 'up':
      if (!maze.cells[player.col][player.row].northWall) {
        steps.value--
        player.row -= 1
        maze.redraw()
      }
      break
    default:
      break
  }
}

function onKeyDown(event) {
  switch (event.keyCode) {
    case 37:
    case 65:
      if (steps.value < 1) {
        return
      }
      if (!maze.cells[player.col][player.row].westWall) {
        player.col -= 1
        steps.value--
        maze.redraw()
      }
      break
    case 39:
    case 68:
      if (steps.value < 1) {
        return
      }
      if (!maze.cells[player.col][player.row].eastWall) {
        player.col += 1
        steps.value--
        maze.redraw()
      }
      break
    case 40:
    case 83:
      if (steps.value < 1) {
        return
      }
      if (!maze.cells[player.col][player.row].southWall) {
        player.row += 1
        steps.value--
        maze.redraw()
      }
      break
    case 38:
    case 87:
      if (steps.value < 1) {
        return
      }
      if (!maze.cells[player.col][player.row].northWall) {
        player.row -= 1
        steps.value--
        maze.redraw()
      }
      break
    default:
      break
  }
}

function onLoad() {
  canvas = document.getElementById('mainForm')
  ctx = canvas.getContext('2d')

  player = new Player()
  maze = new Maze(9, 9, 60)

  document.addEventListener('keydown', onKeyDown)
  document.getElementById('up').addEventListener('click', onControlClick)
  document.getElementById('right').addEventListener('click', onControlClick)
  document.getElementById('down').addEventListener('click', onControlClick)
  document.getElementById('left').addEventListener('click', onControlClick)
}

onMounted(() => {
  onLoad()
})
</script>

<style lang="scss">
@import '@/style/col.scss';
@import './style.css';

body,
#app {
  height: 100vh;
  width: 100vw;
  margin: 0;
  padding: 0;
}

.content {
  margin: 0;
  padding: 0;
  display: flex;
  justify-content: space-around;
  width: 100%;
  max-width: 1200px;
  // background-color: #000;
  border: 1px solid #000;
  aspect-ratio: 2;
  position: relative;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  flex-wrap: wrap;
  // transform: translate(-50%, -50%);

  .canvas_wrap {
    height: 100%;
    width: 50%;
    overflow: hidden;

    position: relative;

    #mainForm {
      position: relative;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      border: 3px solid #000;
    }
  }

  .settings_container {
    // background: #8b5252;
    display: flex;
    padding: 10px;
    width: 50%;
    height: 100%;
    flex-wrap: wrap;

    .settings {
      font-size: 2rem;
      position: relative;
      width: 100%;
      height: 50%;
      text-align: center;

      .switchLang {
        cursor: pointer;
        position: absolute;
        right: 0;
        top: 0;
        width: 2rem;
        height: 2rem;

        img {
          width: 100%;
          height: 100%;
        }
      }

      p {
        display: block;
        width: 100%;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);

      }
    }

    .controls {
      width: 100%;
      grid-template-columns: 3fr 3fr 3fr;
      font-size: 3em;
      max-width: 350px;
      position: relative;
      left: 50%;
      transform: translateX(-50%);
      height: auto;
      justify-content: center;
      align-content: center;

      .move {
        width: 1.5em;
        border-radius: 5px;
        height: 0.9em;
        text-align: center;
        line-height: 1em;
        padding-top: 0.1em;
        box-sizing: border-box;
        margin-left: calc(50% - (1.5em / 2));
        background-color: #509d8c;
        color: white;
        cursor: pointer;
        font-size: 1.2em;
        font-weight: bold;
        padding: auto;
      }

      .move::after {
        content: '';
        display: inline-block;
        margin-left: 0;
        vertical-align: 0.33em;
        border-top: 0.3em solid;
        border-right: 0.3em solid transparent;
        border-bottom: 0;
        border-left: 0.3em solid transparent;
      }

      .move.up::after {
        transform: rotate(180deg);
      }

      .move.left::after {
        transform: rotate(90deg) translateY(0.07em);
      }

      .move.right::after {
        transform: rotate(270deg) translateY(0.07em);
      }
    }
  }

  .Q_A {
    font-size: 2rem;
    position: absolute;
    height: 50%;
    width: 50%;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background: rgba(132, 132, 132, 0.8);
    border-radius: 5px;
    box-shadow: 0px 5px 10px #888888;

    .question {
      height: 50%;
      width: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
    }

    .answer {
      height: 50%;
      width: 100%;
      display: flex;
      flex-wrap: wrap;
      justify-content: space-around;
      align-items: center;

      .answer_item {
        padding: 0.6em;
        height: 50%;
        width: 50%;
        display: flex;
        justify-content: center;
        align-items: center;

        button {
          font-size: 1.3rem;
          // height: 1.5em;
          background: #fff;
          border-radius: 0.6em;
          padding: 0.3em;
          cursor: pointer;

          .DOT {
            margin-right: 0.5em;
            font-size: 1.5rem;
            display: inline-block;
            height: 1.2em;
            width: 1.2em;
            border-radius: 50%;
            background: #aaa;
          }
        }
      }
    }
  }

  .finished {
    position: absolute;
    width: 90%;
    // background: #eee;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    border-radius: 5px;
    box-shadow: 0px 5px 10px #888888;
    background: linear-gradient(180deg, rgba(248, 246, 255, 1) 0%, rgba(189, 172, 232, 1) 100%);

    h1 {
      text-align: center;
      margin-top: 1em;
    }

    .close {
      position: absolute;
      right: 0;
      top: 0;
      height: 3rem;
      width: 3rem;
      border: none;
      background-color: transparent;
      cursor: pointer;

      span {
        display: inline-block;
        width: 100%;
        height: 4px;
        border-radius: 4px;
        background: #333;
        transform-origin: center;
      }

      span.first {
        transform: rotate(45deg) translate3d(5.5px, 5.5px, 0);
      }

      span.second {
        transform: rotate(-45deg) translate3d(6px, -6px, 0);
      }
    }

    .content_wrap {
      height: auto;
      width: 100%;

      .left {
        aspect-ratio: 1;
        padding: 1em;
        background-size: cover;
        position: relative;

        img {
          width: 100%;
          aspect-ratio: 1;
          position: absolute;
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%);
        }
      }

      .right {
        padding: 1em;

        p {
          font-size: 3em;
          text-align: center;

          b {
            font-size: 1.5em;
          }
        }
      }
    }
  }

  footer {
    margin: 5px;
  }
}

@media (max-width: 768px) {
  .content {
    margin-top: 5px;
    border: none;
    width: 100%;
    max-width: 100%;
    height: fit-content;
    display: flex;
    flex-wrap: wrap;
    position: static;
    transform: none;

    .canvas_wrap {
      box-sizing: border-box;
      width: 95%;
      max-width: 450px;
      aspect-ratio: 1;
      height: unset;
    }

    .settings_container {
      box-sizing: border-box;
      width: 95%;
      height: unset;

      .settings {
        height: fit-content;

        p {
          position: static;
          transform: none;
          margin-top: 0.3em;
          margin-bottom: 0.5em;
        }
      }
    }

    .Q_A {
      width: 80%;
      height: unset;

      .question {
        padding: 1.5rem;
        box-sizing: border-box;
      }

      .answer {
        .answer_item {
          width: 80%;

          button {
            width: 100%;
          }
        }
      }
    }

    .finished {
      height: unset;

      .content_wrap {
        .left {
          max-width: 500px;

          img {
            max-width: 450px;
          }
        }

        .right {
          p {
            font-size: 2em;
          }
        }
      }
    }
  }
}
</style>
