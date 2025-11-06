<template>
  <nav class="navbar">
    <div class="logo">Purple Cross Ltd</div>

    <div class="profile">
      <img src="../assets/images/user-profile.svg" alt="User" class="avatar" />
      <span class="username">John Doe</span>
    </div>
  </nav>
</template>

<script>
export default {
  name: "Navbar",
};
</script>

<style scoped>
.navbar {
  width: 100%;
  background-color: #6007a5;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 16px;
  color: #f1f1f1;
  z-index: 10;
}

/* --- Logo --- */
.logo {
  font-size: 20px;
  font-weight: bold;
}

/* --- Profile Section --- */

.profile {
  display: flex;
  align-items: center;
  gap: 8px;
}

.avatar {
  width: 32px;
  height: 32px;
  border-radius: 50%;
}

.username {
  font-weight: 500;
}
</style>
