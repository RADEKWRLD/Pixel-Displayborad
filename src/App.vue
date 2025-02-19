<template>
  <div class="main">
    <a href="https://github.com/RADEKWRLD" target="_blank" class="github-icon">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="36" height="36">
        <path d="M12 0C5.372 0 0 5.372 0 12c0 5.298 3.438 9.799 8.205 11.387.6.111.826-.261.826-.579 0-.288-.011-1.05-.016-2.065-3.338.727-4.043-1.56-4.043-1.56-.544-1.381-1.33-1.749-1.33-1.749-1.088-.743.083-.728.083-.728 1.204.085 1.835 1.236 1.835 1.236 1.07 1.83 2.809 1.303 3.495.997.107-.776.418-1.303.76-1.604-2.665-.303-5.466-1.335-5.466-5.93 0-1.308.47-2.377 1.243-3.215-.124-.303-.537-1.527.116-3.181 0 0 1.007-.32 3.299 1.235.957-.266 1.978-.4 2.995-.405 1.017.005 2.038.14 2.995.405 2.292-1.554 3.298-1.235 3.298-1.235.654 1.654.24 2.878.116 3.181.773.838 1.243 1.907 1.243 3.215 0 4.605-2.801 5.624-5.475 5.923.429.371.81.965.81 1.951 0 .322.226.697.834.579C20.563 21.799 24 17.298 24 12c0-6.628-5.372-12-12-12z"/>
      </svg>
    </a>
    <h1>DisplayBoard</h1>
    <div class="kanban-container">
      <draggable
        v-model="boards"
        item-key="type"
        class="board-container"
        @start="throttledStart"
        @end="throttledEnd"
        :move="checkBoardMove"
        handle=".board-header"
      >
        <template #item="{ element: board }">
          <el-card class="task-board" :shadow="isDraggingBoard ? 'hover' : 'never'">
            <div class="board-header" :class="board.type">
              <h3>{{ board.title }}</h3>
              <el-button
                type="primary"
                style="border: 2px solid black;"
                :icon="Plus"
                @click="openItemForm(board.type, $event)"
              />
            </div>

            <draggable
              v-model="board.items"
              item-key="id"
              group="tasks"
              class="task-list"
              animation="300"
              :disabled="isDraggingBoard"
              drag-class="dragging-item"
              drop-class="drop-zone"
              :touch-end-event="'mouseup'"
              :touch-start-threshold="15"
              @change="throttledItemChange"
            >
              <template #item="{ element: item }">
                <el-card
                  class="task-item"
                  :class="[`urgency-${item.urgency}`]"
                  @click="openItemForm(board.type, $event, item)"
                >
                  <div class="item-header">
                    <el-tag
                      size="small"
                      :type="urgencyTagType[item.urgency]"
                      effect="plain"
                    >
                      {{ urgencyLabels[item.urgency] }}
                    </el-tag>
                    <el-text
                      style="overflow: hidden; text-overflow: ellipsis;"
                      class="item-title"
                    >
                      {{ item.title }}
                    </el-text>
                    <el-button
                      type="danger"
                      :icon="Delete"
                      circle
                      size="small"
                      @click.stop="deleteItem(item, board)"
                      class="delete-btn"
                    />
                  </div>

                  <div class="item-time">
                    <el-icon>
                      <clock />
                    </el-icon>
                    <span class="startEndTime">
                      {{ formatTime(item.startTime) }} - {{ formatTime(item.endTime) }}
                    </span>
                  </div>
                </el-card>
              </template>
            </draggable>
          </el-card>
        </template>
      </draggable>

      <el-dialog
        v-model="showItemForm"
        :title="currentItem ? 'EDIT' : 'NEW'"
        width="500px"
        :close-on-click-modal="false"
      >
        <el-form :model="formData" label-width="auto">
          <el-form-item label="TITLE" required>
            <el-input
              v-model="formData.title"
              placeholder="Please enter content"
              size="small"
            />
          </el-form-item>

          <el-form-item label="TIME">
            <el-date-picker
              v-model="timeRange"
              type="datetimerange"
              range-separator="to"
              start-placeholder="Start"
              end-placeholder="End"
              value-format="YYYY/MM/DD"
              size="small"
              popper-class="popOutItem"
              teleported=true
            />
          </el-form-item>

          <el-form-item label="URGENCY">
            <el-select v-model="formData.urgency" size="small">
              <el-option
                v-for="(label, key) in urgencyLabels"
                :key="key"
                :label="label"
                :value="key"
              />
            </el-select>
          </el-form-item>
        </el-form>

        <template #footer>
          <el-button size="small" @click="showItemForm = false">
            Cancel
          </el-button>
          <el-button type="primary" size="small" @click="saveItem">
            Confirm
          </el-button>
        </template>
      </el-dialog>
    </div>
  </div>
</template>

<script async setup>
import {  ref, watch } from 'vue';
import { ElCard, ElButton, ElDialog, ElForm, ElFormItem, ElInput, ElSelect, ElOption, ElTag, ElText, ElIcon } from 'element-plus';
import { Clock, Plus, Delete } from '@element-plus/icons-vue';
import draggable from 'vuedraggable';
import _ from 'lodash'; // 引入 lodash




// 紧急程度
const urgencyLabels = {
  low: 'LOW',
  medium: 'MEDIUM',
  high: 'HIGH',
};

const urgencyTagType = {
  low: '',
  medium: 'warning',
  high: 'danger',
};

// 表单相关参数
const showItemForm = ref(false);
const currentBoardType = ref('');
const currentItem = ref(null);
const timeRange = ref([]);
const formData = ref({
  title: '',
  urgency: 'low',
});

// 删除功能实现，删除id
const deleteItem = (item, board) => {
  board.items = board.items.filter(i => i.id !== item.id);
};

// Initial Boards Data
const boards = ref(JSON.parse(localStorage.getItem('boards')) || [
  {
    type: 'willdo',
    title: 'To be processed',
    items: [],
  },
  {
    type: 'doing',
    title: 'In progress',
    items: [],
  },
  {
    type: 'done',
    title: 'Completed',
    items: [],
  },
]);

// 设置时间
const formatTime = (time) => {
  return time ? time.split(' ')[0] : 'Unset';
};

// 打开表单控件函数
const openItemForm = (boardType, event, item = null) => {
  currentBoardType.value = boardType;
  currentItem.value = item;

  if (item) {
    formData.value = { ...item };
    timeRange.value = [item.startTime, item.endTime];
  } else {
    formData.value = { title: '', urgency: 'low' };
    timeRange.value = [];
  }

  showItemForm.value = true;
};

// 保存事件
const saveItem = () => {
  if (!formData.value.title.trim()) return;

  const itemData = {
    ...formData.value,
    startTime: timeRange.value[0] || '',
    endTime: timeRange.value[1] || '',
  };

  const targetBoard = boards.value.find(board => board.type === currentBoardType.value);
  if (currentItem.value) {
    Object.assign(currentItem.value, itemData);
  } else {
    targetBoard.items.push({
      id: Date.now(),
      ...itemData,
    });
  }

  showItemForm.value = false;
};

// 实现拖拽功能
const isDraggingBoard = ref(false);
const checkBoardMove = () => !isDraggingBoard.value;

// 节流函数
const throttledStart = _.throttle(() => {
  isDraggingBoard.value = true;
}, 16); // 限制帧率（约 60 FPS）

const throttledEnd = _.throttle(() => {
  isDraggingBoard.value = false;
}, 16); // 限制帧率（约 60 FPS）

const throttledItemChange = _.throttle((evt) => {
  console.log('Item Change:', evt); 
}, 16); // 限制帧率（约 60 FPS）

// 添加监视器，监视是否变化
watch(boards, (newBoards) => {
  localStorage.setItem('boards', JSON.stringify(newBoards));
}, { deep: true });
</script>




<style scoped>
@font-face {
    font-family: 'Press Start 2P';
    src: url('./font/PressStart2P-Regular.ttf') format('truetype');
    font-weight: normal;
    font-style: normal;
    font-display: swap;
}



.main {
  display: block;
  margin: 4rem auto 0 auto;
  padding: 20px;
  background-image: url("data:image/svg+xml,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20width%3D%228%22%20height%3D%228%22%3E%3Cdefs%3E%3Cpattern%20id%3D%22pixelPattern%22%20patternUnits%3D%22userSpaceOnUse%22%20width%3D%228%22%20height%3D%228%22%3E%3Crect%20x%3D%220%22%20y%3D%220%22%20width%3D%228%22%20height%3D%228%22%20fill%3D%22%232C3E50%22%2F%3E%3Cpolygon%20points%3D%220%2C0%204%2C0%200%2C4%22%20fill%3D%22%2334495E%22%2F%3E%3Cpolygon%20points%3D%224%2C0%208%2C0%208%2C4%22%20fill%3D%22%232C3E50%22%2F%3E%3Cpolygon%20points%3D%220%2C4%204%2C4%200%2C8%22%20fill%3D%22%232C3E50%22%2F%3E%3Cpolygon%20points%3D%224%2C4%208%2C4%208%2C8%22%20fill%3D%22%2334495E%22%2F%3E%3C%2Fpattern%3E%3C%2Fdefs%3E%3Crect%20width%3D%228%22%20height%3D%228%22%20fill%3D%22url(%23pixelPattern)%22%2F%3E%3C%2Fsvg%3E");
  background-size: 32px 32px; 
  background-repeat: repeat;
  animation: backgroundShift 60s linear infinite;
  width: 80%;
  min-height: 80vh;
  max-width: 1200px;
  border: 3px solid black;
  border-radius: 8px;
  overflow: hidden;
}

@keyframes backgroundShift {
  0% {
    background-position: 0 0;
  }

  100% {
    background-position: 100% 100%;
  }
}


span{
  font-size: 6px;
}

h1 {
  font-size: 40px;
  text-align: center;
  margin: 10px 0;
  color: #FFB200;
  font-family: "Press Start 2P", Arial, sans-serif;
}

.board-container {
  display: flex;
  flex-wrap: wrap;
  gap: 10vh;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.task-board {
  width: 280px;
height: 60vh;
  background-color: #fff;
  border: 2px solid #000;
  border-radius: 8px;
  box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
}

.board-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px;
  border-bottom: 2px solid #000;
  background-color: #f0f0f0;
}

.board-header h3 {
  margin: 0;
  font-size: 11px;
  color: #000;
  font-family: "Press Start 2P", Arial, sans-serif;
}

.task-list {
  padding: 10px;
  min-height: 280px;
  max-height: 70vh;
  overflow-y: auto;
  border-top: 2px solid #000;
}

.task-item {
  margin-bottom: 10px;
  padding: 10px;
  background-color: #f9f9f9;
  border: 1px solid #ccc;
  border-radius: 2px;
  cursor: pointer;
}

.item-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 8px;
}

.delete-btn {
  margin-left: auto;
  background-color: #ff4d4d;
  border: 1px solid #000;
  border-radius: 2px;
  transition: background-color 0.2s;
}

.delete-btn:hover {
  background-color: #ff0000;
}

.item-title {
  font-size: 12px;
  color: #000;
  font-family: "Press Start 2P", Arial, sans-serif;
  white-space: normal;
  overflow: hidden;
  text-overflow: clip;
}

.item-time {
  display: flex;
  align-items: center;
  gap: 4px;
  color: #666;
  font-size: 12px;
  font-family: "Press Start 2P", Arial, sans-serif;
}


.urgency-high {
  border-left: 8px solid #ff0000;
}

.urgency-medium {
  border-left: 8px solid #ffa500;
}

.urgency-low {
  border-left: 8px solid #008000;
}

:deep(.el-dialog) {
  border-radius: 8px;
  width: 50%;
  overflow: hidden;
  border: 5px solid black;
  background-color:#f0f0f0;
}

:deep(.el-dialog) {
  margin-bottom: 12px;
}




@media screen and (max-width: 768px) {
  .main {
    width: 90%;
    margin: 2rem auto;
  }

  :deep(.el-dialog) {
    width: 90% !important;
  }

  .task-board {
    width: 100%;
    min-height: 360px;
  }

  .item-header {
    flex-direction: column;
  }
}
</style>