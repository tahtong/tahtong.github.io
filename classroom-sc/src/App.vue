<template>
  <div v-if="!accessAble" class="classrooms">
    <section>
      <button
        v-for="n in ['3M', '4H', '4M', '5H', '5K', '5M']"
        :key="n"
        @click="selectedClassroom = n"
        class="btn primary"
      >
        {{ n }}
      </button>
    </section>
    <br />
    <br />
    <section>
      <input
        v-model="password"
        :disabled="selectedClassroom == ''"
        type="password"
      />
      <button
        @click="checkPassword(selectedClassroom, password)"
        :disabled="selectedClassroom == ''"
        class="btn primary"
      >
        Go <span>{{ selectedClassroom }}</span>
      </button>
    </section>
    <br />
    <br />
    <p v-if="isPasswordWrong">Wrong Password!</p>
  </div>
  <Classroom v-else :sheetUrl="selectedSheetUrl" />
</template>

<script lang="ts">
import { defineComponent } from "vue";
import Classroom from "./components/Classroom.vue";

export default defineComponent({
  name: "App",
  data() {
    return {
      selectedClassroom: "",
      accessAble: false,
      password: "",
      isPasswordWrong: false,
      sheetUrls: [
        {
          classroom: "3M",
          totalStudent: 26,
          url: "1G5YN2BbiOm1C4d-f8lfSu-5BqXCQbTz4mqfyuWxFVrA",
        },
        {
          classroom: "4H",
          totalStudent: 23,
          url: "1Oc1wyRSL77jriyCf2-3aHKPdQbNWEctzrK2uzm_F-FQ",
        },
        {
          classroom: "4M",
          totalStudent: 24,
          url: "1dnYsdsb1D-uBf4e3Tl0UyFi4JGe9shy3Un7mX4qyX9k",
        },
        {
          classroom: "5H",
          totalStudent: 24,
          url: "1h7I_XLNA4NnL91nUENt9N2tw3-xrqgxHVXm3dCaVlE4",
        },
        {
          classroom: "5K",
          totalStudent: 27,
          url: "1h_Qk_o_fVGqwh-lbtqdbX8Pq7zYTDOGUCjFNutlBpO8",
        },
        {
          classroom: "5M",
          totalStudent: 26,
          url: "1ov-4Eq_jA-cNI8ZP_7Pjav-4vyZzeSeTTZkAgUMHwPA",
        },
      ],
      selectedSheetUrl: "",
    };
  },
  components: {
    Classroom,
  },
  methods: {
    checkPassword(selected: any, value: any) {
      this.isPasswordWrong = false;
      if (
        (selected === "3M" && (value === "1947" || value === "tigamerah")) ||
        (selected === "4H" && (value === "1947" || value === "empathijau")) ||
        (selected === "4M" && (value === "1947" || value === "empatmerah")) ||
        (selected === "5H" && (value === "1947" || value === "limahijiao")) ||
        (selected === "5K" && (value === "1947" || value === "limakuning")) ||
        (selected === "5M" && (value === "1947" || value === "limamerah"))
      ) {
        this.accessAble = true;
        const filteredClassroom = this.sheetUrls.filter(
          (s: any) => s.classroom === this.selectedClassroom
        )[0];
        this.selectedSheetUrl = filteredClassroom.url;
        document.title = selected + " 发展联盟";
        (window as any).gsUrl = this.selectedSheetUrl;
        (window as any).totalUpdateRow = filteredClassroom.totalStudent;
      } else {
        this.isPasswordWrong = true;
      }
    },
  },
});
</script>

<style lang="scss">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  color: white;
  background: #222;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  padding: 20px 20px 20px 50px;
  @media (max-width: 800px) {
    padding: 8px;
  }
}
input {
  border: 1px solid grey;
  height: 38px;
  font-size: 20px;
  width: 70px;
  background-color: #222;
  color: white;
  padding-left: 5px;
  &[type="number"] {
    width: 50px;
    text-align: center;
  }
  &[type="password"] {
    text-align: center;
    vertical-align: bottom;
  }
  &[disabled] {
    opacity: 0.2;
  }
  border-radius: 5px;
}
.btn {
  border-radius: 5px;
  border: 1px solid grey;
  background: none;
  padding: 0 10px;
  color: white;
  cursor: pointer;
  transition: 0.3s;
  &.minus {
    &:hover {
      background-color: #f28c8c;
      color: black;
    }
  }
  &.plus {
    &:hover {
      background-color: #f0f28c;
      color: black;
    }
  }
  &.primary {
    height: 38px;
  }
  &[disabled] {
    opacity: 0.2;
  }
}
.classrooms {
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
</style>
