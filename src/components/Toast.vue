<template>
  <transition name="fade">
    <div v-if="visible" class="toast" :class="type">{{ message }}</div>
  </transition>
</template>

<script>
export default {
  name: "Toast",
  data() {
    return {
      visible: false,
      message: "",
      type: "success" | "error",
    };
  },
  methods: {
    show(msg, type = "success", duration = 3500) {
      this.message = msg;
      this.type = type;
      this.visible = true;
      setTimeout(() => {
        this.visible = false;
      }, duration);
    },
  },
};
</script>

<style scoped>
.toast {
  position: fixed;
  bottom: 24px;
  right: 24px;
  padding: 12px 18px;
  border-radius: 6px;
  color: white;
  font-size: 14px;
  z-index: 2000;
  box-shadow: 0 4px 14px rgba(0, 0, 0, 0.25);
}

.toast.success {
  background: #158929;
}
.toast.error {
  background: #d9376e;
}

.fase-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
