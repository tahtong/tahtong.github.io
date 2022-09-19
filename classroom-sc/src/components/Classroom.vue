<template>
  <div class="header">
    <!-- subject -->
    <div class="subject">
      <div v-show="teacherMode" class="title">
        <span>课题</span>
        <input v-model="finalResult.title" type="text" />
      </div>
      <div v-show="teacherMode" class="title">
        <span>TP</span>
        <input v-model="selectedTp" type="number" max="6" min="0" />
      </div>
    </div>
    <!-- pointClassroom -->
    <div class="pointClassroom">
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
      <button
        v-show="teacherMode"
        @click="allExp(-1)"
        class="btn primary minus"
      >
        -1 Exp
      </button>
      <button
        v-show="teacherMode"
        @click="updateResult(datas)"
        class="btn primary"
      >
        Update
      </button>
    </div>
  </div>

  <!-- seat -->
  <div class="seats">
    <Seat
      v-for="data in datas"
      :key="data.seat"
      :data="data"
      :teacher-mode="teacherMode"
      @tp="openModalTp"
      @exp="singleExp"
      @absent="absent"
      @skill="openModalSkill"
    />
  </div>

  <!-- ModalTp -->
  <ModalTp
    :open="isOpenModalTp"
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
</template>

<script lang="ts">
import { defineComponent, onMounted, ref } from "vue";
import Seat from "./Seat.vue";
import ModalTp from "./ModalTp.vue";
import ModalRadar from "./ModalRadar.vue";

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
      finalResult: {
        title: "",
      },
      isOpenModalTp: false,
      selectedStudent: Object as any,
      isOpenModalRadar: false,
      selectedSkills: [0, 0, 0, 0, 0, 0],
      isShowResult: false,
      labels: ["未来目标", "理解力", "逻辑力", "自控力", "企业能力", "德行"],
      selectedTp: 1,
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
        obj.tp = 0;
        obj["absent"] = false;
        return obj;
      });
      results.sort((a: any, b: any) => a.seat - b.seat);
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
          data.tp,
        ];
        return obj;
      });

      fetch(this.scriptUrl, {
        method: "POST",
        mode: "no-cors",
        cache: "no-cache",
        headers: {
          "Content-Type": "application/json",
        },
        redirect: "follow",
        body: JSON.stringify({ data: results }),
      }).then(() => alert('Response'));
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
    selectTp(val: number) {
      let tempExp = 0;
      if (this.selectedStudent.tp !== 0) {
        tempExp = this.tpExps[this.selectedStudent.tp - 1];
      }
      // remove all tp
      if (val === 0) {
        this.selectedStudent.exp -= tempExp;
        this.selectedStudent.tp = 0;
        return;
      }
      this.selectedStudent.exp += this.tpExps[val - 1] - tempExp;
      this.selectedStudent.tp = val;
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
  },
});
</script>

<style lang="scss" scoped>
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.subject {
  display: flex;
  align-items: center;
  height: 50px;

  span {
    margin-right: 10px;
  }

  .title {
    margin-right: 10px;
  }
  .tp {
  }
}
.pointClassroom {
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
</style>
