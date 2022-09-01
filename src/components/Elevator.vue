<script setup>
import { reactive, ref, watch } from "vue";

const props = defineProps({
  numberOfFloors: Number,
  queue: Array,
  options: Object,
})

const emit = defineEmits(['change-floor', 'change-state']);

const elevatorQueue = reactive(props.queue);
const offsetTop = ref(0);
const doorOffset = ref(0);
const state = ref('idle');
const currentFloor = ref(0);

const shaftParams = {
  floorHeight: 120,
  doorWidth: 40,
  floorBorderWidth: 1,
}

const options = {
  speed: Math.floor(props.options.speed / shaftParams.floorHeight),
  waiting: {
    total: props.options.waiting,
    doorSpeed: Math.floor(props.options.doorSpeed / shaftParams.doorWidth),
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
    if (Math.floor(offset / shaftParams.floorHeight) === i) {
      currentFloor.value = i;
    }
  }
})

const handleFloor = (i) => {
  let nextOffset = (i * shaftParams.floorHeight) + i + 1;
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
    changeState('waiting');
    return;
  }

  for (let i = 0; i < offset; i++) {
    setTimeout(() => {
      if (i === 0) {
        emit('change-floor', elevatorQueue.shift());
        changeState('moving');
      }

      if (i === offset - 1) {
        changeState('waiting');
      }

      callback();
    }, options.speed * i);
  }
};

const waitingFloor = () => {
  for (let i = 0; i < shaftParams.doorWidth * options.waiting.segments; i++) {
    if (i < shaftParams.doorWidth) {
      setTimeout(openDoors, options.waiting.doorSpeed * i);
    }

    if (i >= shaftParams.doorWidth * (options.waiting.segments - 1)) {
      setTimeout(closeDoors, options.waiting.doorSpeed * i);
    }
  }

  setTimeout(() => {
    changeState('idle');
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

const changeState = (newState) => {
  state.value = newState;
  emit('change-state', newState);
}

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

<style scoped>
.shaft {
  position: relative;

  width: 80px;
  height: 100%;
}

.elevator {
  position: absolute;
  overflow: hidden;
  top: 0;
  left: 0;

  display: flex;

  width: 80px;
  height: 120px;

  border: lightcoral solid 1px;
}

.elevator-door {
  width: 40px;
  height: 100%;

  background-color: lightblue;
}
</style>
