<template>
  <Navbar />

  <div class="toolbar">
    <div>
      <H1>Employees</H1>
    </div>
    <div>
      <div class="control" @click.stop="toggleFilterMenu" ref="filterBtn">
        <svg class="icon" viewBox="0 0 24 24">
          <path fill="currentColor" d="M3 4h18l-7 8v5l-4 3v-8L3 4z" />
        </svg>
        <span>Filter</span>
      </div>

      <div class="control" @click.stop="toggleSortMenu" ref="sortBtn">
        <svg class="icon" viewBox="0 0 24 24">
          <path
            fill="currentColor"
            d="M7 3l4 4H8v10H6V7H3l4-4zm10 18l-4-4h3V7h2v10h3l-4 4z" />
        </svg>
        <span>Sort</span>
      </div>
      <label class="control" ref="importBtn" @click.stop>
        <svg
          width="13"
          height="13"
          viewBox="0 0 13 13"
          fill="none"
          xmlns="http://www.w3.org/2000/svg">
          <path
            d="M11.7941 6.19473V10.7618C11.7941 10.9566 11.636 11.1147 11.4412 11.1147H1.55882C1.364 11.1147 1.20588 10.9566 1.20588 10.7618V6.1912H0.5V10.7618C0.5 11.3456 0.975059 11.8206 1.55882 11.8206H11.4412C12.0249 11.8206 12.5 11.3456 12.5 10.7618V6.19473H11.7941Z"
            fill="currentColor" />
          <path
            d="M3.14941 6.05882L6.57506 9.46824L10 6.05953L9.50235 5.55906L6.928 8.12141V0.5H6.22212V8.12141L3.64706 5.55835L3.14941 6.05882Z"
            fill="currentColor" />
          <path
            d="M11.7941 6.19473V10.7618C11.7941 10.9566 11.636 11.1147 11.4412 11.1147H1.55882C1.364 11.1147 1.20588 10.9566 1.20588 10.7618V6.1912H0.5V10.7618C0.5 11.3456 0.975059 11.8206 1.55882 11.8206H11.4412C12.0249 11.8206 12.5 11.3456 12.5 10.7618V6.19473H11.7941Z"
            stroke="currentColor"
            stroke-linejoin="round" />
          <path
            d="M3.14941 6.05882L6.57506 9.46824L10 6.05953L9.50235 5.55906L6.928 8.12141V0.5H6.22212V8.12141L3.64706 5.55835L3.14941 6.05882Z"
            stroke="currentColor"
            stroke-linejoin="round" />
        </svg>

        <span>Import</span>
        <input
          type="file"
          accept=".json"
          @change="importJSON"
          style="display: none" />
      </label>

      <div class="control" @click.stop="toggleExportMenu" ref="exportBtn">
        <svg
          width="13"
          height="14"
          viewBox="0 0 13 14"
          fill="none"
          xmlns="http://www.w3.org/2000/svg">
          <path
            d="M3.57694 4.74271L3.07012 4.25071L6.5 0.717773L9.92988 4.25071L9.42306 4.74201L6.85294 2.09495V9.69519H6.14706V2.09495L3.57694 4.74271ZM11.7941 6.9126V11.4797C11.7941 11.6745 11.636 11.8326 11.4412 11.8326H1.55882C1.364 11.8326 1.20588 11.6745 1.20588 11.4797V6.90907H0.5V11.4797C0.5 12.0634 0.975059 12.5385 1.55882 12.5385H11.4412C12.0249 12.5385 12.5 12.0634 12.5 11.4797V6.9126H11.7941Z"
            fill="currentColor"
            stroke="currentColor" />
        </svg>
        <span>Export</span>
      </div>

      <ul v-if="showExportMenu" class="menu" :style="menuStyle(exportMenuPos)">
        <li @click.stop="exportJSON">JSON</li>
        <li @click.stop="exportCSV">CSV (Excel)</li>
        <li class="muted" @click.stop="closeExportMenu">Cancel</li>
      </ul>
    </div>
    <ul v-if="showFilterMenu" class="menu" :style="menuStyle(filterMenuPos)">
      <li
        v-for="col in filterableColumns"
        :key="col.field"
        @click.stop="openValueMenu(col)">
        {{ col.label }}
      </li>
      <li class="muted" @click.stop="clearFilter">Clear filter</li>
    </ul>

    <ul v-if="showValueMenu" class="menu" :style="menuStyle(valueMenuPos)">
      <li
        v-for="val in uniqueValues"
        :key="String(val)"
        @click.stop="applyFilter(val)"
        class="value-item">
        <span>{{ displayValue(val) }}</span>

        <svg
          v-if="filterValue === val"
          class="check"
          viewBox="0 0 25 19"
          fill="none"
          xmlns="http://www.w3.org/2000/svg">
          <path
            d="M8.16394 14.1684L2.45248 8.5424L0 10.9582L8.16394 19L25 2.41579L22.5475 0L8.16394 14.1684Z"
            fill="currentColor" />
        </svg>
      </li>
      <li class="muted" @click.stop="closeValueMenu">Cancel</li>
    </ul>

    <ul v-if="showSortMenu" class="menu" :style="menuStyle(sortMenuPos)">
      <li @click.stop="setSort('asc')" class="sortCheck">
        A - Z
        <svg
          v-if="sortDirection === 'asc'"
          class="check"
          viewBox="0 0 25 19"
          fill="none"
          xmlns="http://www.w3.org/2000/svg">
          <path
            d="M8.16394 14.1684L2.45248 8.5424L0 10.9582L8.16394 19L25 2.41579L22.5475 0L8.16394 14.1684Z"
            fill="currentColor" />
        </svg>
      </li>
      <li @click.stop="setSort('desc')" class="sortCheck">
        Z - A
        <svg
          v-if="sortDirection === 'desc'"
          class="check"
          viewBox="0 0 25 19"
          fill="none"
          xmlns="http://www.w3.org/2000/svg">
          <path
            d="M8.16394 14.1684L2.45248 8.5424L0 10.9582L8.16394 19L25 2.41579L22.5475 0L8.16394 14.1684Z"
            fill="currentColor" />
        </svg>
      </li>
      <li class="muted" @click.stop="setSort(null)">Reset Order</li>
    </ul>
  </div>

  <div class="chip-align">
    <div v-if="filterColumn && filterValue" class="active-filter-chip">
      <span>{{ filterColumn.label }}: {{ displayValue(filterValue) }}</span>
      <button class="chip-close" @click="clearFilter">✕</button>
    </div>
  </div>

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

export default {
  name: "App",
  components: {
    Navbar,
    Table,
    ConfirmDeleteModal,
    EmployeeFormModal,
    ViewEmployeeModal,
    Toast,
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

      // Filter menu
      showFilterMenu: false,
      showValueMenu: false,
      // Sort menu
      showSortMenu: false,
      // Filter menu position
      filterMenuPos: { x: 0, y: 0 },
      // Filter submenu position
      valueMenuPos: { y: 0 },
      // Sort menu position
      sortMenuPos: { x: 0, y: 0 },

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

      //export options
      showExportMenu: false,
      exportMenuPos: { x: 0, y: 0 },
    };
  },
  computed: {
    filterableColumns() {
      return this.tableColumns.filter((c) => c.field !== "fullName");
    },

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

    uniqueValues() {
      if (!this.filterColumn) return [];
      const f = this.filterColumn.field;
      const set = new Set(this.employeesData.map((r) => r[f]));
      return Array.from(set);
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

    //filter and order menu behavior
    menuStyle(pos) {
      return { left: pos.x + "px", top: pos.y + "px" };
    },
    getBtnPos(el) {
      const r = el.getBoundingClientRect();
      return { x: r.left - 140, y: r.bottom + 6 };
    },

    toggleFilterMenu() {
      if (this.showFilterMenu) {
        this.showFilterMenu = false;
        this.showValueMenu = false;
        return;
      }
      this.filterMenuPos = this.getBtnPos(this.$refs.filterBtn);
      this.showFilterMenu = true;
      this.showSortMenu = false;
      this.showValueMenu = false;
    },
    openValueMenu(col) {
      this.filterColumn = col;
      this.valueMenuPos = {
        x: this.filterMenuPos.x - 50,
        y: this.filterMenuPos.y + 40,
      };

      this.showValueMenu = true;
    },
    applyFilter(val) {
      this.filterValue = val;
      this.showValueMenu = false;
      this.showFilterMenu = false;
    },
    clearFilter() {
      this.filterColumn = null;
      this.filterValue = null;
      this.showFilterMenu = false;
      this.showValueMenu = false;
    },
    closeValueMenu() {
      this.showValueMenu = false;
      this.showFilterMenu = false;
    },

    displayValue(val) {
      return val === null || val === "" ? "No value" : val;
    },

    toggleSortMenu() {
      if (this.showSortMenu) {
        this.showSortMenu = false;
        return;
      }
      this.sortMenuPos = this.getBtnPos(this.$refs.sortBtn);
      this.showSortMenu = true;
      this.showFilterMenu = false;
      this.showValueMenu = false;
    },
    setSort(dir) {
      this.sortDirection = dir;
      this.showSortMenu = false;
    },

    //export menu logic

    toggleExportMenu() {
      if (this.showExportMenu) {
        this.showExportMenu = false;
        return;
      }
      this.exportMenuPos = this.getBtnPos(this.$refs.exportBtn);
      this.showExportMenu = true;

      //close other menus
      this.showFilterMenu = false;
      this.showValueMenu = false;
      this.showSortMenu = false;
    },

    closeExportMenu() {
      this.showExportMenu = false;
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

    //export JSON
    exportJSON() {
      const blob = new Blob([JSON.stringify(this.employeesData, null, 2)], {
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
      if (!this.employeesData.length) return;
      const columns = Object.keys(this.employeesData[0]);
      const header = columns.join(",");
      const rows = this.employeesData.map((emp) =>
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

    handleClickOutside(e) {
      const btns = [
        this.$refs.filterBtn,
        this.$refs.sortBtn,
        this.$refs.exportBtn,
        this.$refs.importBtn,
      ];
      const clickedBtn = btns.some((ref) => ref && ref.contains(e.target));
      if (!clickedBtn) {
        this.showFilterMenu = false;
        this.showValueMenu = false;
        this.showSortMenu = false;
        this.showExportMenu = false;
      }
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
    window.addEventListener("click", this.handleClickOutside);
  },
  beforeUnmount() {
    window.removeEventListener("click", this.handleClickOutside);
  },
};
</script>

<style scoped>
.toolbar {
  display: flex;
  gap: 16px;
  align-items: center;
  justify-content: space-between;
  padding: 0px 20px;
  margin-top: 20px;
  position: relative;
}

.control {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  color: #6007a5;
  cursor: pointer;
  user-select: none;
  padding: 6px 8px;
  border-radius: 8px;
}
.control:hover {
  background: #f5eefc;
}

.icon {
  width: 18px;
  height: 18px;
  display: block;
}

.menu {
  position: fixed;
  z-index: 1000;
  min-width: 200px;
  background: #fff;
  border-radius: 10px;
  padding: 8px 0;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
  border: 1px solid #eee;
}
.menu li {
  padding: 10px 14px;
  cursor: pointer;
  white-space: nowrap;
  list-style-type: none;
}
.menu li:hover {
  background: #f7f2ff;
}

.menu .muted {
  color: #7a7a7a;
}

.chip-align {
  display: flex;
  justify-content: end;
}

.active-filter-chip {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background: #f5eefc;
  border: 1px solid #d9c5f4;
  color: #6007a5;
  padding: 6px 12px;
  border-radius: 20px;
  margin: 8px 16px 0;
  font-size: 14px;
  justify-content: end;
}

.chip-close {
  background: transparent;
  border: none;
  color: #6007a5;
  cursor: pointer;
  font-size: 14px;
  padding: 0;
}

.value-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.check {
  width: 16px;
  height: 16px;
  color: #6007a5;
}

.sortCheck {
  display: flex;
  align-items: center;
  gap: 10px;
}

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
