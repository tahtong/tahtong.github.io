<template>
  <div v-if="!accessAble" class="classrooms">
    <section>
      <button
        v-for="n in ['3M', '4H', '4M', '5H', '5K', '5M']"
        :key="n"
        @click="
          selectedClassroom = n;
          inputPassword.focus();
        "
        class="btn primary"
      >
        {{ n }}
      </button>
    </section>
    <br />
    <br />
    <section>
      <input
        ref="inputPassword"
        v-model="password"
        :style="{ 'opacity': selectedClassroom === '' ? 0.2 : 1 }"
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
  <Classroom
    v-else
    :sheetUrl="selectedSheetUrl"
    :scriptUrl="selectedScriptUrl"
  />
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import Classroom from "./components/Classroom.vue";

export default defineComponent({
  name: "App",
  setup() {
    const inputPassword = ref();
    return { inputPassword };
  },
  data() {
    return {
      selectedClassroom: "",
      accessAble: false,
      password: "",
      isPasswordWrong: false,
      sheetUrls: [
        {
          classroom: "3M",
          url: "1G5YN2BbiOm1C4d-f8lfSu-5BqXCQbTz4mqfyuWxFVrA",
          scriptUrl:
            "https://script.google.com/macros/s/AKfycbx83cG9Qu1wtQUAHxWzHHQMCmgk7Y1mlEjwvUWQpF-xdwk-Q58kwS2jXDiy5y1zikQx/exec",
        },
        {
          classroom: "4H",
          url: "1Oc1wyRSL77jriyCf2-3aHKPdQbNWEctzrK2uzm_F-FQ",
          scriptUrl:
            "https://script.google.com/macros/s/AKfycbwToRyVVFuamLI4XcXoUwJVVNsxxEcNMm4zomH_AjCK2evhH2NMeL3dbz1gTaU-IUKNJw/exec",
        },
        {
          classroom: "4M",
          url: "1dnYsdsb1D-uBf4e3Tl0UyFi4JGe9shy3Un7mX4qyX9k",
          scriptUrl:
            "https://script.google.com/macros/s/AKfycbwBq0UogO2a_mtdRby4McTm6Ch-RPNHxyyV1s70H_D7cDgV5WWdKD04jHBgAB9l03yzfw/exec",
        },
        {
          classroom: "5H",
          url: "1h7I_XLNA4NnL91nUENt9N2tw3-xrqgxHVXm3dCaVlE4",
          scriptUrl:
            "https://script.google.com/macros/s/AKfycbxIyki3IG6zYOKxchMpVVc_sCSG84t4aiBcIdUA-W4C4TuXznBK7HSKqKQlZ4nCFnFB/exec",
        },
        {
          classroom: "5K",
          url: "1h_Qk_o_fVGqwh-lbtqdbX8Pq7zYTDOGUCjFNutlBpO8",
          scriptUrl:
            "https://script.google.com/macros/s/AKfycbyPR7proIeJ1TLFIPd21XiddFTJM2e68vI_i19xyQgQA_DBQBt0xauPFMvwljWgMRftug/exec",
        },
        {
          classroom: "5M",
          url: "1ov-4Eq_jA-cNI8ZP_7Pjav-4vyZzeSeTTZkAgUMHwPA",
          scriptUrl:
            "https://script.google.com/macros/s/AKfycbyPqew4HUWG1g0DSDvyqkuxOJZRIhZMCDzU5mZdbCxSZHIkn8Y3mviJEQHnuS57kJ9u/exec",
        },
      ],
      selectedSheetUrl: "",
      selectedScriptUrl: "",
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
        this.selectedScriptUrl = filteredClassroom.scriptUrl;
        document.title = selected + " 发展联盟";
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
