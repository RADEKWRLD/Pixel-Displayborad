
# PIXEL-Displayborad！！！🎮

### 项目概述˖ ݁𖥔 ݁˖   👾   ˖ ݁𖥔 ݁˖
这是一个基于 Vue 和 Element Plus 的拖拽式任务看板项目，具有以下特点：
- 随心所欲的编辑事项，时间，紧急程度！
- 现代化的前端开发！
- 随意拖拽你的事项！
- 数据持久化存储！
- 像素化网页设计！
  
项目预览：https://pixel-displayborad.vercel.app/
---
![8b1e9a8d891134675873ce8c90c9162](https://github.com/user-attachments/assets/39e25690-645c-466b-9dcc-438e418b94a2)


### 核心算法及功能🕹️

## 看板拖拽排序🍵⋆｡°🍡°⋆. ࿔*:･
```javascript
// 使用 vuedraggable 实现看板级拖拽
<draggable 
  v-model="boards" 
  @start="throttledStart" 
  @end="throttledEnd"
  :move="checkBoardMove">

// 移动检测逻辑（防止嵌套拖拽冲突）
const checkBoardMove = () => !isDraggingBoard.value;

// 节流处理拖拽状态
const throttledStart = _.throttle(() => {
  isDraggingBoard.value = true;
}, 16);

const throttledEnd = _.throttle(() => {
  isDraggingBoard.value = false;
}, 16);
```

**实现原理**：
- 使用双节流函数控制拖拽状态标识
- 通过move事件检测实现拖拽模式隔离
- 16ms节流间隔确保60FPS流畅度

## 任务项跨看板迁移˙✧˖°📷 ༘ ⋆｡˚
```javascript
// 任务项跨看板拖拽配置
<draggable 
  group="tasks" 
  :disabled="isDraggingBoard"
  @change="throttledItemChange">

// 数据同步处理
const throttledItemChange = _.throttle((evt) => {
  console.log('Item Change:', evt);
  // 自动触发watch持久化存储
}, 16);
```

**核心机制**：
- 共享group实现跨容器拖拽
- disabled属性实现拖拽模式互斥
- 自动数据绑定+节流事件处理

## 数据持久化🐱ྀི ⋆🐾° ᡣ𐭩
```javascript
// 深度监听看板数据变化
watch(boards, (newBoards) => {
  localStorage.setItem('boards', JSON.stringify(newBoards));
}, { deep: true });

// 初始化时加载本地数据
const boards = ref(JSON.parse(localStorage.getItem('boards')) || [
  // 默认数据结构...
]);
```

**特性**：
- 深度监听(deep: true)确保嵌套对象变化可捕获
- JSON序列化/反序列化处理复杂数据结构
- 异常处理：当本地存储为空时初始化默认结构

## 时间处理𐙚 ˚🍰 ⋆｡˚ ᡣ𐭩
```javascript
// 时间范围选择器绑定
<el-date-picker 
  v-model="timeRange" 
  type="datetimerange" 
  value-format="YYYY/MM/DD">

// 时间格式化显示
const formatTime = (time) => {
  return time ? time.split(' ')[0] : 'Unset';
};

// 数据保存处理
startTime: timeRange.value[0] || '',
endTime: timeRange.value[1] || ''
```

**关键技术点**：
- 双向绑定时间范围选择器
- 按需进行日期格式转换
- 空值处理保证数据完整性

## 状态管理🎸⋆⭒˚｡⋆
```javascript
// 紧急程度映射表
const urgencyLabels = {
  low: 'LOW',
  medium: 'MEDIUM',
  high: 'HIGH',
};

// 样式类型映射
const urgencyTagType = {
  low: '',
  medium: 'warning',
  high: 'danger',
};

// 动态class绑定
:class="[`urgency-${item.urgency}`]"
```

**设计亮点**：
- 集中式状态配置管理
- 样式与数据解耦
- 可扩展的状态类型系统

## 性能优化方案༄˖°.🍂.ೃ࿔*:･
```javascript
// 所有事件处理器均采用节流
import _ from 'lodash';

// 统一使用16ms节流间隔
_.throttle(callback, 16)

// 拖拽类名动态控制
:shadow="isDraggingBoard ? 'hover' : 'never'"
```

**优化策略**：
- 高频事件节流处理
- 渲染性能优化（拖拽时阴影控制）
- 按需更新DOM元素
