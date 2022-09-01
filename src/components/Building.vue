<script setup>
import { ref, reactive } from 'vue';
import Elevator from '@/components/Elevator.vue';

const queue = reactive([]);
const targetFloor = ref(0);
const state = ref('idle');

const elevatorOptions = {
  doorSpeed: 1000,
  waiting: 3000,
  speed: 1000,
};

const numberOfFloors = 5;

const toggleFloor = (i) => {
  if (!queue?.includes(i) && targetFloor.value !== i) {
    queue?.push(i);
  }
};

const changeFloor = (floor) => {
  targetFloor.value = floor;
};

const changeState = (newState) => {
  state.value = newState;
}
</script>

<template>
  <div class="building">
    <ul class="floors">
      <li v-for="(i, index) in numberOfFloors" :key="index" class="floor">
        <button
          class="button"
          :class="{
            'button--processing': queue.includes(i - 1),
            'button--current': targetFloor === i - 1 && state !== 'idle',
          }"
          :disabled="queue.includes(i - 1) || targetFloor === i - 1"
          @click="toggleFloor(i - 1)"
        >
          {{ i }}
        </button>
      </li>
    </ul>
    <div class="elevators">
      <Elevator
        :number-of-floors="numberOfFloors"
        :options="elevatorOptions"
        :queue="queue"
        @change-floor="changeFloor"
        @change-state="changeState"
      />
    </div>
    <div>{{ targetFloor }}</div>
    <div>{{ queue }}</div>
  </div>
</template>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.building {
  position: relative;

  display: flex;
}

.floors {
  min-width: 600px;
  margin: 0;
  padding: 0;

  list-style-type: none;

  border: 2px solid gray;
}

.floor {
  position: relative;

  height: 120px;

  border-bottom: 1px solid black;
}

.floor:last-child {
  border-bottom: 0;
}

.elevators {
  display: flex;
}

.button {
  font-weight: 700;

  position: absolute;
  top: 20px;
  right: 20px;

  display: flex;
  align-items: center;
  justify-content: center;

  width: 25px;
  height: 25px;

  cursor: pointer;

  border: 2px solid gray;
  border-radius: 50%;
  background-color: transparent;
}

.button:disabled {
  color: inherit;
}

.button:disabled:active {
  border-color: red;
}

.button--processing {
  border-color: orange;
}

.button--current {
  animation: color-animate 0.5s infinite;
  border-color: cornflowerblue;
}

@keyframes color-animate {
  0% {
    border-color: cornflowerblue;
  }

  50% {
    border-color: orange;
  }

  100% {
    border-color: cornflowerblue;
  }
}
</style>
