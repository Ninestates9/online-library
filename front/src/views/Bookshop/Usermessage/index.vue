<template>
  <div class="rightmain">
    <div class="righttop">
      <h3 class="topinfo">用户消息</h3>
    </div>
    <div class="messages">
      <div v-for="message in messages" :key="message.Uno" class="message" >
      <el-descriptions :border="true"  column="1"  size="large">
            <el-descriptions-item label="账号" label-width="50px">{{ message.Uno }}</el-descriptions-item>
            <el-descriptions-item label="消息">{{ message.message }}</el-descriptions-item>
        </el-descriptions>
        <button class="delete-btn" @click="deleteUserMessage(message.Uno)"><el-icon><Delete /></el-icon></button>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted } from 'vue';
import { mainStore } from '../../../store/index.ts';
import axios from 'axios';
import { ElMessage } from 'element-plus';
import VantaBirds from '../../VantaBirds.vue';
const store = mainStore();
const messages = ref([
  {
    Uno: 'U12345',
    message: '嘻嘻',
  },
]);
const getUserMessage = () => {
  axios({
    method: 'post',
    url: `${store.ip}/api/getMsg`,
    headers: { 'Content-Type': 'multipart/form-data' },
  })
    .then((response) => {
      const responseData = response.data;
      if (responseData.ret === 1) {
        ElMessage({
          message: responseData.msg,
          type: 'error',
          duration: 5 * 1000,
          grouping: true,
        });
      } else {
        messages.value = responseData.messageSet;
      }
    })
}

const deleteUserMessage = (Uno) => {
  let formData = new FormData();
  formData.append('Uno', Uno);
  axios({
    method: 'post',
    url: `${store.ip}/api/deleteMsg`,
    data: formData,
    headers: { 'Content-Type': 'multipart/form-data' },
  })
    .then((response) => {
      const responseData = response.data;
      if (responseData.ret === 1) {
        ElMessage({
          message: responseData.msg,
          type: 'error',
          duration: 5 * 1000,
          grouping: true,
        });
      } else {
        // alert(responseData.ret);
        getUserMessage();
      }
    })
}

onMounted(() => {
  getUserMessage(); // 组件挂载时获取用户信息
});
</script>

<style scoped>
.rightmain {
  position: relative;
  display: flex;
  height: 98.5vh;
  width: 100%;
  overflow: hidden;
  display: grid;
  grid-template-rows: 15% 85%;
}

.righttop {
  position: relative;
  overflow: hidden;
  align-items: center;
  width: 100%;
  height: 100%;
}

.topinfo {
  position: relative;
  width: 100%;
  height: 100%;
  align-items: center;
  font-size: 50px;
}

.messages {
  padding: 20px;
  font-size: 18px;
  text-align: left;
  /* 确保文本左对齐 */
  overflow-y: auto
}

.message {
  display: flex;
  display: grid;
  grid-template-columns: 92% 8%;
}

.user-message {
  margin-bottom: 20px;
}

button {
    margin-left: 10px;
    background-color: rgb(76, 213, 255);
    color:aliceblue;
}

.delete-btn {
  width: 60px; 
  display: flex;
  align-items: center;
  border-top-left-radius: 0px;
  border-bottom-left-radius: 0px;
}

::v-deep .el-collapse-item__header:focus, .el-collapse-item__header:focus-visible {
    outline: none;
    background-color: #42b5fd3b;
}

::v-deep .el-collapse {
  background-color: #90d3fc3b;
}

::v-deep .el-collapse-item__header {
    margin-left: 0;
    background-color: #cbe9fd38;
}

:deep(.el-descriptions__label.el-descriptions__cell.is-bordered-label) {
    background-color: #90d3fc3b;
}

:deep(.el-collapse-item__wrap) {
  background-color: #cbe9fd38;
}

:deep(.el-collapse-item__content) {
  background-color: #a4dafd67;
}

::v-deep .el-descriptions__body {
    background-color: #cbe9fd38;
}

:deep(.el-table__body) {
  --el-table-tr-bg-color: #a4dafd67;
}

:deep(.el-table__header) {
  --el-table-header-bg-color: #64c3ff58;
}
</style>