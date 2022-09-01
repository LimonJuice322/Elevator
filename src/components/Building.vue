<script setup>
import { ref, reactive, watch } from 'vue';
import Elevator from '@/components/Elevator.vue';

const queue = reactive([]);
const currentLevel = ref(0);
const offsetTop = ref(0);
const doorWidth = ref(0);
const state = ref('idle');

const toggleLevel = (i) => {
  if (!queue?.includes(i) && i !== currentLevel.value) {
    queue?.push(i);
  }

  if (state.value === "idle") {
    handleLevel(i);
  }
};

const handleLevel = (i) => {
  let nextLevel = i * 80;
  let offset = Math.abs(nextLevel - offsetTop.value);

  if (nextLevel > offsetTop.value) {
    moveElevator(elevatorDown, offset);
  }

  if (nextLevel < offsetTop.value) {
    moveElevator(elevatorUp, offset);
  }

  if (nextLevel === offsetTop.value) {
    moveElevator(() => {}, 0);
  }
};

const moveElevator = (callback, offset) => {
  if (offset === 0) {
    state.value = "waiting";
    return;
  }

  for (let i = 0; i < offset; i++) {
    setTimeout(() => {
      if (i === 0) {
        currentLevel.value = queue.shift();
        state.value = "moving";
      }

      if (i === offset - 1) {
        state.value = "waiting";
      }

      callback();
    }, 12.5 * i);
  }
};

const waitingLevel = () => {
  for (let i = 0; i < 75; i++) {
    if (i < 25) {
      setTimeout(openDoors, 40 * i);
    }

    if (i >= 50) {
      setTimeout(closeDoors, 40 * i);
    }
  }

  setTimeout(() => {
    state.value = "idle";
  }, 3000);
};

watch(
  () => state.value,
  (newState) => {
    if (newState === "idle") {
      handleLevel(queue[0]);
    }

    if (newState === "waiting") {
      waitingLevel();
    }
  }
)

const elevatorUp = () => {
  offsetTop.value -= 1;
};

const elevatorDown = () => {
  offsetTop.value += 1;
};

const openDoors = () => {
  doorWidth.value += 1;
};

const closeDoors = () => {
  doorWidth.value -= 1;
};
</script>

<template>
  <div class="building">
    <ul class="levels">
      <li v-for="(i, index) in 5" :key="index" class="level">
        <button
          :disabled="
            queue.includes(i - 1) ||
            (currentLevel === i - 1 && ['waiting', 'moving'].includes(state))
          "
          @click="toggleLevel(i - 1)"
        >
          {{ i }}
        </button>
      </li>
    </ul>
    <div :style="`top: ${offsetTop}px`" class="elevator">
      <div
        :style="`transform: translateX(-${doorWidth}px)`"
        class="elevator-door"
      ></div>
      <div
        :style="`transform: translateX(${doorWidth}px)`"
        class="elevator-door"
      ></div>
    </div>
    <Elevator :queue="queue" />
    <div>{{ currentLevel }}</div>
    <div>{{ queue }}</div>
    <div>{{ state }}</div>
  </div>
</template>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.building {
  position: relative;
}

.level {
  height: 80px;

  background-color: lime;
}

.elevator {
  position: absolute;
  overflow: hidden;
  top: 0;
  left: 0;

  display: flex;

  width: 50px;
  height: 80px;

  border: red solid 2px;
}

.elevator-door {
  width: 25px;
  height: 100%;

  background-color: red;
}
</style>
