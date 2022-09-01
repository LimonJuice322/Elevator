<template>
  <div class="shaft">
    <div :style="`top: ${offsetTop}px`" class="elevator">
      <div
        :style="`transform: translateX(-${doorOffset}px)`"
        class="elevator-door"
      ></div>
      <div
        :style="`transform: translateX(${doorOffset}px)`"
        class="elevator-door"
      ></div>
    </div>
    <div>{{ currentFloor }}</div>
  </div>
</template>

<script setup>
import { reactive, ref, watch } from "vue";

const props = defineProps({
  numberOfFloors: Number,
  queue: Array,
  options: Object,
})

const emit = defineEmits(['change-floor']);

const elevatorQueue = reactive(props.queue);
const offsetTop = ref(0);
const doorOffset = ref(0);
const state = ref('idle');
const currentFloor = ref(0);

const elevatorParams = {
  floorHeight: 80,
  doorWidth: 25,
}

const options = {
  speed: Math.floor(props.options.speed / elevatorParams.floorHeight),
  waiting: {
    total: props.options.waiting,
    doorSpeed: Math.floor(props.options.doorSpeed / elevatorParams.doorWidth),
    segments: Math.floor(props.options.waiting / props.options.doorSpeed),
  },
}

watch(() => elevatorQueue.at(-1), (newVal) => {
  if (state.value === 'idle') {
    handleFloor(newVal);
  }
})

watch(() => offsetTop.value, (offset) => {
  for (let i = 0; i < props.numberOfFloors; i++) {
    if (Math.floor(offset / elevatorParams.floorHeight) === i) {
      currentFloor.value = i;
    }
  }
})

const handleFloor = (i) => {
  let nextOffset = i * elevatorParams.floorHeight;
  let offset = Math.abs(nextOffset - offsetTop.value);

  if (nextOffset > offsetTop.value) {
    moveElevator(elevatorDown, offset);
  }

  if (nextOffset < offsetTop.value) {
    moveElevator(elevatorUp, offset);
  }
};

const moveElevator = (callback, offset) => {
  if (offset === 0) {
    state.value = 'waiting';
    return;
  }

  for (let i = 0; i < offset; i++) {
    setTimeout(() => {
      if (i === 0) {
        emit('change-floor', elevatorQueue.shift());
        state.value = 'moving';
      }

      if (i === offset - 1) {
        state.value = 'waiting';
      }

      callback();
    }, options.speed * i);
  }
};

const waitingFloor = () => {
  for (let i = 0; i < elevatorParams.doorWidth * options.waiting.segments; i++) {
    if (i < elevatorParams.doorWidth) {
      setTimeout(openDoors, options.waiting.doorSpeed * i);
    }

    if (i >= elevatorParams.doorWidth * (options.waiting.segments - 1)) {
      setTimeout(closeDoors, options.waiting.doorSpeed * i);
    }
  }

  setTimeout(() => {
    state.value = 'idle';
  }, options.waiting.total);
};

watch(
  () => state.value,
  (newState) => {
    if (newState === 'idle') {
      handleFloor(elevatorQueue[0]);
    }

    if (newState === 'waiting') {
      waitingFloor();
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
  doorOffset.value += 1;
};

const closeDoors = () => {
  doorOffset.value -= 1;
};
</script>

<style scoped>
.shaft {
  position: absolute;
  top: 0;
  left: -100px;
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
