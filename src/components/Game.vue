<template>
  <div class="flex flex-col items-center">
    <div ref="leftDice" style="width: 200px; height: 200px; margin-bottom: 20px"></div>
    <div ref="rightDice" style="width: 200px; height: 200px"></div>
    <button
      @click="rollDiceAndCheck('2-6')"
      style="background-color: #3490dc; color: white"
      class="px-4 py-2 mt-2"
    >
      2 to 6
    </button>
    <button
      @click="rollDiceAndCheck('7-12')"
      style="background-color: #3490dc; color: white"
      class="px-4 py-2 mt-2"
    >
      7 to 12
    </button>
    <div v-if="loading" class="mt-4">
      <p class="text-white">Loading...</p>
    </div>
    <div v-else-if="image" class="mt-4">
      <p class="text-white; mt-4">{{ resultMessage }}</p>
      <img :src="image" alt="image" />
    </div>
  </div>
</template>

<script>
import lottie from "lottie-web";
import axios from "axios"; 

export default {
  data() {
    return {
      leftDiceAnimation: null,
      rightDiceAnimation: null,
      userPrediction: "2-6",
      resultMessage: "",
      image: "",
      loading: false,
      sevenTo12: [
        { dice_1: 1, dice_2: 6, Sum: 7 },
        { dice_1: 2, dice_2: 5, Sum: 7 },
        { dice_1: 3, dice_2: 4, Sum: 7 },
        { dice_1: 4, dice_2: 3, Sum: 7 },
        { dice_1: 5, dice_2: 2, Sum: 7 },
        { dice_1: 6, dice_2: 1, Sum: 7 },
        { dice_1: 2, dice_2: 6, Sum: 8 },
        { dice_1: 3, dice_2: 5, Sum: 8 },
        { dice_1: 4, dice_2: 4, Sum: 8 },
        { dice_1: 5, dice_2: 3, Sum: 8 },
        { dice_1: 6, dice_2: 2, Sum: 8 },
        { dice_1: 3, dice_2: 6, Sum: 9 },
        { dice_1: 4, dice_2: 5, Sum: 9 },
        { dice_1: 5, dice_2: 4, Sum: 9 },
        { dice_1: 6, dice_2: 3, Sum: 9 },
        { dice_1: 4, dice_2: 6, Sum: 10 },
        { dice_1: 5, dice_2: 5, Sum: 10 },
        { dice_1: 6, dice_2: 4, Sum: 10 },
        { dice_1: 5, dice_2: 6, Sum: 11 },
        { dice_1: 6, dice_2: 5, Sum: 11 },
        { dice_1: 6, dice_2: 6, Sum: 12 },
      ],
      twoTo6: [
        { die_1: 1, die_2: 1, sum: 2 },
        { die_1: 1, die_2: 2, sum: 3 },
        { die_1: 1, die_2: 3, sum: 4 },
        { die_1: 1, die_2: 4, sum: 5 },
        { die_1: 1, die_2: 5, sum: 6 },
        { die_1: 2, die_2: 1, sum: 3 },
        { die_1: 2, die_2: 2, sum: 4 },
        { die_1: 2, die_2: 3, sum: 5 },
        { die_1: 2, die_2: 4, sum: 6 },
        { die_1: 3, die_2: 1, sum: 4 },
        { die_1: 3, die_2: 2, sum: 5 },
        { die_1: 3, die_2: 3, sum: 6 },
        { die_1: 4, die_2: 1, sum: 5 },
        { die_1: 4, die_2: 2, sum: 6 },
        { die_1: 5, die_2: 1, sum: 6 },
      ],
    };
  },
  mounted() {},
  methods: {
    async checkWin() {
      this.loading = true;
      const res = await axios.get("https://yesno.wtf/api");
      this.loading = false;
      return res.data
    },
    rollDice(isWin, minRange, maxRange) {
      let rangeArray, leftRollKey, rightRollKey;

      if (minRange === "2" && maxRange === "6") {
        rangeArray = isWin ? this.twoTo6 : this.sevenTo12;
        leftRollKey = isWin ? "die_1" : "dice_1";
        rightRollKey = isWin ? "die_2" : "dice_2";
      } else if (minRange === "7" && maxRange === "12") {
        rangeArray = isWin ? this.sevenTo12 : this.twoTo6;
        leftRollKey = isWin ? "dice_1" : "die_1";
        rightRollKey = isWin ? "dice_2" : "die_2";
      } else {
        return null; // Handle invalid range here
      }

      const randomIndex = Math.floor(Math.random() * rangeArray.length);
      const { [leftRollKey]: leftRoll, [rightRollKey]: rightRoll } = rangeArray[
        randomIndex
      ];

      return {
        leftRoll,
        rightRoll,
      };
    },
    async loadAnimations(leftRoll, rightRoll) {
      // Clean up old animations before loading new ones
      if (this.leftDiceAnimation) {
        this.leftDiceAnimation.destroy();
      }
      if (this.rightDiceAnimation) {
        this.rightDiceAnimation.destroy();
      }

      // Load left dice animation based on roll
      const leftAnimation = await import(`@/assets/L${leftRoll}.json`); // Load from src/assets
      this.leftDiceAnimation = lottie.loadAnimation({
        container: this.$refs.leftDice,
        animationData: leftAnimation.default,
        loop: true,
        autoplay: true,
      });

      // Load right dice animation based on roll
      const rightAnimation = await import(`@/assets/R${rightRoll}.json`); // Load from src/assets
      this.rightDiceAnimation = lottie.loadAnimation({
        container: this.$refs.rightDice,
        animationData: rightAnimation.default,
        loop: true,
        autoplay: true,
      });
    },

    async rollDiceAndCheck(predictionRange) {
      this.userPrediction = predictionRange;
      const { answer, image }  = await this.checkWin();
      const isWin = answer === "yes";
      const [minRange, maxRange] = this.userPrediction.split("-");
      const { leftRoll, rightRoll } = this.rollDice(isWin, minRange, maxRange);
      const sum = leftRoll + rightRoll;
      this.image = image

      if (sum >= parseInt(minRange) && sum <= parseInt(maxRange)) {
        this.resultMessage = `Congratulations! You rolled ${sum} (${leftRoll} + ${rightRoll}). Your prediction was correct!`;
        // Trigger a celebratory animation
      } else {
        this.resultMessage = `Sorry, you rolled ${sum} (${leftRoll} + ${rightRoll}). The correct sum was between ${minRange} and ${maxRange}. Try again!`;
        // Trigger a disappointed animation
      }

      // Load new animations for the next round
      this.loadAnimations(leftRoll, rightRoll);
    },
  },
};
</script>

<style scoped>
/* Tailwind CSS classes can be used here */
</style>
