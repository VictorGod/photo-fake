<template>
  <div :class="{ 'status-bar': true, 'pipette-mode': state === 'pipette' && pickedColor }">
    <div class="left-side">
      <span v-if="state"> St: {{ state }}</span>
      <span v-if="imageWidth && imageHeight"> | W: {{ imageWidth }} | H: {{ imageHeight }}</span>
      <span v-if="state === 'pipette' && pickedColor"> | C: {{ pickedColor }}</span>
      <div v-if="state === 'pipette' && pickedColor" class="pipette-color" :style="{ background: pickedColor }"></div>
      <span v-if="state === 'pipette' && pickedColor && xMouse && yMouse"> | Coords: {{ xMouse }}:{{ yMouse }}</span>
    </div>

    <div class="right-side">
      <label for="scale">Scale</label>
      <input
        type="range"
        class="scale"
        id="scale"
        name="scale"
        min="12"
        max="300"
        step="1"
        :value="scale"
        @input="updateScale"
      />
      <span>Scale: {{ scale }}</span>
    </div>
  </div>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "StatusBar",
  props: {
    state: String,
    imageHeight: Number,
    imageWidth: Number,
    pickedColor: String,
    xMouse: Number,
    yMouse: Number,
    scale: Number,
  },
  methods: {
    updateScale(event) {
      const newScale = +event.target.value;
      this.$emit("updateScale", newScale);
    },
  },
});
</script>

<style scoped lang="scss">
.status-bar {
  position: absolute;
  border: 1px solid #ddd;
  border-radius: 10px;
  padding:15px;
  bottom: 0;
  justify-content: space-between;
  align-items: center;
  padding: 0 10px;
}

.left-side {
  display: flex;
  align-items: center;
}

.right-side {
  display: flex;
  align-items: center;
}

.pipette-color {
  margin-top: 2.5px;
  width: 15px;
  height: 15px;
  border-radius: 50%;
}

.scale {
  margin-left: 10px;
}
</style>