<template>
  <!-- App Header -->
  <v-app-bar class="app-bar">
    <v-app-bar-nav-icon color="white" variant="text"></v-app-bar-nav-icon>
    <v-app-bar-title class="text-white ">供膳管理系統</v-app-bar-title>
    <v-spacer></v-spacer>
    <v-menu>
      <template v-slot:activator="{ props }">
        <v-btn v-bind="props" variant="text" color="white" class="text-caption">
          <v-avatar size="24" class="mr-2">
            <v-img src="https://images.icon-icons.com/2265/PNG/512/doctor_avatar_medical_icon_140443.png"
              alt="User avatar"></v-img>
          </v-avatar>
          午班營養師 <v-icon>mdi-chevron-down</v-icon>
        </v-btn>
      </template>
      <v-list>
        <v-list-item value="profile">個人資料</v-list-item>
        <v-list-item value="logout">登出</v-list-item>
      </v-list>
    </v-menu>
  </v-app-bar>

  <!-- Main Content -->
  <div class="bg-grey-lighten-3 d-flex flex-column pa-4">
    <!-- Page Header -->
    <div class="d-flex align-center mb-4">
      <h1 class="text-h5 font-weight-bold">任務看板</h1>
      <v-spacer></v-spacer>
      <div class="text-caption text-grey">首頁 / 任務看板</div>
    </div>
    <v-divider color="warning" class="my-1"></v-divider>

    <!-- Time and Location Bar -->
    <v-card class="mb-4" variant="flat">
      <v-card-text class="d-flex flex-row align-center pa-2">
        <v-icon class="mr-2">mdi-clock-outline</v-icon>
        <span>{{ currentDateTime }}</span>
        <v-select v-model="branch" :items="branches" class="ml-4 brach-selection" variant="outlined" hide-details />
      </v-card-text>
    </v-card>

    <!-- Task Cards Carousel -->
    <v-carousel :show-arrows="false" height="420" hide-delimiters interval="3000" hide-delimiter-background
      class="homepage-carousel">
      <!-- <v-carousel-item
            v-for="(task, i) in tasks"
            :key="i"
          >
            <v-container class="d-flex justify-center align-center ">
              <v-card width="400" outlined class="d-flex flex-column align-center">
                <v-card-title class="font-weight-bold">{{ task.title }}</v-card-title>
                <v-divider color="warning" :thickness="7"></v-divider>
                <v-card-text class="py-2">
                  <div class="text-body-2">{{ task.description }}</div>
                </v-card-text>
                <v-divider color="warning" :thickness="7"></v-divider>
                  <v-btn
                    color="#6A36DE"
                    text
                    class="font-weight-bold"
                    @click="openTaskDetail(task.id)"
                  >
                    立即開始
                  </v-btn>
              </v-card>
            </v-container>
          </v-carousel-item> -->


      <v-carousel-item>
        <div class="d-flex justify-center align-center">
          <v-card outlined class="d-flex flex-column align-center carousel-card" rounded color="purple-lighten-4">
            <div class="icon-background mt-2">
              <v-icon size="30" class="text-white">
                mdi-calendar-month-outline
              </v-icon>
            </div>
            <v-card-title class="task-name">供膳管理日誌</v-card-title>
            <v-divider :thickness="7"></v-divider>
            <v-card-text class="py-2">
              <ul class="task-list">
                <li><v-icon class="yellow-dot">mdi-check-circle</v-icon>2/8 廚點作業</li>
                <li><v-icon class="white-dot">mdi-check-circle</v-icon> 0/1 生鮮食材驗收及登錄</li>
                <li><v-icon class="white-dot">mdi-check-circle</v-icon> 0/6 清潔衛具設備衛生</li>
                <li><v-icon class="white-dot">mdi-check-circle</v-icon> 0/5 營日誌各事項作業</li>
                <li><v-icon class="white-dot">mdi-check-circle</v-icon> 0/5 烹煮督導作業</li>
                <li><v-icon class="white-dot">mdi-check-circle</v-icon> 0/4 午餐督導作業</li>
                <li><v-icon class="white-dot">mdi-check-circle</v-icon> 0/5 補餐、午餐後清潔作業</li>
              </ul>
            </v-card-text>
            <v-divider :thickness="7"></v-divider>
            <v-btn variant="plain" class="font-weight-bold mb-3 start-button" @click="openTaskDetail('meal-log')">
              立即開始
            </v-btn>
          </v-card>
        </div>
      </v-carousel-item>

      <!-- Second Carousel Item - 每日衛生檢查紀錄 -->
      <v-carousel-item>
        <div class="d-flex justify-center align-center">
          <v-card outlined class="d-flex flex-column align-center carousel-card" color="yellow-lighten-4">
            <div class="icon-background mt-2">
              <v-icon size="30" class="text-white">
                mdi-calendar-month-outline
              </v-icon>
            </div>
            <v-card-title class="task-name">每日衛生檢查紀錄</v-card-title>
            <v-divider :thickness="7"></v-divider>
            <v-card-text class="py-2">
              <ul class="task-list">
                <li><v-icon class="white-dot">mdi-check-circle</v-icon> 0/5 廚房衛生檢查</li>
                <li><v-icon class="white-dot">mdi-check-circle</v-icon> 0/3 冰箱溫度確認</li>
                <li><v-icon class="white-dot">mdi-check-circle</v-icon> 0/4 料理區域清潔檢查</li>
                <li><v-icon class="white-dot">mdi-check-circle</v-icon> 0/2 食材保存區確認</li>
              </ul>
            </v-card-text>
            <v-divider :thickness="7"></v-divider>
            <v-btn variant="plain" class="font-weight-bold mb-3 start-button" @click="openTaskDetail('meal-log')">
              立即開始
            </v-btn>
          </v-card>
        </div>
      </v-carousel-item>

      <!-- Third Carousel Item - 食材驗收查驗檢查紀錄 -->
      <v-carousel-item>
        <div class="d-flex justify-center align-center">
          <v-card outlined class="d-flex flex-column align-center carousel-card" color="pink-lighten-4">
            <div class="icon-background mt-2">
              <v-icon size="30" class="text-white">
                mdi-calendar-month-outline
              </v-icon>
            </div>
            <v-card-title class="task-name">食材驗收查驗檢查紀錄</v-card-title>
            <v-divider :thickness="7"></v-divider>
            <v-card-text class="py-2">
              <ul class="task-list">
                <li><v-icon class="white-dot">mdi-check-circle</v-icon> 0/4 蔬果類驗收</li>
                <li><v-icon class="white-dot">mdi-check-circle</v-icon> 0/5 肉品類驗收</li>
                <li><v-icon class="white-dot">mdi-check-circle</v-icon> 0/3 乾貨類驗收</li>
                <li><v-icon class="white-dot">mdi-check-circle</v-icon> 0/2 調味品類驗收</li>
              </ul>
            </v-card-text>
            <v-divider :thickness="7"></v-divider>
            <v-btn variant="plain" class="font-weight-bold mb-3 start-button" @click="openTaskDetail('meal-log')">
              立即開始
            </v-btn>
          </v-card>
        </div>
      </v-carousel-item>

    </v-carousel>
    <v-card class="my-4" variant="flat">
      <v-card-text>
        <div class="d-flex justify-space-between mb-1">
          <div class="text-subtitle-1 font-weight-bold">今日工作表</div>
        </div>
        <div class="text-h5 font-weight-bold d-flex flex-row align-end">
          {{ totalCompletion }}<span class="text-h6 text-grey-darken-1">%</span>
          <div class="text-caption text-grey-darken-1 mb-1">/ 任務完成度</div>
        </div>

        <!-- Timeline -->
        <v-sheet class="timeline-container mt-6">
          <!-- Time markers -->
          <div class="timeline-header">
            <span v-for="(time, index) in timeMarkers" :key="index" class="time-marker">{{ time }}</span>
          </div>

          <!-- Timeline content with task blocks -->
          <div class="timeline-content">
            <!-- Task blocks -->
            <div v-for="(task, index) in taskBlocks" :key="`task-${index}`" class="task-block" :class="[task.status]"
              :style="{
                width: `${task.width}px`,
                left: `${task.left}px`,
                top: `${task.top}px`
              }">
              <div class="task-completion">{{ task.completion }}<span class="small">%</span></div>
            </div>

            <!-- Current time indicator -->
            <div class="current-time-indicator" :style="{ left: `${currentTimePosition}px` }">
              <div class="time-bubble">{{ formattedCurrentTime }}</div>
              <div class="time-line"></div>
            </div>
          </div>
        </v-sheet>
      </v-card-text>
    </v-card>
    <v-btn rounded-sm class="mb-2">
      <v-icon>mdi-plus</v-icon>
      待簽核
      <template v-slot:append>
        <div color="red">
          <span class="text-caption">3</span>
        </div>
      </template>
    </v-btn>
    <v-btn rounded-sm class="mb-2">
      <v-icon>mdi-plus</v-icon>
      已簽核
      <template v-slot:append>
        <div color="green">
          <span class="text-caption">3</span>
        </div>
      </template>
    </v-btn>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { useRouter } from 'vue-router'

const branch = ref('院本部');
const branches = ref(['院本部', '中興', '仁愛', '其他分院']);
const timeMarkers = ref([
  '7:00', '7:30', '8:00', '8:30', '9:00', '9:30', '10:00', '10:30',
  '11:00', '11:30', '12:00', '12:30', '13:00', '13:30', '14:00'
]);
// Total completion percentage
const totalCompletion = ref(2);
// Calculate time to pixels
const PIXELS_PER_HOUR = 100; // How many pixels per hour
const START_HOUR = 7; // Our timeline starts at 7:00
// Current time 
const currentDateTime = ref(new Date().toLocaleString());

const currentTime = ref(new Date());
const formattedCurrentTime = computed(() => {
  const hours = currentTime.value.getHours().toString().padStart(2, '0');
  const minutes = currentTime.value.getMinutes().toString().padStart(2, '0');
  return `${hours}:${minutes}`;
});

// Position of the current time indicator
const currentTimePosition = computed(() => {
  const hours = currentTime.value.getHours();
  const minutes = currentTime.value.getMinutes();
  const hoursDecimal = hours + minutes / 60;
  return (hoursDecimal - START_HOUR) * PIXELS_PER_HOUR;
});


const tasks = ref([
  {
    id: 1,
    title: '廚點作業',
    startTime: '07:00',
    endTime: '07:30',
    completion: 2,
    needToComplete: 8,
    status: 'active',
    row: 0
  },
  {
    id: 2,
    title: '生鮮食材驗收及登錄',
    startTime: '07:30',
    endTime: '08:00',
    completion: 0,
    needToComplete: 1,
    status: 'pending',
    row: 0
  },
  {
    id: 3,
    title: '清潔衛具設備衛生',
    startTime: '08:30',
    endTime: '09:15',
    completion: 0,
    status: 'pending',
    row: 0
  },
  {
    id: 4,
    title: '營日誌各事項作業',
    startTime: '09:00',
    endTime: '10:00',
    completion: 0,
    status: 'pending',
    row: 1
  },
  {
    id: 5,
    title: '烹煮督導作業',
    startTime: '10:00',
    endTime: '10:30',
    completion: 0,
    status: 'pending',
    row: 0
  },
  {
    id: 6,
    title: '午餐督導作業',
    startTime: '11:00',
    endTime: '11:45',
    completion: 0,
    status: 'pending',
    row: 0
  },
  {
    id: 7,
    title: '補餐、午餐後清潔作業',
    startTime: '12:00',
    endTime: '13:30',
    completion: 0,
    status: 'pending',
    row: 0
  },
  {
    id: 8,
    title: '下午業務',
    startTime: '08:00',
    endTime: '09:30',
    completion: 0,
    status: 'pending',
    row: 2
  },
  {
    id: 9,
    title: '晚餐準備',
    startTime: '13:00',
    endTime: '14:00',
    completion: 0,
    status: 'pending',
    row: 3
  },
]);

const router = useRouter();

function openTaskDetail(taskName) {
  router.push(`/${taskName}`)
}

// Calculate task block positions and dimensions
const taskBlocks = computed(() => {
  return tasks.value.map(task => {
    // Parse times and calculate positions
    const [startHour, startMin] = task.startTime.split(':').map(Number);
    const [endHour, endMin] = task.endTime.split(':').map(Number);

    const startDecimal = startHour + startMin / 60;
    const endDecimal = endHour + endMin / 60;

    // Calculate left position and width in pixels
    const left = (startDecimal - START_HOUR) * PIXELS_PER_HOUR;
    const width = (endDecimal - startDecimal) * PIXELS_PER_HOUR;

    // Vertical positioning - 30px per row, with some margin
    const top = task.row * 35;

    // Status class
    const statusClass = task.completion > 0 ? 'task-block-active' : 'task-block-pending';

    return {
      ...task,
      left,
      width,
      top,
      status: task.id === 1 ? 'task-block-purple' :
        task.id === 2 ? 'task-block-red' :
          'task-block-grey'
    };
  });
});

// Update current time every minute
let timeInterval;
onMounted(() => {
  timeInterval = setInterval(() => {
    currentTime.value = new Date();
  }, 60000);
});

onUnmounted(() => {
  clearInterval(timeInterval);
});
</script>

<style scoped>
.app-bar {
  background: #442a9b;
  background: linear-gradient(90deg, rgba(68, 42, 155, 1) 0%, rgba(83, 191, 237, 1) 100%);
}

.homepage-carousel {
  max-width: 50vw;
  margin: auto;
}

.brach-selection{
  scale: 0.7;
}

.icon-background{
  background-color: rgba(250, 250, 250, 0.153);
  width: 70px;
  height: 70px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.task-name{
  font-size: 18px;
}

.task-list {
  list-style: none;
  padding: 0;
  text-align: left;
}

.task-list li {
  font-size: 11px;
  margin-bottom: 8px;
  display: flex;
  align-items: center;
}

.start-button{
  width: 100% !important;
  height: 11%;
  font-size: 25px;
  height: max-content;
}

.yellow-dot,
.white-dot {
  display: inline-block;
  margin-right: 8px;
}

.yellow-dot {
  color: #FFC107;
}

.white-dot {
  color: gray;
}

.timeline-container {
  position: relative;
  padding-top: 20px;
  overflow-x: auto;
  min-height: 180px;
}

.timeline-header {
  display: flex;
  justify-content: space-between;
  padding-bottom: 10px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.12);
}

.time-marker {
  font-size: 12px;
  color: rgba(0, 0, 0, 0.6);
  flex: 1;
  text-align: center;
}

.timeline-content {
  position: relative;
  margin-top: 20px;
  height: 140px;
}

.task-block {
  position: absolute;
  height: 26px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.task-completion {
  font-size: 14px;
  color: white;
}

.small {
  font-size: 10px;
  margin-left: 1px;
}

.task-block-purple {
  background: linear-gradient(90deg, #8465CE 50%, #D1C6EC 100%);
}

.task-block-red {
  background: linear-gradient(90deg, #FF7373 50%, #FFB1B1 100%);
}

.task-block-grey {
  background-color: #DDDDDD;
}

.current-time-indicator {
  position: absolute;
  top: -20px;
  bottom: 0;
  width: 2px;
  z-index: 10;
}

.time-bubble {
  position: absolute;
  top: 10px;
  left: -28px;
  background-color: white;
  border-radius: 16px;
  padding: 4px 12px;
  font-size: 14px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  z-index: 11;
}

.time-line {
  position: absolute;
  top: 45px;
  height: 100%;
  width: 2px;
  background: repeating-linear-gradient(to bottom, #FF7373 0, #FF7373 5px, transparent 5px, transparent 10px);
  z-index: 10;
}

.carousel-card {
  width: 100% !important;
  height: 380px !important;
  margin: auto;
  display: flex;
  border-radius: 10%;
  flex-direction: column;
}

.carousel-card .v-card-text {
  flex: 1;
  overflow-y: auto;
}
</style>