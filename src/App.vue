<script lang="ts">
import { computed, defineComponent, reactive, watch } from "vue";
import Square from "./components/square/Square.vue";
import Button from "./components/button/Button.vue";

export default defineComponent({
  name: "App",
  components: {
    Square,
    Button,
  },
  setup() {
    const state = reactive({
      values: [10, 20, 30, 100, 50, 70, 65, 45, 15],
      active: Array(9).fill(false),
      total: 0,
      timer: 5,
      gameStart: false,
      stopGameId: null as number | null,
      randomShuffleId: null as number | null,
    });

    const setTimerHandler = (): void => {
      if (state.timer > 0) {
        state.timer -= 1;
      }
    };

    const setValuesHandler = (): void => {
      const shuffledArray = [...state.values].sort(() => Math.random() - 0.5);
      state.values = shuffledArray;
    };

    const setActiveHandler = (): void => {
      const randomIndex = Math.floor(Math.random() * state.active.length);
      const newActive = state.active.map((_, index) => (index === randomIndex ? true : false));
      state.active = newActive;
    };

    const resetActiveHandler = (): void => {
      state.active.fill(false);
    };

    const resetTimer = (): void => {
      state.timer = 5;
    };

    const setGameStartHandler = (): void => {
      state.gameStart = !state.gameStart;

      if (state.gameStart) {
        state.stopGameId = setTimeout(() => {
          setGameStartHandler();
        }, 5000);

        state.randomShuffleId = setInterval(() => {
          setActiveHandler();
          setValuesHandler();
          setTimerHandler();
        }, 1000);
      } else {
        clearTimeout(state.stopGameId!);
        clearInterval(state.randomShuffleId!);
        resetActiveHandler();
        resetTimer();
      }
    };

    const setTotalHandler = (value: number): void => {
      if (state.gameStart) {
        state.total += value;
      }
    };

    const resetTotalHandler = (): void => {
      state.total = 0;
    };

    const updatedTimer = computed(() => state.timer);
    const updatedTotal = computed(() => state.total);
    const updatedActive = computed(() => state.active);
    const updatedGameStartStatus = computed(() => state.gameStart);

    return {
      ...state,
      setGameStartHandler,
      resetTotalHandler,
      setTotalHandler,
      updatedTimer,
      updatedTotal,
      updatedActive,
      updatedGameStartStatus,
    };
  },
});
</script>

<template>
  <div class="game-area">
    <div class="game-area__controllers">
      <Button use="pay" size="large" title="Начать игру" :clickHandler="setGameStartHandler" />
      <Button use="primary" size="large" title="Сбросить счёт" :clickHandler="resetTotalHandler" />
    </div>
    <div class="game-area__timer">Время: {{ updatedTimer }} секунд (-ы)</div>
    <span class="game-area__total">Счёт: {{ updatedTotal }}</span>
    <div :class="['game-area__wrapper', { 'game-area__wrapper--not-active': !updatedGameStartStatus }]">
      <Square
        v-for="(value, i) in values"
        :value="value"
        :active="updatedActive[i]"
        :clickHandler="() => setTotalHandler(value)"
        :key="i"
      />
    </div>
  </div>
</template>

<style scoped>
.game-area {
  display: flex;
  height: 100vh;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  gap: 20px;
}

.game-area__wrapper {
  width: 340px;
  height: 320px;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.game-area__wrapper--not-active {
  opacity: 0.3;
}

.game-area__controllers {
  display: flex;
  gap: 10px;
}

.game-area__timer,
.game-area__total {
  color: #fff;
  font-size: 18px;
}
</style>
