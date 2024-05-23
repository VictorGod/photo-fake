<template>
  <div class="menu">
    <button
      class="menu-button"
      @click="onButtonClick"
      value="input"
      title="Выберите изображение с помощью диалогового окна выбора файла"
    >
      <label for="fileInput" class="button-label">
        <img class="button-image" src="../assets/file.svg" alt="Выбрать изображение" />
      </label>
      <input
        id="fileInput"
        class="button-input"
        type="file"
        @change="onImageSelected"
      />
    </button>
    <div class="url">
      <input
        class="url-input"
        v-model="imageUrl"
        placeholder="URL"
        @keydown.enter="loadImageFromUrl"
      />
    </div>
    <button
      class="menu-button"
      @click="onButtonClick"
      :disabled="!selectedImage"
      value="pipette"
      title="Используйте пипетку для выбора цвета из изображения"
    >
      <label
        class="button-label"
        :class="{
          'button-label-tapped': state === 'pipette',
          disabled: !selectedImage,
        }"
      >
        <img class="button-image" src="../assets/pipette.svg" alt="Пипетка" />
      </label>
    </button>
    <button
      class="menu-button"
      @click="onButtonClick"
      :disabled="!selectedImage"
      value="modal"
      title="Отобразите модальное окно для ввода дополнительной информации или выполнения определенного действия"
    >
      <label
        class="button-label"
        :class="{
          'button-label-tapped': state === 'modal',
          disabled: !selectedImage,
        }"
      >
        <img class="button-image" src="../assets/modal.svg" alt="Модальное окно" />
      </label>
    </button>
    <button
      class="menu-button"
      @click="onButtonClick"
      :disabled="!selectedImage"
      value="hand"
      title="Переместите изображение по холсту"
    >
      <label
        class="button-label"
        :class="{
          'button-label-tapped': state === 'hand',
          disabled: !selectedImage,
        }"
      >
        <img class="button-image" src="../assets/hand.svg" alt="Переместить изображение" />
      </label>
    </button>
    <button
      class="menu-button"
      @click="onButtonClick"
      :disabled="!selectedImage"
      value="save"
      title="Сохраните текущее изображение после внесения необходимых изменений"
    >
      <label class="button-label" :class="{ disabled: !selectedImage }">
        <img
          class="button-image"
          src="../assets/downloads.svg"
          alt="Сохранить изображение"
        />
      </label>
    </button>
    <button
      class="menu-button"
      @click="onButtonClick"
      :disabled="!selectedImage"
      value="correct"
      title="Редактировать текущее изображение"
    >
      <label class="button-label" :class="{ disabled: !selectedImage }">
      <img
          class="button-image"
          src="../assets/correction.svg"
          alt="Редактировать изображение"
        />
      </label>
    </button>
    <button
      class="menu-button"
      @click="onButtonClick"
      :disabled="!selectedImage"
      value="filter"
      title="Применить фильтры к текущему изображению"
    >
      <label class="button-label" :class="{ disabled: !selectedImage }">
        <img
          class="button-image"
          src="../assets/filtration.svg"
          alt="Фильтровать изображение"
        />
      </label>
    </button>
  </div>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "MainMenu",
  props: {
    state: String,
  },
  data() {
    return {
      selectedImage: "",
      imageUrl: "",
    };
  },
  methods: {
    onImageSelected(event) {
      const target = event.target;
      const selectedFile = target.files?.[0];

      if (selectedFile) {
        const imageUrl = URL.createObjectURL(selectedFile);
        this.selectedImage = imageUrl;
        this.$emit("onImageSelected", imageUrl);

        target.value = "";
      }
    },
    loadImageFromUrl() {
      if (this.imageUrl) {
        this.$emit("onImageSelected", this.imageUrl);
        this.selectedImage = this.imageUrl;
        this.imageUrl = "";
      }
    },
    onButtonClick(event) {
      const target = event.target;
      const value = target.parentElement?.value
        ? target.parentElement.value
        : target.parentElement?.parentElement.value;
      this.$emit("changeState", value);
    },
  },
  beforeUnmount() {
    if (this.selectedImage) {
      URL.revokeObjectURL(this.selectedImage);
    }
  },
  watch: {
    state(newValue) {
      if (newValue === "pipette") {
        this.stateCursor = "pipette";
      } else this.stateCursor = null;
    },
  },
});
</script>

<style scoped lang="scss">
.menu {
  width: 50px;
  height: calc(100% - 10px);
  padding: 5px;
  display: flex;
  flex-direction: column;
  gap: 5px;
  align-items: center;
}
.menu-button {
  position: relative;
  display: inline-block;
  cursor: pointer;
  border: none;
  padding: 5px;
  margin: 0;
  background-color: #ffffff;
}
.disabled {
  opacity: 0.1;
}
.menu-button:hover {
  background-color: #999999; /* Затемненный фон */
  border-radius: 30%;
}
.button {
  &-input {
    width: 100%;
    display: none;
  }

  &-label {
    cursor: pointer;
    display: block;
    transition: 0.3s;
    &:hover {
      background: #a0a0a0;
      border-color: #a0a0a0;
    }
    &-tapped {
      background: #a0a0a0;
      border-color: #a0a0a0;
    }
  }

  &-image {
    height: 25px;
  }
}

.url {
  &-input {
    width: 48px;
  }
}
.disabled {
  opacity: 0.3;
}
</style>
