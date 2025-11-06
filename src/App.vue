<template>
  <Navbar />
  <Table
    :columns="tableColumns"
    :data="employeesData"
    @view="handleView"
    @edit="handleEdit"
    @delete="handleDelete" />
</template>

<script>
import Navbar from "./components/NavBar.vue";
import Table from "./components/Table.vue";
import employeeData from "./data/purple_cross_employees.json";

export default {
  name: "App",
  components: { Navbar, Table },
  data() {
    return {
      employeesData: this.formatEmployeeData(employeeData),
      tableColumns: [
        { label: "Full Name", field: "fullName" },
        { label: "Occupation", field: "occupation" },
        { label: "Department", field: "department" },
        { label: "Date of Employment", field: "dateOfEmployment" },
        { label: "Termination Date ", field: "terminationDate" },
      ],
    };
  },
  methods: {
    formatEmployeeData(data) {
      const today = new Date().setHours(0, 0, 0, 0);

      return data.map((emp) => {
        const employmentDate = new Date(emp.dateOfEmployment).setHours(
          0,
          0,
          0,
          0
        );
        const terminationDate = emp.terminationDate
          ? new Date(emp.terminationDate).setHours(0, 0, 0, 0)
          : null;

        return {
          ...emp,
          dateOfEmployment:
            employmentDate > today ? "Employed soon" : "Currently employed",

          terminationDate:
            terminationDate === null
              ? "To be terminated"
              : terminationDate > today
              ? "To be terminated"
              : "Terminated",
        };
      });
    },

    handleView(row) {
      console.log("Viewing:", row);
    },
    handleEdit(row) {
      console.log("Editing:", row);
    },
    handleDelete(row) {
      console.log("Deleting:", row);
    },
  },
};
</script>

<style scoped></style>
