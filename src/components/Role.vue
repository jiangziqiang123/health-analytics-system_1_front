<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { ElTag } from 'element-plus';

// 假设角色信息存储在localStorage，实际可根据项目调整
const userRole = ref('');
const userName = ref('');

onMounted(() => {
  userRole.value = localStorage.getItem('userRole') || '普通用户';
  userName.value = localStorage.getItem('userName') || '用户';
});
</script>

<template>
  <div class="role-welcome">
    <span class="role-text">欢迎，{{ userName }}！</span>
    <el-tag v-if="userRole !== '普通用户'" type="success" effect="dark" class="role-tag">{{ userRole }}</el-tag>
    <el-tag v-else class="role-tag tech-user" effect="dark">
      <span class="tech-user-inner">
        <svg class="user-icon" viewBox="0 0 24 24" width="16" height="16" fill="none" xmlns="http://www.w3.org/2000/svg">
          <circle cx="12" cy="8" r="4" stroke="#00eaff" stroke-width="1.5" fill="#101c2c"/>
          <path d="M4 20c0-4 4-6 8-6s8 2 8 6" stroke="#00eaff" stroke-width="1.5" fill="#101c2c"/>
        </svg>
        <span class="tech-text">普通用户</span>
      </span>
    </el-tag>
  </div>
</template>

<style scoped>
.role-welcome {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 16px;
  color: #00eaff;
}
.role-text {
  user-select: none;
  cursor: default;
}
.role-tag {
  margin-left: 6px;
}
.tech-user {
  background: #101c2c;
  border: 1px solid #00eaff;
  color: #00eaff;
  display: flex;
  align-items: center;
  font-weight: 500;
  letter-spacing: 1px;
  box-shadow: none;
  padding: 0 6px;
}
.tech-user-inner {
  display: flex;
  align-items: center;
}
.user-icon {
  margin-right: 4px;
  vertical-align: middle;
  display: inline-block;
}
.tech-text {
  font-family: 'Consolas', 'monospace';
  font-size: 15px;
  user-select: none;
  cursor: default;
  display: inline-block;
}
</style>