<template>
  <Navbar />

  <Toolbar
    :tableColumns="tableColumns"
    :employeesData="employeesData"
    :filterColumn="filterColumn"
    :filterValue="filterValue"
    :sortDirection="sortDirection"
    @update:filterColumn="filterColumn = $event"
    @update:filterValue="filterValue = $event"
    @update:sortDirection="sortDirection = $event"
    @import-json="importJSON"
    @export-json="exportJSON"
    @export-csv="exportCSV" />

  <ConfirmDeleteModal
    v-if="showDeleteConfirm"
    :itemName="rowPendingDelete?.fullName"
    @confirm="confirmDelete"
    @cancel="showDeleteConfirm = false" />

  <ViewEmployeeModal
    v-if="showEmployeeViewModal"
    :employee="employeeToView"
    @edit="handleEdit"
    @cancel="showEmployeeViewModal = false" />

  <EmployeeFormModal
    v-if="showEmployeeForm"
    :employee="employeeToEdit"
    @save="saveEmployee"
    @cancel="showEmployeeForm = false" />

  <Table
    :columns="tableColumns"
    :data="processedData"
    :loading="loading"
    @view="handleView"
    @edit="handleEdit"
    @delete="handleDelete" />

  <div class="create-employee">
    <button class="control" @click="openAddForm">Create Employee</button>
  </div>
  <Toast ref="toast" />
</template>

<script>
import Navbar from "./components/NavBar.vue";
import Table from "./components/Table.vue";
import employeeData from "./data/purple_cross_employees.json";
import ConfirmDeleteModal from "./components/ConfirmDelete.vue";
import EmployeeFormModal from "./components/EmployeeForm.vue";
import ViewEmployeeModal from "./components/ViewEmployee.vue";
import Toast from "./components/Toast.vue";
import Toolbar from "./components/Toolbar.vue";

export default {
  name: "App",
  components: {
    Navbar,
    Table,
    ConfirmDeleteModal,
    EmployeeFormModal,
    ViewEmployeeModal,
    Toast,
    Toolbar,
  },
  data() {
    return {
      employeesData: [],
      loading: false,
      tableColumns: [
        { label: "Full Name", field: "fullName" },
        { label: "Occupation", field: "occupation" },
        { label: "Department", field: "department" },
        { label: "Date of Employment", field: "dateOfEmployment" },
        { label: "Termination Date ", field: "terminationDate" },
      ],

      //Filter values
      filterColumn: null,
      filterValue: null,

      //Field directive to sort
      sortField: "fullName",
      sortDirection: null,

      // Employee Delete Modal
      showDeleteConfirm: false,
      rowPendingDelete: null,

      // Employee Edit Modal
      showEmployeeForm: false,
      employeeToEdit: null,

      // Employee View Modal
      showEmployeeViewModal: false,
      employeeToView: null,
    };
  },
  computed: {
    processedData() {
      let rows = [...this.employeesData];

      if (this.filterColumn && this.filterValue !== null) {
        const f = this.filterColumn.field;
        rows = rows.filter((r) => String(r[f]) === String(this.filterValue));
      }

      if (this.sortDirection) {
        const dir = this.sortDirection === "asc" ? 1 : -1;
        const field = this.sortField;
        rows.sort(
          (a, b) =>
            String(a[field] ?? "").localeCompare(String(b[field] ?? "")) * dir
        );
      } else {
        rows.sort((a, b) => a.originalIndex - b.originalIndex);
      }

      return rows;
    },
  },
  methods: {
    async fetchData() {
      //simulating real fetch from actual API
      this.loading = true;
      return new Promise((resolve) => {
        setTimeout(() => {
          resolve(this.formatEmployeeData(employeeData));
        }, 1000);
      });
    },
    async loadData() {
      this.employeesData = await this.fetchData(); // Wait for fetched data
      this.loading = false;
    },

    formatEmployeeData(data) {
      const today = new Date().setHours(0, 0, 0, 0);

      return data.map((emp, index) => {
        const employmentDateRaw = emp.dateOfEmployment || null;
        const terminationDateRaw = emp.terminationDate || null;

        const employmentDate = new Date(employmentDateRaw).setHours(0, 0, 0, 0);
        const terminationDate = emp.terminationDate
          ? new Date(terminationDateRaw).setHours(0, 0, 0, 0)
          : null;

        return {
          ...emp,
          originalIndex: index,

          rawEmploymentDate: employmentDateRaw,
          rawTerminationDate: terminationDateRaw,

          dateOfEmployment:
            terminationDate !== null && terminationDate <= today
              ? "Terminated"
              : employmentDate > today
              ? "Employed soon"
              : "Currently employed",

          terminationDate:
            terminationDate === null
              ? "No date"
              : terminationDate > today
              ? "To be terminated"
              : "Terminated",
        };
      });
    },

    //import file validation
    importJSON(event) {
      const file = event.target.files[0];
      if (!file) return;

      const reader = new FileReader();

      reader.onload = (e) => {
        try {
          const imported = JSON.parse(e.target.result);

          if (!Array.isArray(imported)) {
            this.$refs.toast.show(
              "JSON must be an array of employees.",
              "error"
            );
            return;
          }

          const formatted = this.formatEmployeeData(imported);

          // Add new employees to the top of the list
          formatted.forEach((emp) => {
            const exists = this.employeesData.some(
              (existing) =>
                existing.code?.toLowerCase() === emp.code?.toLowerCase()
            );

            if (!exists) {
              this.employeesData.unshift(emp);
            }
          });

          this.employeesData.forEach((row, i) => {
            row.originalIndex = i;
          });

          this.$refs.toast.show(
            "✅ Imported successfully. New employees added to top.",
            "success"
          );
        } catch {
          this.$refs.toast.show("❌ Invalid JSON file", "error");
        }
      };
      reader.readAsText(file);
    },

    //prepare export data

    prepareExportData() {
      return this.employeesData.map((emp) => {
        return {
          code: emp.code,
          fullName: emp.fullName,
          occupation: emp.occupation,
          department: emp.department,
          dateOfEmployment: emp.rawEmploymentDate || null, // Export the real date
          terminationDate: emp.rawTerminationDate || null, // Export real date or empty
        };
      });
    },

    //export JSON
    exportJSON() {
      const cleanData = this.prepareExportData();
      const blob = new Blob([JSON.stringify(cleanData, null, 2)], {
        type: "application/json",
      });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "employees.json";
      a.click();
      URL.revokeObjectURL(url);
    },

    //export csv

    exportCSV() {
      const cleanData = this.prepareExportData();
      if (!cleanData.length) return;
      const columns = Object.keys(cleanData[0]);
      const header = columns.join(",");
      const rows = cleanData.map((emp) =>
        columns
          .map((k) => `"${(emp[k] ?? "").toString().replace(/"/g, '""')}"`)
          .join(",")
      );
      const csvContent = [header, ...rows].join("\n");

      const blob = new Blob([csvContent], { type: "text/csv;charset=utf-8;" });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "employees.csv";
      a.click();
      URL.revokeObjectURL(url);
    },

    //buttons behavior
    handleView(row) {
      this.employeeToView = row;
      this.showEmployeeViewModal = true;
    },
    handleEdit(row) {
      this.employeeToEdit = row;
      this.showEmployeeForm = true;
      this.showEmployeeViewModal = false;
    },
    handleDelete(row) {
      this.rowPendingDelete = row;
      this.showDeleteConfirm = true;
    },

    //confirm Delete Modal
    confirmDelete() {
      this.employeesData = this.employeesData.filter(
        (emp) => emp !== this.rowPendingDelete
      );
      this.$refs.toast.show("✅ Employee deleted successfully", "success");
      this.rowPendingDelete = null;
      this.showDeleteConfirm = false;
    },

    //Add employee modal
    openAddForm() {
      this.employeeToEdit = null;
      this.showEmployeeForm = true;
      this.showEmployeeViewModal = false;
    },

    saveEmployee(data) {
      const today = new Date().setHours(0, 0, 0, 0);
      const rawEmploymentDate =
        data.rawEmploymentDate || new Date().toISOString().slice(0, 10);

      const employmentDate = new Date(rawEmploymentDate).setHours(0, 0, 0, 0);

      const rawTerminationDate = data.rawTerminationDate || null;
      const terminationDate = rawTerminationDate
        ? new Date(rawTerminationDate).setHours(0, 0, 0, 0)
        : null;

      const formatted = {
        ...data,
        rawEmploymentDate,
        rawTerminationDate,

        dateOfEmployment:
          employmentDate > today ? "Employed soon" : "Currently employed",

        terminationDate:
          terminationDate === null
            ? "No date"
            : terminationDate > today
            ? "To be terminated"
            : "Terminated",
      };

      if (this.employeeToEdit) {
        //handling edit employee
        Object.assign(this.employeeToEdit, formatted);
        this.$refs.toast.show("✅ Employee saved successfully", "success");
      } else {
        //handling add new employee
        this.employeesData.forEach((emp) => emp.originalIndex++);

        this.employeesData.unshift({
          ...formatted,
          originalIndex: 0,
        });
        this.$refs.toast.show("✅ Employee created successfully", "success");
      }
      this.showEmployeeForm = false;
    },
  },

  mounted() {
    this.loadData();
  },
};
</script>

<style scoped>
.create-employee {
  position: fixed;
  bottom: 24px;
  right: 24px;
  z-index: 1200;
}

.create-employee .control {
  color: #f1f1f1;
  padding: 10px 16px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.45);
  transition: background 0.3s ease;
}

.create-employee .control:hover {
  background: #4c0583;
}

/* -- Mobile -- */

@media (max-width: 768px) {
  .toolbar {
    padding: 10px 12px;
  }
  .menu {
    min-width: 180px;
  }
}
</style>
