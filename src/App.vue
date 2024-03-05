<template>
  <div id="app">
    <!-- Start Screen -->
    <div v-if="!isGameStarted" class="start-screen">
      <button @click="startGame">Start Game</button>
    </div>

    <!-- Game UI -->
    <div v-if="isGameStarted" @keydown="keyDown" tabindex="0" ref="gameContainer">
      <div v-if="gameOver">
        <h2>Game Over</h2>
        <p>Score: {{ score }}</p>
        <button @click="resetGame">Restart Game</button>
      </div>
      <div v-else>
        <div v-for="row in rows" :key="row" class="row">
          <div v-for="col in cols" :key="col" class="cell" :class="{
      'snake-cell': isSnake(row, col),
      'food-cell': isFood(row, col)
    }"></div>
        </div>
      </div>
    </div>
  </div>
</template>


<script>
export default {
  data() {
    return {
      rows: 20,
      cols: 20,
      snake: [],
      direction: "right",
      foods: [],
      gameOver: false,
      score: 0,
      gameInterval: null,
      isGameStarted: false, // Add this new property
    };
  },
  methods: {
    startGame() {
      this.isGameStarted = true;
      this.resetGame(); // Initialize or reset the game state
      this.$nextTick(() => {
        this.$refs.gameContainer.focus(); // Set focus to the game container
      });
    },

    resetGame() {
      this.snake = [[10, 10]];
      this.direction = "right";
      this.score = 0;
      this.gameOver = false;
      this.foods = [];
      this.generateFood();
      if (this.gameInterval) {
        clearInterval(this.gameInterval);
      }
      this.gameInterval = setInterval(this.gameTick, 200);
    },

    gameTick() {
      this.moveSnake();
      if (!this.foods.length) {
        this.generateFood(); // Ensure food generation if missing
      }
    },
    moveSnake() {
      let newHead = [this.snake[0][0], this.snake[0][1]];
      switch (this.direction) {
        case "up": newHead[0]--; break;
        case "down": newHead[0]++; break;
        case "left": newHead[1]--; break;
        case "right": newHead[1]++; break;
      }
      if (this.isCollision(newHead)) {
        this.gameOver = true;
        //console.log("Game Over. Snake's last position:", newHead);
        clearInterval(this.gameInterval);
        return;
      }
      this.snake.unshift(newHead);
      if (!this.eatFood()) {
        this.snake.pop(); // Remove the tail if no food eaten
      }
    },
    generateFood() {
      let newFoodPosition;
      let foundSafeSpot = false;
      let attempts = 0;
      while (!foundSafeSpot && attempts < 1000) { // Safety limit to prevent infinite loops
        newFoodPosition = [
          Math.floor(Math.random() * (this.rows - 1)) + 1, // Exclude row 0
          Math.floor(Math.random() * (this.cols - 1)) + 1, // Exclude col 0
        ];
        // Check if the new position is not on the snake
        if (!this.isSnake(...newFoodPosition)) {
          foundSafeSpot = true;
          this.foods.push(newFoodPosition); // Add new food position to the array
          //console.log("New food generated at:", newFoodPosition); // Log the food's location
        }
        attempts++;
      }

      if (!foundSafeSpot) {
        console.error("Failed to place food in a safe spot after 1000 attempts.");
      }
    },


    eatFood() {
      let eaten = false;
      this.foods = this.foods.filter(food => {
        if (food[0] === this.snake[0][0] && food[1] === this.snake[0][1]) {
          this.score++;
          eaten = true;
          return false; // Remove the eaten food
        }
        return true;
      });
      if (eaten) this.generateFood(); // Generate a new food item
      return eaten;
    },
    isCollision(head) {
      return (
        head[0] < 0 || // Too far up, hitting the top boundary
        head[0] > 20 || // Adjusted to allow movement to row 20, game over if moving beyond
        head[1] < 0 || // Too far left, hitting the left boundary
        head[1] > 20 || // Adjusted to allow movement to column 20, game over if moving beyond
        this.snake.some((segment, index) =>
          index !== 0 && segment[0] === head[0] && segment[1] === head[1]) // Check for self-collision
      );
    },
    keyDown(e) {
      const keyMap = { ArrowUp: "up", ArrowDown: "down", ArrowLeft: "left", ArrowRight: "right" };
      const newDirection = keyMap[e.key];
      const oppositeDirection = {
        up: "down",
        down: "up",
        left: "right",
        right: "left"
      };

      // Prevent reversing direction if the snake has length > 1
      if (newDirection && this.snake.length > 1 && newDirection !== oppositeDirection[this.direction]) {
        this.direction = newDirection;
      } else if (newDirection && this.snake.length === 1) {
        // Allow any direction if the snake has not eaten any food (length === 1)
        this.direction = newDirection;
      }
    },
    isFood(row, col) {
      return this.foods.some(food => food[0] === row && food[1] === col);
    },
    isSnake(row, col) {
      return this.snake.some(part => part[0] === row && part[1] === col);
    },
  },
  mounted() {
    //this.startGame();
  },
};
</script>

<style>
#app {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  flex-direction: column;
}

.row {
  display: flex;
}

.cell {
  width: 20px;
  height: 20px;
  border: 1px solid #eee;
}

.snake-cell {
  background-color: green;
}

.food-cell {
  background-color: red;
}
</style>
