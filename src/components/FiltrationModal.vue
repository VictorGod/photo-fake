<template>
  <div class="filter-modal">
    <div class="header">
      <button class="close-btn" @click="close">X</button>
    </div>
    <div class="filter-graph">
      <select class="preset-select" @change="setPreset($event.target.value)">
        <option value="identity" title="Идентичное преобразование">Идентичное</option>
        <option value="sharpen" title="Фильтр резкости">Резкость</option>
        <option value="gaussian" title="Гауссовский фильтр (3x3)">Гаусс</option>
        <option value="boxBlur" title="Фильтр размытия">Размытие</option>
      </select>
      <div class="filter-content">
        <div class="inputs">
          <div class="matrix-inputs">
            <div v-for="(row, rowIndex) in matrix" class="matrix-column" :key="rowIndex">
              <div v-for="(value, colIndex) in row" :key="colIndex" class="matrix-cell">
                <input
                  :value="matrix[rowIndex][colIndex]"
                  @change="updateMatrix($event, rowIndex, colIndex)"
                  :disabled="previewEnabled"
                />
              </div>
            </div>
          </div>
        </div>
        <div class="actions">
          <div class="checkbox">
            <input type="checkbox" v-model="previewEnabled" />
            <span v-if="previewEnabled" class="checkbox-label">Предпросмотр</span>
            <span v-else class="checkbox-label">Предпросмотр выключен</span>
          </div>
          <button class="btn apply-btn" @click="applyFilter">Применить</button>
          <button class="btn reset-btn" @click="resetFilter">Сбросить</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "FilterModal",
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
      previewEnabled: false,
      matrix: [
        [0, 0, 0],
        [0, 1, 0],
        [0, 0, 0],
      ],
    };
  },
  methods: {
    close() {
      this.$emit("close");
    },
    setPreset(preset) {
      switch (preset) {
        case "identity":
          this.matrix = [
            [0, 0, 0],
            [0, 1, 0],
            [0, 0, 0],
          ];
          break;
        case "sharpen":
          this.matrix = [
            [0, -1, 0],
            [-1, 5, -1],
            [0, -1, 0],
          ];
          break;
        case "gaussian":
          this.matrix = [
            [1, 2, 1],
            [2, 4, 2],
            [1, 2, 1],
          ];
          break;
        case "boxBlur":
          this.matrix = [
            [1, 1, 1],
            [1, 1, 1],
            [1, 1, 1],
          ];
          break;
        default:
          break;
      }
    },
    updateMatrix(event, rowIndex, colIndex) {
      const num = +event.target.value;
      if (!isNaN(num)) {
        this.matrix[rowIndex][colIndex] = num;
      } else {
        event.target.value = this.matrix[rowIndex][colIndex];
      }
    },
    applyFilter() {
      this.$emit("close");
      this.calculateFilter();
    },
    resetFilter() {
      this.$emit("revertNewImg");
      this.previewEnabled = false;
      this.setPreset("identity");
    },
    calculateFilter() {
      const ctx = this.canvasRef?.getContext("2d");
      const imageData = ctx.getImageData(this.dx, this.dy, this.iw, this.ih);
      const newData = new Uint8ClampedArray(imageData.data.length);

      const paddedData = this.padImageData(imageData.data, imageData.width, imageData.height);

      for (let y = 0; y < imageData.height; y++) {
        for (let x = 0; x < imageData.width; x++) {
          for (let c = 0; c < 4; c++) {
            const outputIndex = (y * imageData.width + x) * 4 + c;
            let sum = 0;
            let kernelSum = 0;
            for (let ky = 0; ky < 3; ky++) {
              for (let kx = 0; kx < 3; kx++) {
                const inputIndex = ((y + ky) * (imageData.width + 2) + (x + kx)) * 4 + c;
                sum += paddedData[inputIndex] * this.matrix[ky][kx];
                kernelSum += this.matrix[ky][kx];
              }
            }
            newData[outputIndex] = sum / kernelSum;
          }
        }
      }

      imageData.data.set(newData);
      ctx.putImageData(imageData, this.dx, this.dy);
    },
    padImageData(data, width, height) {
      const paddedWidth = width + 2;
      const paddedHeight = height + 2;
      const paddedData = new Uint8ClampedArray(paddedWidth * paddedHeight * 4);

      for (let y = 0; y < height; y++) {
        for (let x = 0; x < width; x++) {
          const inputIndex = (y * width + x) * 4;
          const outputIndex = ((y + 1) * paddedWidth + x + 1) * 4;
          paddedData.set(data.subarray(inputIndex, inputIndex + 4), outputIndex);
        }
      }

      for (let y = 0; y < paddedHeight; y++) {
        for (let x = 0; x < paddedWidth; x++) {
          const outputIndex = (y * paddedWidth + x) * 4;
          if (x === 0 || x === paddedWidth - 1 || y === 0 || y === paddedHeight - 1) {
            const nearestX = Math.max(1, Math.min(x, paddedWidth - 2));
            const nearestY = Math.max(1, Math.min(y, paddedHeight - 2));
            const nearestIndex = (nearestY * paddedWidth + nearestX) * 4;
            paddedData.set(paddedData.subarray(nearestIndex, nearestIndex + 4), outputIndex);
          }
        }
      }
      return paddedData;
    },
  },
  watch: {
    previewEnabled(newVal) {
      if (newVal) {
        this.calculateFilter();
      } else {
        this.resetFilter();
      }
    },
  },
});
</script>

<style scoped lang="scss">
.filter-modal {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 220px;
  padding: 20px;
  background: #ffffff;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  font-family: 'Arial', sans-serif;

  .header {
    display: flex;
    justify-content: flex-end;

    .close-btn {
      font-size: 18px;
      cursor: pointer;
      color: #333;
      background: transparent;
      border: none;
      &:hover {
        color: #ff0000;
      }
    }
  }

  .filter-graph {
    display: flex;
    flex-direction: column;
    align-items: center;

    .preset-select {
      margin-bottom: 10px;
      padding: 5px;
      border: 1px solid #ccc;
      border-radius: 5px;
    }
  }

  .filter-content {
    display: flex;
    flex-direction: column;
    gap: 10px;
    width: 100%;

    .inputs {
      .matrix-inputs {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 5px;

        .matrix-column {
          display: flex;
          gap: 5px;

          .matrix-cell {
            input {
              width: 30px;
              height: 30px;
              text-align: center;
              border: 1px solid #ccc;
              border-radius: 5px;
              &:disabled {
                background-color: #f5f5f5;
              }
            }
          }
        }
      }
    }

    .actions {
      display: flex;
      justify-content: space-between;
      align-items: flex-start;
      gap: 10px;
      flex-direction: column;

      .checkbox {
        display: flex;
        align-items: center;
        input {
          margin-right: 5px;
        }
        .checkbox-label {
          font-size: 14px;
        }
      }

      .btn {
        padding: 5px 10px;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        transition: background-color 0.3s ease;
        &:hover {
          background-color: #f0f0f0;
        }
      }

      .apply-btn {
        background-color: #4caf50;
        color: #fff;
      }

      .reset-btn {
        background-color: #f44336;
        color: #fff;
      }
    }
  }
}
</style>
