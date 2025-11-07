<template>
  <div class="modal-overlay">
    <div class="modal-box">
      <h2>View Employee</h2>
      <div class="modal-body">
        <div class="items">
          <div class="big-group">
            <div class="group-item">
              <p class="title">Code:</p>
              <p>{{ employee.code ?? "-" }}</p>
            </div>
            <div class="group-item">
              <p class="title">Full Name:</p>
              <p>{{ employee.fullName ?? "-" }}</p>
            </div>
          </div>
          <div class="big-group">
            <div class="group-item">
              <p class="title">Occupation:</p>
              <p>{{ employee.occupation ?? "-" }}</p>
            </div>
            <div class="group-item">
              <p class="title">Department:</p>
              <p>{{ employee.department ?? "-" }}</p>
            </div>
          </div>
          <div class="big-group">
            <div class="group-item">
              <p class="title">Date of Employment:</p>
              <p>{{ formatDate(employee.rawEmploymentDate) }}</p>
            </div>
            <div class="group-item">
              <p class="title">Termination Date:</p>
              <p>{{ formatDate(employee.rawTerminationDate) }}</p>
            </div>
          </div>
        </div>
      </div>

      <div class="actions">
        <button class="edit" @click="$emit('edit', employee)">
          Edit Employee
        </button>
        <button class="cancel" @click="$emit('cancel')">Back</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ViewEmployeeModal",
  props: {
    employee: {
      type: Object,
      required: true,
    },
  },
  methods: {
    formatDate(date) {
      if (!date) return "-";
      const localDate = new Date(date + "T00:00");
      return localDate.toLocaleDateString("en-GB");
    },
  },
};
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.45);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2000;
  overflow: auto;
  overscroll-behavior: contain;
}

.modal-box {
  background: #f1f1f1;
  padding: 24px 28px;
  border-radius: 8px;
  text-align: center;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
  display: flex;
  gap: 12px;
  flex-direction: column;
  width: 400px;
  max-height: 80vh;
  display: flex;
  flex-direction: column;
}

h2 {
  margin: 0;
  text-align: left;
}

.modal-body {
  flex: 1 1 auto;
  overflow-y: auto;
  margin-top: 12px;
  padding-right: 6px;
}

.items {
  display: flex;
  flex-direction: column;
}

.big-group {
  display: flex;
  flex-direction: column;
}

.group-item {
  display: flex;

  gap: 8px;
}

p {
  margin: 8px 4px;
  text-align: left;
}

.title {
  font-weight: bold;
  color: #6007a5;
  min-width: 160px;
  text-align: left;
}

.actions {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin-top: 18px;
}

.cancel {
  background: #ddd;
  color: #1a1a1a;
  border: none;
  padding: 6px 14px;
  border-radius: 6px;
  cursor: pointer;
}

.edit {
  background: #6007a5;
  color: white;
  border: none;
  padding: 6px 14px;
  border-radius: 6px;
  cursor: pointer;
}

@media (max-width: 768px) {
  .modal-overlay {
    padding: 0px 20px;
  }
}
</style>
