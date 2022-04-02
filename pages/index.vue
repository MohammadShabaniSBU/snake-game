<template>
  <div>
    <div class="h-screen w-screen flex justify-center items-center">
      <div>
        <div class="my-6">
            <div v-if="status == 'start'" class="w-full">
                <button @click="start" class="btn btn-blue">
                  Start
                </button>
            </div>
            <div v-else-if="status == 'playing'" class="w-full">
              <button @click="stop" class="btn btn-red">
                Stop
              </button>
            </div>
            <div v-else-if="status == 'stopped'" class="w-full flex gap-2">
              <div class="flex-grow">
                <button @click="start" class="btn btn-blue">Resume</button>
              </div>
              <div class="flex-grow">
                <button @click="tryAgain" class="btn btn-red">Try Again</button>
              </div>
            </div>
            <div v-else-if="status == 'lost'">
              <div class="flex-grow">
                <button @click="tryAgain" class="btn btn-red">Try Again</button>
              </div>
            </div>
        </div>
        <div class="my-4 text-center text-2xl font-bold">
          Score: <span>{{ score }}</span>
        </div>
        <div v-for="i in width" :key="i">
          <div class="flex g-3">
              <div v-for="j in width" :key="j" class="w-7 h-7" :class="{'bg-pink-400': (counter) % 2 == 0, 'bg-pink-600': (counter++) % 2 == 1}">
                <div :id="'c' + (100 * i) + j" class="w-full h-full relative">
                  <div>
                    <div></div>
                  </div>
                  <div>
                    <div></div>
                  </div>
                </div>
              </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      counter: 0,
      status: '', // start playing stopped
      width: [],
      table: 17,
      score: 0,
      interval: null,
      intervalTime: 100,
      dir: '',
      snake: {
        squares: [],
      },
      food: {}
    }
  },
  created() {
    for (let i = 0; i < this.table; i++)
      this.width.push(i)

    this.firstStart()
  },

  mounted() {
    document.addEventListener('keydown', this.updateDir);
  },

  methods: {
    move() {
      let next = this.findNextMove();

      let firstCell = this.snake.squares[0];
      let nextCell = {};
      Object.assign(nextCell, firstCell);
      
      this.moveCell(nextCell, next);

      if (nextCell.x != this.food.x || nextCell.y != this.food.y) {

        let len = this.snake.squares.length;
        this.deleteBody(this.snake.squares[len - 1])

        for (let i = this.snake.squares.length - 1; i > 0; i--) {
          this.snake.squares[i].x = this.snake.squares[i - 1].x
          this.snake.squares[i].y = this.snake.squares[i - 1].y
        }

        this.deleteHead(firstCell);

        this.moveCell(firstCell, next);

        this.addHead(firstCell);

      } else {

        this.deleteHead(firstCell)
        this.deleteFood(nextCell)
        this.addHead(nextCell)

        this.snake.squares.unshift(nextCell);
        
        this.score += 1;
        this.counter = 1;
        this.putFood()

      }

      if (this.isLost()) {
        this.stop();
        this.status = 'lost';

      }

    },

    makeId(cell) {
      return 'c' + (100 * cell.x) + cell.y;
    },

    addBody(cell) {
      document.getElementById(this.makeId(cell)).classList.add('bg-yellow-200', 'rounded-full')
    },

    deleteBody(cell) {
      document.getElementById(this.makeId(cell)).classList.remove('bg-yellow-200', 'rounded-full')
    },

    addHead(cell) {
      this.addBody(cell);
      let dir = this.dir == 'r' || this.dir == 'l' ? 'h' : 'v';
      document.getElementById(this.makeId(cell)).classList.add('head', 'head-' + dir);
    },

    deleteHead(cell) {
      document.getElementById(this.makeId(cell)).classList.remove('head', 'head-v', 'head-h')
    },

    deleteFood(cell) {
      document.getElementById(this.makeId(cell)).classList.remove('bg-blue-500')
    },

    moveCell(cell, next) {
      cell.x += next.x + this.table;
      cell.y += next.y + this.table;
      cell.x %= this.table;
      cell.y %= this.table;
    },

    findNextMove() {
      let next;
      switch (this.dir) {
        case "r":
          next = {x: 0, y: 1};
          break;
        case "u":
          next = {x: -1, y: 0};
          break;
        case "d":
          next = {x: 1, y: 0};
          break;
        case "l":
          next = {x: 0, y: -1};
          break;
      }
      return next;
    },
    
    updateDir(e) {

      switch (e.key) {
        case "ArrowRight":
          if (this.dir != 'l')
            this.dir = 'r';
          break;
        case "ArrowLeft":
          if (this.dir != 'r')
            this.dir = 'l';
          break;
        case "ArrowUp":
          if (this.dir != 'd')
            this.dir = 'u';
          break;
        case "ArrowDown":
          if (this.dir != 'u')
            this.dir = 'd';
      }
    },

    putFood() {
      let squares = JSON.parse(JSON.stringify(this.snake.squares))
      squares = squares.map((val) => {
        return val.x * this.table + val.y;
      });

      let foods = [];

      for (let i = 0; i < this.table; i++) {
        for (let j = 0; j < this.table; j++) {
          if (!squares.includes(i * this.table + j)) 
            foods.push(i * this.table + j)
        }
      }

      let rand = Math.floor(Math.random() * foods.length);
      let x = Math.floor(rand / this.table);
      let y = rand % this.table;
      this.food = {
        x: x,
        y: y
      };
      document.getElementById('c' + (100 * x) + y).classList.add('bg-blue-500');
    },

    firstStart() {
      this.snake.squares = [
          {
            x: 2,
            y: 2,
          },
          {
            x: 2,
            y: 1,
          }
        ];

      this.dir = 'r';
      this.status = 'start';
      this.score = 0;
    },  

    start() {
      this.counter = 1;

      let head = true;
      if (this.status == 'start') {
        for (let square of this.snake.squares) {
          this.addBody(square);

          if (head) {
            this.addHead(square)
          }
          head = false;
        }
        this.putFood();
      }

      this.interval = setInterval(this.move, this.intervalTime);
      this.status = "playing";
    },
    
    stop() {
      this.counter = 1;

      clearInterval(this.interval);
      this.status = 'stopped';
    },

    tryAgain() {
      this.counter = 1;
      clearInterval(this.interval);
      this.deleteFood(this.food);

      this.deleteHead(this.snake.squares[0])
      for (let square of this.snake.squares) {
        this.deleteBody(square)
      }

      this.firstStart();
      this.status = 'start';
    },

    isLost() {
      let head = this.snake.squares[0];
      for (let i = 1; i < this.snake.squares.length; i++) {
        if (head.x == this.snake.squares[i].x && head.y == this.snake.squares[i].y) {
          return true;
        }
      }
      return false;
    }
  }
}
</script>

<style>

.btn {
  @apply font-bold text-white w-full py-3 rounded-lg text-xl;
}

.btn-blue {
  @apply bg-blue-500;
}

.btn-red {
  @apply bg-red-500;
}

.head > div {
  @apply absolute rounded-full bg-white w-2.5 h-2.5 flex justify-center items-center;
}

.head > div > div {
  @apply rounded-full bg-black m-auto w-1.5 h-1.5;
}

.head-v > div:nth-child(1) {
  @apply left-1 top-2;
}

.head-v > div:nth-child(2) {
  @apply right-1 top-2;
}

.head-h > div:nth-child(1) {
  @apply top-1 left-2;
}

.head-h > div:nth-child(2) {
  @apply bottom-1 left-2;
}
</style>