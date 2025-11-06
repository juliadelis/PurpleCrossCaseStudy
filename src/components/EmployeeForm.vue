<template>
  <div class="modal-overlay">
    <div class="modal">
      <h2>{{ isEdit ? "Edit Employee" : "Create Employee" }}</h2>
      <div class="modal-body">
        <form @submit.prevent="onSave">
          <div class="form-group">
            <label>Code *</label>
            <input v-model="form.code" :class="{ error: errors.code }" />
            <p v-if="errors.code" class="error-text">{{ errors.code }}</p>
          </div>
          <div class="form-group">
            <label>Full Name *</label>
            <input
              v-model="form.fullName"
              :class="{ error: errors.fullName }" />
            <p v-if="errors.fullName" class="error-text">
              {{ errors.fullName }}
            </p>
          </div>
          <div class="form-group">
            <label>Occupation *</label>
            <input
              v-model="form.occupation"
              :class="{ error: errors.occupation }" />
            <p v-if="errors.occupation" class="error-text">
              {{ errors.occupation }}
            </p>
          </div>
          <div class="form-group">
            <label>Department *</label>
            <input
              v-model="form.department"
              :class="{ error: errors.department }" />
            <p v-if="errors.department" class="error-text">
              {{ errors.department }}
            </p>
          </div>
          <div class="form-group">
            <label>Date of Employment</label>
            <input v-model="form.rawEmploymentDate" type="date" />
          </div>
          <div class="form-group">
            <label>Termination Date</label>
            <input
              v-model="form.rawTerminationDate"
              type="date"
              :class="{ error: errors.rawTerminationDate }" />
            <p v-if="errors.rawTerminationDate" class="error-text">
              {{ errors.rawTerminationDate }}
            </p>
          </div>
        </form>
      </div>

      <div class="actions">
        <button class="save-btn" @click="onSave">
          {{ isEdit ? "Save Changes" : "Add Employee" }}
        </button>
        <button type="button" class="cancel-btn" @click="$emit('cancel')">
          Cancel
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "EmployeeFormModal",
  props: {
    employee: {
      type: Object,
      default: null,
    },
  },
  computed: {
    isEdit() {
      return !!this.employee;
    },
  },
  data() {
    return {
      form: {
        code: this.employee?.code || "",
        fullName: this.employee?.fullName || "",
        occupation: this.employee?.occupation || "",
        department: this.employee?.department || "",
        rawEmploymentDate: this.employee?.rawEmploymentDate || "",
        rawTerminationDate: this.employee?.rawTerminationDate || "",
      },
      errors: {},
    };
  },
  methods: {
    validate() {
      this.errors = {};

      if (!this.form.code.trim()) this.errors.code = "Code is required.";
      if (!this.form.fullName.trim())
        this.errors.fullName = "Full Name is required.";
      if (!this.form.occupation.trim())
        this.errors.occupation = "Occupation is required.";
      if (!this.form.department.trim())
        this.errors.department = "Department is required.";

      const start = this.form.rawEmploymentDate
        ? new Date(this.form.rawEmploymentDate)
        : null;
      const end = this.form.rawTerminationDate
        ? new Date(this.form.rawTerminationDate)
        : null;

      if (start && end && start > end) {
        this.errors.rawTerminationDate =
          "Termination date cannot be earlier than employment date.";
      }

      return Object.keys(this.errors).length === 0;
    },

    onSave() {
      if (!this.validate()) return;
      this.$emit("save", { ...this.form });
    },
  },
};
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: auto;
  overscroll-behavior: contain;
  z-index: 2000;
}
.modal {
  background: #f1f1f1;
  padding: 24px 28px;
  border-radius: 8px;
  width: 400px;
  max-height: 80vh;
  display: flex;
  flex-direction: column;
}

h2 {
  margin: 0;
}

.modal-body {
  flex: 1 1 auto;
  overflow-y: auto;
  margin-top: 12px;
  padding-right: 6px;
}

.form-group {
  display: flex;
  flex-direction: column;
  margin-bottom: 12px;
}

label {
  font-weight: 600;
  display: block;
}

input {
  width: 100%;
  padding: 8px;
  margin-top: 4px;
  border-radius: 6px;
  border: 1px solid #ccc;
}

input.error {
  border: 1px solid #e63946;
}

.error-text {
  font-size: 12px;
  color: #e63946;
  margin-top: 2px;
}

.actions {
  flex-shrink: 0;
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 16px;
}

.save-btn {
  background: #6007a5;
  color: #f1f1f1;
  border: none;
  padding: 8px 14px;
  border-radius: 6px;
  cursor: pointer;
}

.cancel-btn {
  background: #e2e2e2;
  border: none;
  padding: 8px 14px;
  cursor: pointer;
  color: #1a1a1a;
}

/* -- Mobile -- */

@media (max-width: 768px) {
  .modal-overlay {
    padding: 0px 20px;
  }
}
</style>
