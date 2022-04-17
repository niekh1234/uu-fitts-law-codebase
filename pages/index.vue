<template>
  <div class="flex items-center justify-center w-screen h-screen bg-grey-darkest">
    <p v-if="activeCircle !== null" class="absolute text-xl font-bold text-white top-4 left-4">
      {{ game.length - 1 }}
    </p>
    <section v-if="activeCircle === null && !hasEnded" class="flex flex-col max-w-2xl p-4">
      <h1 class="text-4xl font-bold text-grey-lighter">Fitt's law experiment</h1>
      <h2 class="mt-4 text-3xl font-semibold text-grey-light">
        Gather a lot of data using random circle parameters.
      </h2>
      <div class="inline-flex items-center justify-between mt-12">
        <div class="flex flex-col">
          <label for="range_slider" class="text-grey-light"
            >Number of circles to complete: {{ maximumCircles }}</label
          >
          <input
            v-model="maximumCircles"
            id="range_slider"
            type="range"
            min="1"
            max="500"
            class="w-64"
          />
        </div>

        <div class="flex flex-col">
          <label for="enable_difficulty" class="text-grey-light">Enable varying width</label>
          <input v-model="useDifficulty" type="checkbox" />
        </div>
      </div>
      <button
        @click="setNewActiveCircle"
        class="px-6 py-3 mt-8 mr-auto text-lg font-bold transition-colors duration-200 bg-white rounded hover:bg-grey-lighter text-grey-darkest"
      >
        Start now
      </button>
    </section>

    <section v-if="activeCircle === null && hasEnded" class="max-w-2xl p-4">
      <h2 class="text-4xl font-bold text-grey-lighter">Thank you for participating!</h2>
      <h2 class="mt-4 text-3xl font-semibold text-grey-light">
        Download your data below:
      </h2>
      <button
        @click="getCsv()"
        class="px-6 py-3 mt-8 text-lg font-bold transition-colors duration-200 bg-white rounded hover:bg-grey-light text-grey-darkest"
      >
        Download CSV
      </button>
    </section>
    <GameCircle v-if="activeCircle" @pressed="handleNext()" :circle="activeCircle"></GameCircle>
  </div>
</template>

<script>
export default {
  data() {
    return {
      maximumCircles: 50,
      game: [],
      activeCircle: null,
      hasEnded: false,
      useDifficulty: true,
    };
  },

  methods: {
    handleNext() {
      if (this.game.length < this.maximumCircles - 1) {
        this.setNewActiveCircle();
      } else {
        this.activeCircle = null;
        this.hasEnded = true;
      }
    },

    setNewActiveCircle() {
      if (this.game.length > 0) {
        this.activeCircle.timeTaken =
          new Date().getTime() - new Date(this.activeCircle.timeGenerated).getTime();
      } else {
        this.activeCircle = this.createCircle();
        this.game.push(this.activeCircle);
        return;
      }

      if (this.game.length > 1) {
        this.activeCircle.distance = this.getEuclidean();
        this.activeCircle.fitts = this.calcFitts(this.activeCircle);
      }

      this.activeCircle = this.createCircle();
      this.game.push(this.activeCircle);
    },

    createCircle() {
      const size = 25 + Math.random() * 100;
      const randomCircle = {
        size: this.useDifficulty ? size : 100,
        x: (window.innerHeight - size) * Math.random(),
        y: (window.innerWidth - size) * Math.random(),
        timeGenerated: new Date().getTime(),
      };

      return randomCircle;
    },

    getEuclidean() {
      const x1 = this.game[this.game.length - 1].x;
      const y1 = this.game[this.game.length - 1].y;
      const x2 = this.game[this.game.length - 2].x;
      const y2 = this.game[this.game.length - 2].y;
      return Math.sqrt((x1 - y1) ** 2 + (x2 - y2) ** 2);
    },

    calcFitts(node) {
      return Math.log2(2 * (node.distance / node.size));
    },

    getCsv() {
      if (this.game.length === 0) {
        return alert('something went wrong, reload the page');
      }

      let csvContent = 'data:text/csv;charset=utf-8,';
      csvContent +=
        'difficulty(size),x-coord,y-coord,time_generated,time_taken,distance,fitts_estimation\r\n';

      for (const entry of this.game) {
        if (entry.timeTaken === undefined) {
          continue;
        }

        csvContent += Object.values(entry).join(',');
        csvContent += '\r\n';
      }

      const encodedUri = encodeURI(csvContent);
      const link = document.createElement('a');
      link.setAttribute('href', encodedUri);
      link.setAttribute('download', 'fitts-export.csv');
      document.body.appendChild(link);
      link.click();
    },
  },
};
</script>
