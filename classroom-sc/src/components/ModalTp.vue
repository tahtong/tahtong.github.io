<template>
  <transition name="open">
    <div v-show="open" class="modalTp">
      <div class="overlay"></div>
      <div class="content">
        <span>TP: </span>
        <button @click="selectTp(0)" class="btn large">0</button>
        <button
          v-for="n in 6"
          :key="n"
          @click="selectTp(n)"
          class="btn large"
        >
          <img
            v-for="m in n"
            :key="m"
            src="../assets/images/star.png"
            width="12"
          />
        </button>
        <button @click="close" class="btn large">x</button>
      </div>
    </div>
  </transition>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  name: "ModalTp",
  props: ["open"],
  emits: ["close", "selectTp"],
  data() {
    return {
      selectedStudent: null,
    };
  },
  methods: {
    close() {
      this.$emit("close");
    },
    selectTp(val: number) {
      this.$emit("selectTp", val);
      this.$emit("close");
    },
  },
});
</script>

<style lang="scss" scoped>
.modalTp {
  position: fixed;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  padding-bottom: 10%;
  .overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(white, 0.8);
  }
  .content {
    border-radius: 5px;
    display: flex;
    align-items: center;
    position: relative;
    padding: 20px;
    background-color: black;
    .large {
      height: 56px;
      width: 48px;
      margin-left: 10px;
    }
  }
}

.open-enter-active,
.open-leave-active {
  transition: 0.6s ease;
  .overlay {
    transition: opacity 0.5s ease;
  }
  .content {
    transition: all 0.3s 0.3s ease;
  }
}

.open-enter-from,
.open-leave-to {
  .overlay {
    opacity: 0;
  }
  .content {
    transform: translateY(-100%);
    opacity: 0;
  }
}
</style>
