<template>
  <transition name="open">
    <div v-if="open" class="modal">
      <div class="overlay"></div>
      <div class="content">
        <RadarChart :data="data" :labels="labels" />
        <div class="buttons">
          <button
            v-for="(label, index) in labels"
            :key="label"
            @click="addSkill(index)"
            class="btn"
          >
            {{ label }}
          </button>
          <button @click="close" class="btn large">x</button>
        </div>
      </div>
    </div>
  </transition>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import RadarChart from "./RadarChart.vue";

export default defineComponent({
  name: "ModalChart",
  props: ["open", "skills", "data", "labels"],
  emits: ["skills", "close"],
  components: {
    RadarChart,
  },
  methods: {
    addSkill(index: number) {
      this.data[index]++;
      this.$emit("skills", this.data);
    },
    close() {
      this.$emit("close");
    },
  },
});
</script>

<style lang="scss" scoped>
.modal {
  position: fixed;
  z-index:2;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  color: black;
  .overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: white;
  }
  .content {
    display: flex;
    align-items: center;
    position: relative;
    padding: 20px;
    .buttons {
      display: flex;
      flex-direction: column;
      margin-top: -30px;
      margin-left: 20px;
      button {
        color: black;
        height: 40px;
        width: 80px;
        &:hover {
          opacity: 0.5;
        }
        &:nth-child(1) {
          background-color: #5b1647;
          color: white;
        }
        &:nth-child(2) {
          background-color: #93073e;
          color: white;
        }
        &:nth-child(3) {
          background-color: #c90035;
          color: white;
        }
        &:nth-child(4) {
          background-color: #ff5627;
        }
        &:nth-child(5) {
          background-color: #ffc400;
        }
        &:nth-child(6) {
          background-color: #205765;
          color: white;
        }
      }
    }
  }
}

.open-enter-active,
.open-leave-active {
  transition: 0.6s ease;
}

.open-enter-from,
.open-leave-to {
  opacity: 0;
}
</style>
