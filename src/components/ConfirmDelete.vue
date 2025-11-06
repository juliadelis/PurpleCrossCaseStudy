<template>
  <div class="confirm-overlay">
    <div class="confirm-box">
      <p>
        Are you sure you want to delete <b>{{ itemName }}</b
        >?
      </p>

      <div class="confirm-actions">
        <button class="cancel" @click="$emit('cancel')">Cancel</button>
        <button class="confirm" @click="$emit('confirm')">Delete</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ConfirmDeleteModal",
  props: {
    itemName: {
      type: String,
      required: true,
    },
  },
};
</script>

<style scoped>
.confirm-overlay {
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
}

.confirm-box {
  background: #f1f1f1;
  padding: 24px 28px;
  border-radius: 8px;
  max-width: 320px;
  text-align: center;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
}

.confirm-actions {
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

.confirm {
  background: #d9534f;
  color: white;
  border: none;
  padding: 6px 14px;
  border-radius: 6px;
  cursor: pointer;
}
</style>
