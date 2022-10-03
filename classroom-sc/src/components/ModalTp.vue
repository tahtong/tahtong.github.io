<template>
  <transition name="open">
    <div v-show="open" class="modalTp">
      <div class="overlay"></div>
      <div class="content">
        <h3>{{ title }}</h3>
        <div class="singleTp" v-for="(tp, index) in tps" :key="tp">
          <span>{{ index + 1 }}.</span>
          <button @click="selectTp(index, 0)" class="btn large">0</button>
          <button
            v-for="n in 6"
            :key="n"
            @click="selectTp(index, n)"
            :class="{ active: tps[index] === n }"
            class="btn large"
          >
            <img
              v-for="m in n"
              :key="m"
              src="../assets/images/star.png"
              width="12"
            />
          </button>
        </div>
        <div class="bottom">
          <button @click="updateTp" class="btn">Update</button>
          <button @click="close" class="btn">Close</button>
        </div>
      </div>
    </div>
  </transition>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";

export default defineComponent({
  name: "ModalTp",
  props: ["open", "evaluateCount", "selectedStudent","selectedGroup"],
  emits: ["close", "selectTp"],
  setup(props) {
    const tps = ref<number[]>([]);
    for (let i = 0; i < props.evaluateCount; i++) {
      tps.value.push(0);
    }
    return { tps };
  },
  data() {
    return {
      tps: [] as number[],
      title: ""
    };
  },
  methods: {
    close() {
      this.$emit("close");
    },
    selectTp(index: number, val: number) {
      this.tps[index] = val;
    },
    updateTp() {
      this.$emit("selectTp", JSON.stringify(this.tps));
      this.$emit("close");
    },
  },
  watch: {
    open(newVal) {
      if (newVal === true) {
        // single student
        if (
          this.selectedStudent &&
          Object.keys(this.selectedStudent).length > 0
        ) {
          this.title = this.selectedStudent.name;
          const parse = JSON.parse(this.selectedStudent.tpStr);
          const totalTp = parse.reduce(
            (v: number, t: number) => (t = t + v),
            0
          );

          if (totalTp === 0) {
            const arr = [];
            for (let i = 0; i < this.evaluateCount; i++) {
              arr.push(0);
            }
            this.tps = arr;
          } else {
            this.tps = parse;
          }
        }
        
        console.log(this.selectedGroup)
        // group students
        if (
          this.selectedGroup &&
          Object.keys(this.selectedGroup).length > 0
        ) {
          this.title = `${this.selectedGroup.name}组`;
        }
      }
    },
  },
});
</script>

<style lang="scss" scoped>
.modalTp {
  position: fixed;
  z-index: 2;
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
    display: flex;
    flex-direction: column;

    h3{
      margin-bottom:10px;
    }
    .singleTp {
      display: flex;
      align-items: center;
      & ~ .singleTp {
        margin-top: 8px;
      }
    }
    .large {
      height: 56px;
      width: 48px;
      margin-left: 10px;
    }
    .active {
      border: 2px solid white;
    }
  }
  .bottom {
    margin-top: 20px;
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
