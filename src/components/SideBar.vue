<template>
  <div class="color-info-panel">
    <div class="panel-header">
      <button class="close-button" @click="closePanel">x</button>
    </div>
    <div class="color-buttons">
      <input
        v-model="selectedButton"
        value="button1"
        type="radio"
        class="color-button"
        :style="{ background: color1 }"
      />
      <input
        v-model="selectedButton"
        value="button2"
        type="radio"
        class="color-button"
        :style="{ background: color2 }"
      />
    </div>
    <div class="color-details">
      <p class="color-label">X</p>
      <div class="color-values">
        <p>{{ color1X }}</p>
        <p>{{ color2X }}</p>
      </div>
      <p class="color-label">Y</p>
      <div class="color-values">
        <p>{{ color1Y }}</p>
        <p>{{ color2Y }}</p>
      </div>
      <p
        class="color-label"
        title="Цветовая модель, сочетающая интенсивность красного, зеленого и синего света для создания цветов на цифровых дисплеях."
      >
        RGB
      </p>
      <div class="color-values">
        <p>{{ color1RGB }}</p>
        <p>{{ color2RGB }}</p>
      </div>
      <p
        class="color-label"
        title="Стандартное цветовое пространство, представляющее зрительное восприятие цветов человеком, используется в качестве основы для других цветовых пространств."
      >
        XYZ
      </p>
      <div class="color-values">
        <p>{{ color1XYZ }}</p>
        <p>{{ color2XYZ }}</p>
      </div>
      <p
        class="color-label"
        title="Визуально однородное цветовое пространство с компонентами, отвечающими за яркость (L*), и осями цветопередачи (a*, b*), идеально подходящими для точной цветопередачи и коррекции."
      >
        Lab
      </p>
      <div class="color-values">
        <p>{{ color1Lab }}</p>
        <p>{{ color2Lab }}</p>
      </div>
    </div>
    <div
      class="contrast-ratio"
      v-if="contrastRatio"
      :style="{ color: contrastRatio < 4.5 ? 'red' : 'black' }"
    >
      Contrast: {{ contrastRatio }}:1
    </div>
  </div>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "ColorInfoPanel",
  props: {
    pickedColor: String,
    xMouse: Number,
    yMouse: Number,
  },
  data() {
    return {
      selectedButton: null,
      color1: null,
      color2: null,
      color1X: 0,
      color1Y: 0,
      color2X: 0,
      color2Y: 0,
      color1RGB: "",
      color2RGB: "",
      color1XYZ: "",
      color2XYZ: "",
      color1Lab: "",
      color2Lab: "",
      contrastRatio: null,
    };
  },
  methods: {
    closePanel() {
      this.$emit("close");
    },
    updateColors() {
      const rgb = this.parseColor(this.pickedColor);
      const xyz = this.rgbToXyz(rgb);
      const lab = this.xyzToLab(xyz);

      if (this.selectedButton === "button1") {
        this.color1X = this.xMouse;
        this.color1Y = this.yMouse;
        this.color1RGB = this.formatColor(rgb, true);
        this.color1XYZ = this.formatColor(xyz);
        this.color1Lab = this.formatColor(lab);
      } else {
        this.color2X = this.xMouse;
        this.color2Y = this.yMouse;
        this.color2RGB = this.formatColor(rgb, true);
        this.color2XYZ = this.formatColor(xyz);
        this.color2Lab = this.formatColor(lab);
      }

      if (this.color1 && this.color2) {
        this.contrastRatio = this.calculateContrastRatio(this.color1, this.color2);
      }
    },
    formatColor([tone1, tone2, tone3], isRGB = false) {
      if (isRGB) return `${tone1}, ${tone2}, ${tone3}`;
      return `${tone1.toFixed(1)}, ${tone2.toFixed(1)}, ${tone3.toFixed(1)}`;
    },
    calculateContrastRatio(color1, color2) {
      const sRGBToLinear = (c) => {
        c = c / 255;
        return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4);
      };

      const getLuminance = (color) => {
        const rgb = this.parseColor(color);
        const r = sRGBToLinear(rgb[0]);
        const g = sRGBToLinear(rgb[1]);
        const b = sRGBToLinear(rgb[2]);
        return 0.2126 * r + 0.7152 * g + 0.0722 * b;
      };

      const luminance1 = getLuminance(color1);
      const luminance2 = getLuminance(color2);

      const maxLuminance = Math.max(luminance1, luminance2);
      const minLuminance = Math.min(luminance1, luminance2);

      const contrastRatio = (maxLuminance + 0.05) / (minLuminance + 0.05);

      return contrastRatio.toFixed(1);
    },
    parseColor(input) {
      if (input.substr(0, 1) == "#") {
        var collen = (input.length - 1) / 3;
        var fact = [17, 1, 0.062272][collen - 1];
        return [
          Math.round(parseInt(input.substr(1, collen), 16) * fact),
          Math.round(parseInt(input.substr(1 + collen, collen), 16) * fact),
          Math.round(parseInt(input.substr(1 + 2 * collen, collen), 16) * fact),
        ];
      } else
        return input
          .split("(")[1]
          .split(")")[0]
          .split(",")
          .map((x) => +x);
    },
    rgbToXyz(rgb) {
      let r = rgb[0] / 255;
      let g = rgb[1] / 255;
      let b = rgb[2] / 255;

      r = r > 0.04045 ? Math.pow((r + 0.055) / 1.055, 2.4) : r / 12.92;
      g = g > 0.04045 ? Math.pow((g + 0.055) / 1.055, 2.4) : g / 12.92;
      b = b > 0.04045 ? Math.pow((b + 0.055) / 1.055, 2.4) : b / 12.92;

      r *= 100;
      g *= 100;
      b *= 100;

      const x = r * 0.4124 + g * 0.3576 + b * 0.1805;
      const y = r * 0.2126 + g * 0.7152 + b * 0.0722;
      const z = r * 0.0193 + g * 0.1192 + b * 0.9505;

      return [x, y, z];
    },
    xyzToLab(xyz) {
      const [x, y, z] = xyz;

      const xn = 95.047;
      const yn = 100.0;
      const zn = 108.883;

      const fx = x / xn;
      const fy = y / yn;
      const fz = z / zn;

      const epsilon = 0.008856;
      const kappa = 903.3;

      const f = (t) => (t > epsilon ? Math.pow(t, 1 / 3) : (kappa * t + 16) / 116);

      const L = 116 * f(fy) - 16;
      const a = 500 * (f(fx) - f(fy));
      const b = 200 * (f(fy) - f(fz));

      return [L, a, b];
    },
  },
  watch: {
    pickedColor(newVal) {
      if (this.selectedButton === "button1") {
        this.color1 = newVal;
        this.updateColors();
      }
      if (this.selectedButton === "button2") {
        this.color2 = newVal;
        this.updateColors();
      }
    },
  },
});
</script>

<style scoped lang="scss">
.color-info-panel {
  position: absolute;
  top: 20px;
  right: 20px;
  background: #fff;
  border: 1px solid #ddd;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 2px
  10px rgba(0, 0, 0, 0.1);
  width: 300px;
}

.panel-header {
  display: flex;
  justify-content: flex-end;
}

.close-button {
  background: transparent;
  border: none;
  font-size: 20px;
  cursor: pointer;
}

.color-buttons {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

.color-button {
  -webkit-appearance: none;
  cursor: pointer;
  border: 1px solid black;
  border-radius: 50%;
  width: 23px;
  height: 23px;
  &:checked {
    outline: 2px solid white;
    border: 2px solid black;
  }
}

.color-details {
  .color-label {
    font-weight: bold;
    margin: 10px 0 5px;
  }
  .color-values {
    display: flex;
    justify-content: space-between;
    p {
      margin: 0;
    }
  }
}

.contrast-ratio {
  margin-top: 20px;
  font-weight: bold;
  text-align: center;
}
</style>
