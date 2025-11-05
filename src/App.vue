<template>
  <Navbar />
</template>

<script>
import Navbar from "./components/NavBar.vue";
import employeeData from "./data/purple_cross_employees.json";

export default {
  name: "App",
  components: { Navbar },
  data() {
    return {
      employeesData: employeeData,
    };
  },
};
</script>

<style scoped></style>
