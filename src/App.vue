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
      <li @click.stop="setSort('asc')">A - Z</li>
      <li @click.stop="setSort('desc')">Z - A</li>
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
    @view="handleView"
    @edit="handleEdit"
    @delete="handleDelete" />

  <div class="create-employee">
    <button class="control" @click="openAddForm">Create Employee</button>
  </div>
</template>

<script>
import Navbar from "./components/NavBar.vue";
import Table from "./components/Table.vue";
import employeeData from "./data/purple_cross_employees.json";
import ConfirmDeleteModal from "./components/ConfirmDelete.vue";
import EmployeeFormModal from "./components/EmployeeForm.vue";
import ViewEmployeeModal from "./components/ViewEmployee.vue";

export default {
  name: "App",
  components: {
    Navbar,
    Table,
    ConfirmDeleteModal,
    EmployeeFormModal,
    ViewEmployeeModal,
  },
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

      showFilterMenu: false,
      showValueMenu: false,
      showSortMenu: false,
      filterMenuPos: { x: 0, y: 0 },
      valueMenuPos: { y: 0 },
      sortMenuPos: { x: 0, y: 0 },

      filterColumn: null,
      filterValue: null,
      sortField: "fullName",
      sortDirection: null,

      showDeleteConfirm: false,
      rowPendingDelete: null,

      showEmployeeForm: false,
      employeeToEdit: null,

      showEmployeeViewModal: false,
      employeeToView: null,
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

    //buttons behavior
    handleView(row) {
      this.employeeToView = row;
      this.showEmployeeViewModal = true;
    },
    handleEdit(row) {
      this.openEditForm(row);
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

      this.rowPendingDelete = null;
      this.showDeleteConfirm = false;
    },

    handleClickOutside(e) {
      const btns = [this.$refs.filterBtn, this.$refs.sortBtn];
      const clickedBtn = btns.some((ref) => ref && ref.contains(e.target));
      if (!clickedBtn) {
        this.showFilterMenu = false;
        this.showValueMenu = false;
        this.showSortMenu = false;
      }
    },

    //Edit employee modal
    openAddForm() {
      this.employeeToEdit = null;
      this.showEmployeeForm = true;
      this.showEmployeeViewModal = false;
    },
    openEditForm(row) {
      this.employeeToEdit = row;
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
        Object.assign(this.employeeToEdit, formatted);
      } else {
        this.employeesData.forEach((emp) => emp.originalIndex++);

        this.employeesData.unshift({
          ...formatted,
          originalIndex: 0,
        });
      }
      this.showEmployeeForm = false;
    },
  },

  mounted() {
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
  width: 18px;
  height: 18px;
  color: #6007a5;
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
