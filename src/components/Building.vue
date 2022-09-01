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

const changeFloor = (level) => {
  targetFloor.value = level;
};
</script>

<template>
  <div class="building">
    <ul class="levels">
      <li v-for="(i, index) in numberOfFloors" :key="index" class="level">
        <button
          :disabled="queue.includes(i - 1) || targetFloor === i - 1"
          @click="toggleFloor(i - 1)"
        >
          {{ i }}
        </button>
      </li>
    </ul>
    <Elevator
      :number-of-floors="numberOfFloors"
      :options="elevatorOptions"
      :queue="queue"
      @change-floor="changeFloor"
    />
    <div>{{ targetFloor }}</div>
    <div>{{ queue }}</div>
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
