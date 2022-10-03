<template>
  <div :class="{ absent: data.absent, group: isShowGrouping }" class="seat">
    <div @click="skill(data.seat)" class="table">
      <div
        :class="'avatart-' + getLevelAndExp(data.exp).level"
        class="avatar"
      ></div>
      <div class="content">
        <div class="name">{{ data.name }}</div>
        <div class="level">LV {{ getLevelAndExp(data.exp).level }}</div>
        <div class="exp">Exp: {{ data.exp }}</div>
        <pre v-show="isShowTp">{{ data.tpStr }}</pre>
      </div>
      <div class="tp">
        <img v-for="n in data.tp" :key="n" src="../assets/images/star.png" />
      </div>
      <div class="skills">
        <div v-if="data.s1 > 0" class="skill s1">{{ data.s1 }}</div>
        <div v-if="data.s2 > 0" class="skill s2">{{ data.s2 }}</div>
        <div v-if="data.s3 > 0" class="skill s3">{{ data.s3 }}</div>
        <div v-if="data.s4 > 0" class="skill s4">{{ data.s4 }}</div>
        <div v-if="data.s5 > 0" class="skill s5">{{ data.s5 }}</div>
        <div v-if="data.s6 > 0" class="skill s6">{{ data.s6 }}</div>
      </div>
      <div v-show="isShowNumber" class="no">{{ data.no }}</div>
    </div>
    <div class="chair">
      <div class="expBar">
        <div
          class="currentExp"
          :style="{ width: getExpPercent(data.exp) }"
        ></div>
        <div class="number">{{ getExpInfo(data.exp) }}</div>
      </div>
      <div v-show="teacherMode" class="buttonArea">
        <button @click="tp(data.seat)" class="btn plus">
          <img src="../assets/images/startOutline.png" width="10" />
        </button>
        <button @click="exp(data.seat, 1)" class="btn plus">+</button>
        <button @click="exp(data.seat, -1)" class="btn minus">-</button>
        <button @click="absent(data.seat)" class="btn absent minus">x</button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  name: "Seat",
  props: ["data", "teacherMode", "isShowNumber", "isShowTp", "isShowGrouping"],
  emits: ["tp", "exp", "absent", "skill"],
  data() {
    return {
      levelExps: [10, 15, 20, 25, 30, 35],
    };
  },
  methods: {
    getLevelAndExp(studentExp: number) {
      let studentLessExp = studentExp;
      let level = 0;
      for (let i = 0; i < this.levelExps.length; i++) {
        const exp = this.levelExps[i];
        if (studentLessExp - exp >= 0) {
          studentLessExp = studentLessExp - exp;
          continue;
        } else {
          level = i + 1;
          break;
        }
      }
      return { level, studentLessExp };
    },
    getExpInfo(studentExp: number) {
      const levelAndExp = this.getLevelAndExp(studentExp);
      return `${levelAndExp.studentLessExp}/${
        this.levelExps[levelAndExp.level - 1]
      }`;
    },
    getExpPercent(studentExp: number) {
      const levelAndExp = this.getLevelAndExp(studentExp);
      return `${
        (1 -
          levelAndExp.studentLessExp / this.levelExps[levelAndExp.level - 1]) *
        100
      }%`;
    },
    tp(seat: number) {
      this.$emit("tp", seat);
    },
    exp(seat: number, val: number) {
      this.$emit("exp", { seat, val });
    },
    absent(seat: number) {
      this.$emit("absent", seat);
    },
    skill(seat: number) {
      this.$emit("skill", seat);
    },
  },
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.seat {
  background: rgba(0, 0, 0, 0.5);
  border-radius: 5px;
  &.absent {
    opacity: 0.2;
    button {
      &:nth-child(1),
      &:nth-child(2),
      &:nth-child(3) {
        pointer-events: none;
      }
    }
  }
  &.group {
    font-size: 10px;
    .content {
      display: none;
    }
    .chair {
      button {
        display: none;
      }
    }
  }
}
.table {
  display: flex;
  margin: 5px 0;
  cursor: pointer;
  position: relative;
  .avatar {
    background-image: url(../assets/images/heros.png);
    height: 50px;
    width: 50px;
    flex: 0 0 50px;
    background-size: 300px;
    position: relative;
    z-index: 1;
    @for $i from 23 through 0 {
      &.avatart-#{$i + 1} {
        background-position: #{floor($i / 4) * -100%} #{($i % 4) * -100%};
      }
    }
    transition: 0.3s;
    transition-delay: 0.5s;
    &:hover {
      animation: shake 1s;
      animation-iteration-count: infinite;
    }
  }
  .content {
    flex: 0 0 50px;
    margin-left: 5px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    .level {
      font-size: 14px;
      margin-top: 4px;
      color: #eee;
    }
    .exp {
      font-size: 12px;
      margin-top: -4px;
      color: #eee;
    }
  }
  .tp {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    justify-content: center;
    img {
      width: 14px;
      height: 14px;
      &:nth-last-child(1),
      &:nth-last-child(2),
      &:nth-last-child(3) {
        width: 10px;
        height: 10px;
      }
    }
  }
  .skills {
    display: flex;
    flex-direction: column;
    position: absolute;
    width: 10px;
    height: 100%;
    .skill {
      width: 10px;
      height: 10px;
      font-size: 7px;
      display: grid;
      place-items: center;
      &.s1 {
        background-color: #5b1647;
      }
      &.s2 {
        background-color: #93073e;
      }
      &.s3 {
        background-color: #c90035;
      }
      &.s4 {
        background-color: #ff5627;
      }
      &.s5 {
        background-color: #ffc400;
        color: black;
      }
      &.s6 {
        background-color: #205765;
      }
    }
  }

  .no {
    position: absolute;
    right: 2px;
    font-family: monospace;
    top: 10px;
    width: 30px;
    height: 30px;
    border: 1px solid white;
    display: grid;
    place-content: center;
    background-color: #111111;
  }
}
.chair {
  .expBar {
    border: 2px solid #abbba4;
    border-radius: 5px;
    position: relative;
    height: 16px;
    overflow: hidden;
    background-image: linear-gradient(to right, yellow, green);
    .currentExp {
      position: absolute;
      right: 0;
      background-color: black;
      height: 18px;
      transition: 1s;
      transition-delay: 1s;
    }
    .number {
      font-size: 12px;
      position: absolute;
      width: 100%;
      top: 0px;
      text-align: center;
      text-shadow: 1px 1px black;
    }
  }
  button {
    width: 25%;
  }
}

@keyframes shake {
  0% {
    transform: translate(1px, 1px) rotate(0deg);
  }
  10% {
    transform: translate(-1px, -2px) rotate(-1deg);
  }
  20% {
    transform: translate(-3px, 0px) rotate(1deg);
  }
  30% {
    transform: translate(3px, 2px) rotate(0deg);
  }
  40% {
    transform: translate(1px, -1px) rotate(1deg);
  }
  50% {
    transform: translate(-1px, 2px) rotate(-1deg);
  }
  60% {
    transform: translate(-3px, 1px) rotate(0deg);
  }
  70% {
    transform: translate(3px, 1px) rotate(-1deg);
  }
  80% {
    transform: translate(-1px, -1px) rotate(1deg);
  }
  90% {
    transform: translate(1px, 2px) rotate(0deg);
  }
  100% {
    transform: translate(1px, -2px) rotate(-1deg);
  }
}
</style>
