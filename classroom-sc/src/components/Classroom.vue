<template>
  <div class="header">
    <div v-show="teacherMode" style="margin-right: 10px">
      <span>Evaluate </span>
      <input v-model="evaluateCount" type="number" min="1" />
    </div>
    <div v-show="teacherMode" style="margin-right: 10px">
      <span>TP </span>
      <input v-model="selectedTp" type="number" max="6" min="0" />
    </div>
    <input
      ref="inputPassword"
      v-show="!teacherMode"
      v-model="password"
      type="password"
    />
    <button
      v-show="!teacherMode"
      @click="checkPassword(password)"
      class="btn primary"
    >
      Run
    </button>

    <button
      v-show="teacherMode"
      @click="allTp(selectedTp)"
      class="btn primary plus"
    >
      {{ selectedTp }} Tp
    </button>
    <button v-show="teacherMode" @click="allExp(1)" class="btn primary plus">
      +1 Exp
    </button>
    <button v-show="teacherMode" @click="allExp(-1)" class="btn primary minus">
      -1 Exp
    </button>
    <button @click="isShowNumber = !isShowNumber" class="btn primary">
      {{ isShowNumber ? "Hide" : "Show" }} no.
    </button>
    <button
      v-show="teacherMode"
      @click="updateResult(datas)"
      class="btn primary"
    >
      <span v-if="isLoading">Loading...</span>
      <span v-else>Update</span>
    </button>

    <button @click="sortBy()" class="btn primary">
      Sort by: {{ sortType }}
    </button>
  </div>

  <!-- seat -->
  <div class="seats">
    <Seat
      v-for="data in datas"
      :key="data.seat"
      :data="data"
      :teacher-mode="teacherMode"
      :isShowNumber="isShowNumber"
      @tp="openModalTp"
      @exp="singleExp"
      @absent="absent"
      @skill="openModalSkill"
    />
  </div>

  <!-- ModalTp -->
  <ModalTp
    :open="isOpenModalTp"
    :evaluateCount="evaluateCount"
    :selectedStudent="selectedStudent"
    @close="isOpenModalTp = false"
    @selectTp="selectTp"
  />

  <!-- ModalRadar -->
  <ModalRadar
    :open="isOpenModalRadar"
    :data="selectedSkills"
    :labels="labels"
    @skills="skills"
    @close="isOpenModalRadar = false"
  />

  <!-- keyboard -->
  <div v-show="isOpenKeyboard && teacherMode" class="keyboard">
    <p class="saved">{{ keyboard }}</p>
    <p v-show="selectedStudent" class="selectedStudentName">
      {{ selectedStudent.name }}
    </p>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import Seat from "./Seat.vue";
import ModalTp from "./ModalTp.vue";
import ModalRadar from "./ModalRadar.vue";

function aveTpFn(val: string) {
  return Math.round(
    JSON.parse(val).reduce((a: number, b: number) => a + b, 0) /
      JSON.parse(val).length
  );
}

export default defineComponent({
  name: "Classroom",
  props: ["sheetUrl", "scriptUrl"],
  components: {
    Seat,
    ModalTp,
    ModalRadar,
  },
  data() {
    return {
      password: "",
      teacherMode: false,
      tpExps: [0, 0, 0, 2, 4, 6],
      datas: [] as any,
      isOpenModalTp: false,
      selectedStudent: Object as any,
      isOpenModalRadar: false,
      selectedSkills: [0, 0, 0, 0, 0, 0],
      isShowResult: false,
      labels: ["未来目标", "理解力", "逻辑力", "自控力", "企业能力", "反思"],
      selectedTp: 4,
      evaluateCount: 1,
      isLoading: false,
      isOpenKeyboard: false,
      keyboard: "",
      keyboardOperator: "",
      isShowNumber: false,
      sortType: "",
    };
  },
  setup(props) {
    const inputPassword = ref();
    let datas = ref([]);
    const url = `https://docs.google.com/spreadsheets/d/${props.sheetUrl}/gviz/tq?tqx=out:json`;

    function jsonToObject(rep: any) {
      const data = JSON.parse(rep.substring(47).slice(0, -2));
      const cols = data.table.cols;
      const results = data.table.rows.map((row: any) => {
        const obj = {} as any;
        row.c.forEach((data: any, index: number) => {
          obj[cols[index].label] = data.v;
        });
        obj.tp = aveTpFn(obj.tpStr);
        return obj;
      });
      results.sort((a: any, b: any) => a.seat < b.seat);
      return results;
    }

    fetch(url + "&sheet=Sheet1")
      .then((res) => res.text())
      .then((rep) => (datas.value = jsonToObject(rep)));

    return { inputPassword, datas, scriptUrl: props.scriptUrl };
  },
  mounted() {
    this.inputPassword.focus();
  },
  methods: {
    updateResult(datas: any) {
      const sortByDefault = [...datas].sort((a: any, b: any) => a.no - b.no);
      const results = sortByDefault.map((data: any) => {
        const obj: any = [
          data.exp,
          data.s1,
          data.s2,
          data.s3,
          data.s4,
          data.s5,
          data.s6,
          data.tpStr,
          data.absent,
        ];
        return obj;
      });
      const url = `https://script.google.com/macros/s/${this.scriptUrl}/exec`;

      console.log(datas);

      this.isLoading = true;

      fetch(url, {
        method: "POST",
        mode: "no-cors",
        cache: "no-cache",
        headers: {
          "Content-Type": "application/json",
        },
        redirect: "follow",
        body: JSON.stringify({ data: results }),
      }).then(() => (this.isLoading = false));
    },
    allTp(val: number) {
      this.datas.forEach((d: any) => {
        let tempExp = 0;
        if (d.tp !== 0) {
          tempExp = this.tpExps[d.tp - 1];
        }
        // remove all tp
        if (val === 0) {
          d.exp -= tempExp;
          d.tp = 0;
          return;
        }
        d.exp += this.tpExps[val - 1] - tempExp;
        d.tp = val;
      });
    },
    allExp(val: number) {
      this.datas.forEach((d: any) => {
        if (!d.absent) {
          d.exp += val;
        }
      });
    },
    openModalTp(val: number) {
      this.selectedStudent = this.datas.find((d: any) => d.seat === val);
      this.isOpenModalTp = true;
    },
    selectTp(val: string) {
      let tempExp = 0;
      if (this.selectedStudent.tp !== 0) {
        tempExp = this.tpExps[this.selectedStudent.tp - 1];
      }
      // remove all tp
      const aveTp = aveTpFn(val);

      if (aveTp === 0) {
        this.selectedStudent.exp -= tempExp;
        this.selectedStudent.tp = 0;
        return;
      }
      this.selectedStudent.exp += this.tpExps[aveTp - 1] - tempExp;
      this.selectedStudent.tp = aveTp;
      this.selectedStudent.tpStr = val;
    },
    singleExp(value: any) {
      this.datas.find((d: any) => d.seat === value.seat).exp += value.val;
    },
    absent(seat: any) {
      const student = this.datas.find((d: any) => d.seat === seat);
      student.absent = !student.absent;
    },
    openModalSkill(val: number) {
      this.selectedStudent = this.datas.find((d: any) => d.seat === val);
      this.selectedSkills = [
        this.selectedStudent.s1,
        this.selectedStudent.s2,
        this.selectedStudent.s3,
        this.selectedStudent.s4,
        this.selectedStudent.s5,
        this.selectedStudent.s6,
      ];
      this.isOpenModalRadar = true;
    },
    skills(skills: any) {
      this.datas.forEach((d: any) => {
        if (d.seat === this.selectedStudent.seat) {
          d.s1 = skills[0];
          d.s2 = skills[1];
          d.s3 = skills[2];
          d.s4 = skills[3];
          d.s5 = skills[4];
          d.s6 = skills[5];
        }
      });
    },
    checkPassword(value: any) {
      if (value === "1947") {
        this.teacherMode = true;
      }
    },
    onKeyboard(e: KeyboardEvent) {
      if (!this.teacherMode) {
        return;
      }
      this.isOpenKeyboard = true;
      this.keyboard += e.key;
      // this.keyboard.length === 2
      if (this.keyboard.length === 2) {
        this.selectedStudent = this.datas.find(
          (d: any) => d.no === parseInt(this.keyboard)
        );

        if (parseInt(this.keyboard) < 10) {
          this.keyboard = "0" + parseInt(this.keyboard);
        }
      }
      // this.keyboard.length === 3
      if (this.keyboard.length === 3) {
        const operation = this.keyboard.substring(2, 3);
        if (operation === "+" || operation === "-") {
          this.keyboardOperator = operation;
        } else if (operation === "*") {
          if (this.evaluateCount === 1) {
            this.keyboardOperator = operation;
          } else {
            alert("Not allow to use! (evaluateCount must 1)");
            this.isOpenKeyboard = false;
            this.keyboard = "";
          }
        }
      }
      // this.keyboard.length === 4
      if (this.keyboard.length === 4) {
        const value = parseInt(this.keyboard.substring(3, 4));
        if (this.keyboardOperator === "+") {
          this.selectedStudent.exp += value;
        } else if (this.keyboardOperator === "-") {
          this.selectedStudent.exp -= value;
        } else if (this.keyboardOperator === "*") {
          this.selectTp(`[${value}]`);
        }
        this.isOpenKeyboard = false;
        this.keyboard = "";
      }

      if (e.key === ".") {
        this.isOpenKeyboard = false;
        this.keyboard = "";
      }
    },
    sortBy() {
      if (this.sortType === "") {
        this.sortType = "High to Low";
        this.datas.sort((a: any, b: any) => a.exp - b.exp).reverse();
      } else if (this.sortType === "High to Low") {
        this.sortType = "Low to High";
        this.datas.sort((a: any, b: any) => a.exp - b.exp);
      } else if (this.sortType === "Low to High") {
        this.sortType = "";
        this.datas.sort((a: any, b: any) => a.no - b.no);
      }
    },
  },
  created() {
    window.addEventListener("keypress", this.onKeyboard);
  },
});
</script>

<style lang="scss" scoped>
.header {
  display: flex;
  button ~ button {
    margin-left: 5px;
  }
}
.seats {
  margin-top: 10px;
  display: grid;
  grid-gap: 10px;
  grid-template-columns: repeat(5, 1fr);
  grid-auto-flow: row dense;
  @media (max-width: 800px) {
    grid-template-columns: repeat(2, 1fr);
  }
}

.keyboard {
  position: fixed;
  z-index: 2;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(255, 255, 255, 0.9);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  color: black;
  font-size: 60px;
  padding-bottom: 10%;
  .saved {
    font-size: 100px;
  }
}
</style>
