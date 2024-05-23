<template>
  <div class="correction-panel">
    <div class="panel-header">
      <button class="close-btn" @click="close">×</button>
    </div>
    <div class="correction-inputs">
      <div class="point-inputs">
        <div
          class="point"
          v-for="(label, index) in labels"
          :key="index"
        >
          <label :for="label">{{ label }}</label>
          <input
            :id="label"
            :value="values[label]"
            @change="update(label, $event)"
            :disabled="preview"
          />
        </div>
      </div>
      <div class="correction-preview">
        <div class="custom-checkbox">
          <input type="checkbox" v-model="preview" />
          <span v-if="preview" class="checkbox-text">Выключить предпросмотр</span>
          <span v-else class="checkbox-text">Включить предпросмотр</span>
        </div>
      </div>
    </div>
    <div class="correction-actions">
      <button
        class="apply-btn"
        @click="apply"
        :disabled="corrected"
      >
        Применить
      </button>
      <button class="reset-btn" @click="reset">
        Сбросить
      </button>
    </div>
    <div class="correction-chart">
      <canvas id="histogram-chart" width="100" height="100" ref="chart"></canvas>
    </div>
  </div>
</template>

<script>
import { defineComponent } from "vue";
import Chart from "chart.js/auto";

export default defineComponent({
  name: "CorrectionPanel",
  props: {
    state: String,
    canvasRef: Object,
    origImg: Object,
    newImg: Object,
    dx: Number,
    dy: Number,
    ih: Number,
    iw: Number,
  },
  data() {
    return {
      values: {
        x1: 0,
        y1: 0,
        x2: 255,
        y2: 255,
      },
      labels: ["x1", "y1", "x2", "y2"],
      preview: false,
      chartInstance: null,
      corrected: false,
    };
  },
  mounted() {
    this.chartRef = this.$refs.chart;
  },
  methods: {
    close() {
      this.$emit("close");
    },
    normalize(histogram) {
      const max = Math.max(...histogram);
      return histogram.map(value => (value / max) * 255);
    },
    buildGraph() {
      const img = new Image();
      img.onload = () => {
        const ctx = this.canvasRef?.getContext("2d");
        try {
          const imageData = ctx.getImageData(this.dx, this.dy, this.iw, this.ih);
          const data = imageData.data;
          const redHist = new Array(256).fill(0);
          const greenHist = new Array(256).fill(0);
          const blueHist = new Array(256).fill(0);

          for (let i = 0; i < data.length; i += 4) {
            redHist[data[i]]++;
            greenHist[data[i + 1]]++;
            blueHist[data[i + 2]]++;
          }

          this.drawHistograms(
            this.normalize(redHist),
            this.normalize(greenHist),
            this.normalize(blueHist)
          );
        } catch (e) {
          console.error(e);
        }
      };
      img.src = this.origImg.src;
    },
    drawHistograms(redHist, greenHist, blueHist) {
      const ctx = this.chartRef?.getContext("2d");

      if (this.chartInstance) {
        this.chartInstance.destroy();
      }

      this.chartInstance = new Chart(ctx, {
        type: "line",
        data: {
          labels: Array.from({ length: 256 }, (_, i) => i),
          datasets: [
            {
              label: "Красный",
              data: redHist,
              backgroundColor: "rgba(255, 0, 0, 0.8)",
            },
            {
              label: "Зеленый",
              data: greenHist,
              backgroundColor: "rgba(0, 255, 0, 0.8)",
            },
            {
              label: "Синий",
              data: blueHist,
              backgroundColor: "rgba(0, 0, 255, 0.8)",
            },
            {
              label: "Линия",
              data: [
                { x: 0, y: this.values.y1 },
                { x: this.values.x1, y: this.values.y1 },
                { x: this.values.x2, y: this.values.y2 },
                { x: 255, y: this.values.y2 },
              ],
              borderColor: "rgba(0,0,0,1)",
              borderWidth: 1,
              fill: false,
            },
          ],
        },
        options: {
          animation: false,
          scales: {
            x: {
                type: "linear",
                position: "bottom",
                min: 0,
                max: 260,
                ticks: {
                    stepSize: 15,
                },
            },
            y: {
                beginAtZero: true,
                min: 0,
                max: 260,
                ticks: {
                    stepSize: 15,
                },
            },
          },
        },
      });
    },
    calculate() {
      try {
        const lut = [];
        for (let i = 0; i < this.values.x1; i++) {
          lut[i] = this.values.y1;
        }
        for (let i = this.values.x1; i < this.values.x2; i++) {
          const slope =
            (this.values.y2 - this.values.y1) / (this.values.x2 - this.values.x1);
          let correctedValue = this.values.y1 + slope * (i - this.values.x1);
          correctedValue = Math.max(0, Math.min(255, correctedValue));
          lut[i] = correctedValue;
        }
        for (let i = this.values.x2; i < 256; i++) {
          lut[i] = this.values.y2;
        }

        const ctx = this.canvasRef?.getContext("2d");
        const imageData = ctx.getImageData(this.dx, this.dy, this.iw, this.ih);
        const data = imageData.data;
        for (let i = 0; i < data.length; i += 4) {
          data[i] = lut[data[i]]; // Коррекция красного канала
          data[i + 1] = lut[data[i + 1]]; // Коррекция зеленого канала
          data[i + 2] = lut[data[i + 2]]; // Коррекция синего канала
        }

        ctx.putImageData(imageData, this.dx, this.dy);
        this.$emit("updateNewImgData", data);
        this.corrected = true;
      } catch (e) {
        console.log(e);
      }
    },
    revert() {
      this.$emit("revertNewImg");
      this.corrected = false;
    },
    apply() {
      this.$emit("close");
      this.calculate();
      this.buildGraph();
    },
    reset() {
      this.values.x1 = 0;
      this.values.y1 = 0;
      this.values.x2 = 255;
      this.values.y2 = 255;
      this.revert();
    },
    update(label, event) {
      const num = +event.target.value;
      const max = 255;
      const min = 0;

      if (!isNaN(num) && num >= min && num <= max) {
        if (label === 'x1' && num > this.values.x2) {
          alert('x1 не может быть больше x2');
          event.target.value = this.values[label];
        } else if (label === 'x2' && num < this.values.x1) {
          alert('x2 не может быть меньше x1');
          event.target.value = this.values[label];
        } else {
          this.values[label] = num;
          this.buildGraph();
        }
      } else {
        event.target.value = this.values[label]; // reset input to valid value
      }
    },
  },
  watch: {
    newImg() {
      this.buildGraph();
    },
    preview(newVal) {
      if (newVal) {
        this.calculate();
      } else {
        this.revert();
      }
    },
  },
});
</script>

<style scoped lang="scss">
.correction-panel {
  position: absolute;
  top: 0;
  right: 0;
  width: 250px;
  background: #f0f0f0;
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 10px 15px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);

  .panel-header {
    display: flex;
    justify-content: flex-end;

    .close-btn {
      font-size: 20px;
      cursor: pointer;
      font-weight: bold;
      color: #444;
      background: transparent;
      border: none;
    }
  }

  .correction-inputs {
    display: flex;
    flex-direction: column;
    gap: 10px;
    .point-inputs {
      display: flex;
      flex-direction: column;
      gap: 5px;

      .point {
        display: flex;
        align-items: center;
        gap: 5px;

        label {
          font-size: 14px;
          color: #444;
          width: 30px;
        }

        input {
          flex: 1;
          padding: 5px;
          font-size: 14px;
          border: 1px solid #ccc;
          border-radius: 5px;
          background: #fff;
        }
      }
    }

    .correction-preview {
      display: flex;
      align-items: center;
      gap: 5px;

      .custom-checkbox {
        display: flex;
        align-items: center;
        gap: 5px;

        input[type="checkbox"] {
          width: 18px;
          height: 18px;
          cursor: pointer;
        }

        .checkbox-text {
          width: 110px;
          font-size: 14px;
          color: #444;
        }
      }
    }
  }

  .correction-actions {
    display: flex;
    gap: 10px;

    .correction-button {
      flex: 1;
      padding: 10px;
      font-size: 14px;
      color: #fff;
      background: #007bff;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      text-align: center;

      &.apply {
        background: #28a745;
      }

      &.reset {
        background: #dc3545;
      }

      &:disabled {
        background: #6c757d;
        cursor: not-allowed;
      }
    }
  }

  .correction-chart {
    flex: 1;
    background: #fff;
    border: 1px solid #ccc;
    border-radius: 5px;
    padding: 10px;
  }
}
</style>