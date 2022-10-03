<template>
  <div class="header">
    <!-- teacherMode = false -->
    <section v-show="!teacherMode">
      <input ref="inputPassword" v-model="password" type="password" />
      <button @click="checkPassword(password)" class="btn primary">Run</button>
    </section>
    <!-- teacherMode = true -->
    <section v-show="teacherMode">
      <div v-show="isDefaultEvalute">
        <span>Evaluate </span>
        <input v-model="evaluateCount" type="number" min="1" />
      </div>
      <div>
        <span>TP </span>
        <input v-model="selectedTp" type="number" max="6" min="0" />
      </div>
      <button @click="allTp(selectedTp)" class="btn primary plus">
        {{ selectedTp }} Tp
      </button>
      <button @click="allExp(1)" class="btn primary plus">+1 Exp</button>
      <button @click="allExp(-1)" class="btn primary minus">-1 Exp</button>
      <button @click="sortBy()" class="btn primary">
        Sort by: {{ sortType }}
      </button>
      <button @click="updateResult(datas)" class="btn primary">
        <span v-if="isLoading">Loading...</span>
        <span v-else>Update</span>
      </button>
      <button @click="teacherMode = false" class="btn primary">x</button>
      <button @click="showOption = !showOption" class="btn primary">
        {{ showOption ? "Hide" : "Show" }} more
      </button>
    </section>
    <!-- showOption = true -->
    <section v-show="showOption">
      <section v-show="!isShowGrouping">
        <div>Group <input v-model="groupTotal" type="number" /></div>
        <button @click="grouping" class="btn primary">
          {{ groupTotal }} Group
        </button>
      </section>
      <button
        v-show="isShowGrouping"
        @click="isShowGrouping = false"
        class="btn primary"
      >
        x Group
      </button>
      <button @click="isShowNumber = !isShowNumber" class="btn primary">
        {{ isShowNumber ? "Hide" : "Show" }} no.
      </button>
      <button @click="isShowTp = !isShowTp" class="btn primary">
        {{ isShowTp ? "Hide" : "Show" }} TP
      </button>
      <button @click="isShowTest = !isShowTest" class="btn primary">
        {{ isShowTest ? "Hide" : "Show" }} Test
      </button>
      <section v-show="isShowTest">
        <button @click="updateTextSize('+')" class="btn primary">A+</button>
        <button @click="updateTextSize('-')" class="btn primary">A-</button>
      </section>
    </section>
  </div>

  <textarea
    v-model="test"
    v-show="isShowTest"
    :style="{ fontSize: textSize + 'px' }"
    rows="10"
    class="testBox"
  >
  </textarea>

  <!-- seat -->
  <div
    :class="{ group: isShowGrouping }"
    :style="gridStyle"
    class="seats"
  >
    <Seat
      v-for="data in datas"
      :key="data.seat"
      :data="data"
      :teacher-mode="teacherMode"
      :isShowNumber="isShowNumber"
      :isShowTp="isShowTp"
      :isShowGrouping="isShowGrouping"
      @tp="openModalTpForStudent"
      @exp="singleExp"
      @absent="absent"
      @skill="openModalSkill"
    />
  </div>
  <div v-if="isShowGrouping" class="grouping">
    <div
      @click="openModalTpForGroup(index)"
      v-for="(group, index) in groups"
      :key="index"
      class="group"
    >
      <div class="top">
        <p class="title">{{ indexToAlphabet(index) }}</p>
        <div class="name">
          <span v-for="student in group.students" :key="student">{{
            student.name
          }}</span>
        </div>
      </div>
      <pre>{{ group.tpStr }}</pre>
    </div>
  </div>

  <!-- ModalTp -->
  <ModalTp
    :open="isOpenModalTp"
    :evaluateCount="evaluateCount"
    :selectedStudent="selectedStudent"
    :selectedGroup="selectedGroup"
    @close="
      isOpenModalTp = false;
      (selectedStudent = {}), (selectedGroup = {});
    "
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
      isShowTp: false,
      isShowGrouping: false,
      groupTotal: 5,
      showOption: false,
      groups: [] as any,
      selectedGroup: {} as any,
      isShowTest: false,
      textSize: 16,
    };
  },
  computed: {
    gridStyle() {
      return {
        gridTemplateColumns: `repeat(${Math.ceil(this.datas.length / this.groupTotal)}, 1fr)`
      }  
    },
  },
  setup(props) {
    const inputPassword = ref();
    let datas = ref([]);
    let test = ref("");
    const url = `https://docs.google.com/spreadsheets/d/${props.sheetUrl}/gviz/tq?tqx=out:json`;
    let isDefaultEvalute = ref(false);

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
      .then((rep) => {
        datas.value = jsonToObject(rep);
        isDefaultEvalute.value = datas.value.every(
          (d: any) => d.tpStr === "[0]"
        );
      });

    fetch(url + "&sheet=Test")
      .then((res) => res.text())
      .then((rep) => {
        const data = JSON.parse(rep.substring(47).slice(0, -2));
        test.value = data.table.rows[1].c[0].v;
      });

    return {
      inputPassword,
      datas,
      test,
      scriptUrl: props.scriptUrl,
      isDefaultEvalute,
    };
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
    openModalTpForStudent(val: number) {
      this.selectedStudent = this.datas.find((d: any) => d.seat === val);
      this.isOpenModalTp = true;
    },
    selectTp(obj: any) {
      const tp = obj.val;
      const type = obj.type;
      // type === student
      if (type === "student") {
        let tempExp = 0;
        if (this.selectedStudent.tp !== 0) {
          tempExp = this.tpExps[this.selectedStudent.tp - 1];
        }
        // remove all tp
        const aveTp = aveTpFn(tp);

        if (aveTp === 0) {
          this.selectedStudent.exp -= tempExp;
          this.selectedStudent.tp = 0;
          return;
        }
        this.selectedStudent.exp += this.tpExps[aveTp - 1] - tempExp;
        this.selectedStudent.tp = aveTp;
        this.selectedStudent.tpStr = tp;

        this.selectedStudent = {};
      }
      // type === group
      else if (type === "group") {
        this.selectedGroup.tpStr = tp;
        this.selectedGroup.students.forEach((s: any) => {
          this.selectedStudent = s;
          this.selectTp({ val: tp, type: "student" });
        });
        this.selectedGroup = {};
      }
      this.isOpenModalTp = false;
      this.setDefaultEvalute();
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
          this.selectTp({ val: `[${value}]`, type: "student" });
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
    grouping() {
      this.isShowGrouping = true;
      this.groups = [];
      const JSONTpStr = this.getTpStrDefault(this.evaluateCount);

      for (let i = 0; i < this.groupTotal; i++) {
        this.groups.push({
          students: [],
          tpStr: JSONTpStr === false ? "[0]" : JSONTpStr,
          name: this.indexToAlphabet(i),
        });
      }

      for (let i = 0; i < this.datas.length; i++) {
        const groupIndex = Math.floor(
          i / Math.ceil(this.datas.length / this.groupTotal)
        );
        this.groups[groupIndex].students.push(this.datas[i]);
      }
    },
    indexToAlphabet(val: number) {
      const alphabets = ["A", "B", "C", "D", "E", "F", "G", "H"];
      return alphabets[val];
    },
    openModalTpForGroup(val: number) {
      this.selectedGroup = this.groups[val];
      this.isOpenModalTp = true;
    },
    getTpStrDefault(evaluateCount: number) {
      const allZero = this.datas.every((d: any) => {
        const sum = JSON.parse(d.tpStr).reduce(
          (v: number, t: number) => (t += v),
          0
        );
        return sum === 0;
      });

      if (allZero) {
        const arr = [];
        for (let i = 0; i < evaluateCount; i++) {
          arr.push(0);
        }
        return JSON.stringify(arr);
      }

      return false;
    },
    setDefaultEvalute() {
      // show or hide evalute input
      const studentsSumTps = this.datas.every((d: any) => {
        const sum = JSON.parse(d.tpStr).reduce(
          (v: number, t: number) => (t += v),
          0
        );
        return sum === 0;
      });
      const groupSumTps = this.groups.every((g: any) => {
        const sum = JSON.parse(g.tpStr).reduce(
          (v: number, t: number) => (t += v),
          0
        );
        return sum === 0;
      });
      this.isDefaultEvalute = studentsSumTps && groupSumTps;
    },
    updateTextSize(val: string) {
      if (val === "+") {
        this.textSize++;
      } else {
        this.textSize--;
      }
    },
  },
  watch: {
    evaluateCount(newVal, oldVal) {
      if (newVal !== oldVal) {
        const JSONTpStr = this.getTpStrDefault(newVal);
        if (JSONTpStr !== false) {
          this.datas.forEach((d: any) => (d.tpStr = JSONTpStr));
        }

        if (this.isShowGrouping) {
          this.groups.forEach((d: any) => (d.tpStr = JSONTpStr));
        }
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
  section {
    display: flex;
    margin-top: 5px;
    & > * {
      margin-left: 5px;
    }
    & > section {
      margin: 0;
    }
  }
}
.seats {
  margin-top: 10px;
  display: grid;
  grid-gap: 10px;
  grid-template-columns: repeat(5, 1fr);
  @media (max-width: 800px) {
    grid-template-columns: repeat(2, 1fr);
  }
  &.group {
    width: 50%;
    grid-gap: 2px;
    float: left;
  }
}
.grouping {
  margin-top: 10px;
  width: 50%;
  float: left;
  padding: 10px;
  .group {
    border: 1px solid white;
    padding: 10px;
    cursor: pointer;
    &:hover {
      border: 3px solid white;
    }
    .top {
      display: flex;
      align-items: center;

      .title {
        margin-right: 8px;
        font-size: 18px;
        padding: 5px;
        font-weight: 700;
        background-color: white;
        color: #222222;
        display: grid;
        place-content: center;
      }
      .name {
        span {
          margin-right: 5px;
          white-space: nowrap;
        }
      }
    }
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

.testBox {
  margin-top: 10px;
  background-color: white;
  color: black;
  padding: 10px;
  width: 100%;
  resize: vertical;
}
</style>
