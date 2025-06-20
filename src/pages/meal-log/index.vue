<template>
  <div class="meal-log-page" @scroll.passive="onScroll" ref="mealLogPage" v-resize="onResize">
    <v-divider ref="scrollIntoViewElement" :thickness="0"></v-divider>
    <div class="d-flex justify-space-between align-center pa-2 mx-2">
      <h2>供膳管理日誌</h2>
      <nav>
        <ul class="breadcrumb">
          <span v-for="(breadcrumb, index) in breadcrumbs" :key=index>
            <router-link :to="breadcrumb.path">{{ breadcrumb.label }}</router-link>
            <span v-if="index < breadcrumbs.length - 1"> / </span>
          </span>
        </ul>
      </nav>
    </div>
    <div class="d-flex justify-space-between align-center pa-2 mx-2">
      <div class="current-time">
        <v-icon>mdi-clock-outline</v-icon>
        {{ currentTime }}
      </div>
      <div class="branch-selection">
        <v-select v-model="branch" hide-details :items="branches" label="" class="ml-4" />
      </div>
    </div>
    <v-card class="mx-3">
      <div class="d-flex justify-space-between align-center pa-2">
        <div>日誌檢核表</div>
        <v-chip color="purple" text-color="white">{{ completedCount }}/{{ sectionsToDone }}</v-chip>
      </div>
      <v-list>
        <v-list-group v-for="(job, index) in jobs" :key="job.section" :value="job.section">
          <template v-slot:activator="{ props }">
            <v-list-item v-bind="props" :title="job.time"
              :subtitle="`${job.title} (${job.items.filter(i => i.checked).length}/${job.items.length})`"
              class="font-weight-bold">
              <template v-slot:prepend>
                <v-icon color="amber" :size="listStatusIconSize">mdi-circle</v-icon>
              </template>
            </v-list-item>
          </template>
          <div class="task-timeline">
            <v-list-item v-for="item in job.items" :key="`${index}-${item.id}`" :value="`${index}-${item.id}-val`"
              lines="one" class="task-item">
              <template v-slot:prepend>
                <div class="timeline-indicator"></div>
              </template>
              <v-list-item-title class="pl-3 d-flex align-center meal-list-item-title"
                :style="{ 'background-color': (item.id % 2 ? '#fff' : '#e0f7fa'), width: '100%' }">{{ item.title
                }}</v-list-item-title>
              <div class="d-flex align-center" :style="{ 'background-color': (item.id % 2 ? '#fff' : '#e0f7fa') }">
                <v-btn :key="`button-${index}-${item.id}`" hide-details variant="text" icon class="ma-0 pa-0"
                  :disabled="!canComplete(item, job.section)" @click="handleButtonClick(item, job.section)">
                  <!--TODO canComplete前一個section沒完成不能做下一個section -->
                  <v-icon
                    :color="state[`button-${job.section}-${item.id}`] && item.checked ? state[`button-${job.section}-${item.id}`] : 'grey'">
                    mdi-check-circle</v-icon>
                </v-btn>
                <v-btn variant="text" icon @click="openRemarkDialog(item)">
                  <v-icon>mdi-dots-horizontal-circle</v-icon>
                </v-btn>
              </div>
            </v-list-item>
          </div>
        </v-list-group>
      </v-list>
    </v-card>
    <v-card class="associated-forms mt-5 mx-3">
      <div class="d-flex justify-space-between align-center pa-2">
        <div>關聯表單</div>
      </div>
      <div class="pa-4">
        <v-row class="associate-forms-row">
          <v-col cols="5" md="2" v-for="(form, formName) in forms" :key="formName" class="associate-form-col"
            :style="{'border-right': formName !== Object.keys(forms).at(-1) ? '2px dotted' : '' }">
            <div class="associate-form" variant="text" @click="openReadOnlyForm(formName)">
              {{ form.additionalForm.title || formName }}</div>
          </v-col>
        </v-row>
      </div>
    </v-card>
    <!-- TODO 這個方法有bug width會是v-app的width 不知道v-bottom-naciagtion的calc(100%-N)是怎麼抓N的達成只顯示在v-container裡面 -->
    <header class="sticky-bottom-action">
      <v-btn variant="flat" class="border-md ml-10" color="white" @click="tempSaveDraft" rounded>暫存草稿</v-btn>
      <v-btn variant="flat" class="mr-10" :disabled="!isRootCompleted" @click="signDraft" rounded>簽章送出</v-btn>
      <v-btn variant="text" v-show="showBackToTop" icon :ripple="false" @click="backToTop" class="back-to-top">
        <v-icon class="back-to-top-icon">mdi-chevron-up-circle</v-icon>
      </v-btn>
    </header>
  </div>
  <!--TODO 只傳入time即可 -->
  <DialogComponent v-model:show="dialogState.show" :title="dialogState.title" :check-object="dialogState.checkObject"
    :check-boxs="dialogState.checkBoxs" :form-buttons="dialogState.formButtons"
    :additional-form="dialogState.additionalForm" :form-name="dialogState.formName"
    :form-required="dialogState.formRequired" :time="dialogState.time" :reminder="dialogState.reminder"
    @addtionalFormSubmit="saveDialogAndAdditionalForm" />
  <remarks-dialog v-model:showRemarks="showRemarksDialog" :item="jobRemarks" />
  <v-dialog v-model="showSignatureDialog" max-width="500px">
    <v-card class="pa-2">
      <v-card-title>簽章送出</v-card-title>
      <v-divider></v-divider>
      <v-card-text>
        <div>表單：供膳管理日誌</div>
        <div>時間戳記：{{ signatureTimestamp }}</div>
        <div>時段：<span class="fillTime">{{ shift === 'morning' ? '早班' :
          shift === 'evening' ? '晚班' : 'ERROR' }}</span></div>
      </v-card-text>
      <v-card-text>
        <canvas ref="signatureCanvas" width="450" height="200" style="border: 1px solid #ccc;"></canvas>
      </v-card-text>
      <v-card-text class="default-signature-checkbox">
        <v-checkbox v-slot:append hide-details @update:modelValue="useDefaultSignature" />
        <div>使用個人預設簽名檔</div>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="primary" @click="saveSignatureAndSend">簽章送出</v-btn>
        <v-btn color="error" @click="cancelSendSignature">取消</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script setup>
import { ref, computed, reactive, watch, onMounted, provide } from 'vue';
import DialogComponent from '../../components/TCHG/DialogComponent.vue';
import RemarksDialog from '../../components/TCHG/RemarksDialog.vue';
import { testMorningTCHGJobs } from "../../data/testTCHGMealJobs";
import { testTCHGForms } from '../../data/testTCHGForms';
import { defaultSignature } from '../../data/testSignature';
import { useRouter } from 'vue-router'

const router = useRouter();

const jobs = ref(testMorningTCHGJobs);
const forms = ref(testTCHGForms);
const signature = ref(defaultSignature);
const shift = ref('morning');
const showBackToTop = ref(false);
const readForm = ref('');
const showReadonlyForm = ref(false);
const listStatusIcon = ref([]);
const listStatusIconSize = ref(10);
const scrollIntoViewElement = ref();

const breadcrumbs = [
  { label: '首頁', path: '/' },
  { label: '任務看板', path: '/TCHGmealSys' },
  { label: '供膳管理日誌', path: '/meal-log' },
]
const branch = ref('院本部');
const branches = ['院本部', '中興', '仁愛', '其他分院'];
const currentTime = ref(new Date().toLocaleString('zh-TW', { timeZone: 'Asia/Taipei' }));

const dialogState = reactive({
  show: false,
  title: '',
  checkObject: '',
  checkBoxs: [],
  formButtons: [],
  formName: [],
  formRequired: false,
  time: '',
  additionalForm: null,
  reminder: ''
});

const showRemarksDialog = ref(false);
const state = ref({});
const mealLogPage = ref(null);

watch(() =>
  jobs.value.filter(job => Array.isArray(job.items))
    .map(job => job.items.map(item => item.checked))
  ,
  () => updateCompletedCount(),
  { deep: true }
);

onMounted(() => {
  setInterval(() => {
    let localDateString = new Date().toLocaleString('zh-TW', { timeZone: 'Asia/Taipei' });
    currentTime.value = localDateString;
  }, 1000);

  jobs.value.forEach((job, jobIndex) => {
    job.items.forEach(item => {
      state.value[`button-${job.section}-${item.id}`] = "success";
    });
  });
  updateCompletedCount();
});


const signatureTimestamp = computed(() => {
  return new Date().toLocaleString('zh-TW', { timeZone: 'Asia/Taipei' });
});

const sectionsToDone = computed(() => {
  return jobs.value.length;
});

const isRootCompleted = computed(() => {
  return completedCount.value === sectionsToDone.value ? true : false;
});

provide('getAddiForm', (formName) => {
  return forms.value[formName]?.additionalForm || [];
});

provide('modifyJobPass', (formName, time, pass) => {
  const job = jobs.value.find(j =>
    j.items.some(item => {
      return (item.forms &&
        item.forms.formName === formName &&
        (!time || item.forms.time === time))
    }
    )
  );
  if (job) {
    const section = job.section;
    const item = job.items.find(item =>
      item.forms && item.forms.formName === formName && (!time || item.forms.time === time)
    );
    if (item && item.forms) {
      item.forms.passed = pass;
      const key = `button-${section}-${item.id}`;
      state.value[key] = pass ? 'success ' : 'error';
      item.checked = true;
      updateCompletedCount();
    }
  }
});


// TODO 改成api呼叫
provide('updateAddiForm', (formName, newData) => {
  const form = forms.value[formName];

  if (form && form.additionalForm) {
    form.additionalForm = { ...newData };
  }
});

function onResize() {
  listStatusIconSize.value = Math.max(window.innerWidth / 100, 15);
}

function backToMealSys() {
  router.push(`/TCHGmealSys`);
}

function onScroll(event) {
  const container = event.target;
  const scrollTop = container.scrollTop;
  const containerHeight = container.clientHeight;
  const scrollHeight = container.scrollHeight;

  showBackToTop.value = scrollTop > (scrollHeight - containerHeight) * 0.7;
}
function backToTop() {
  scrollIntoViewElement.value.$el.scrollIntoView({ behavior: 'smooth' });
}

function canComplete(item, jobSection) {
  const currentJob = jobs.value.find(job => job.section === jobSection);
  if (!currentJob) return false;

  const currentItemIndex = currentJob.items.findIndex(i => i.id === item.id);

  if (currentItemIndex === 0) return true;

  return currentJob.items
    .slice(0, currentItemIndex)
    .every(prevItem => prevItem.checked);
}

const completedCount = ref(0);

const jobRemarks = ref(null);

function openRemarkDialog(item) {
  jobRemarks.value = item || '';
  showRemarksDialog.value = true;
}

function updateCompletedCount() {
  let completed = 0;
  jobs.value.forEach(job => {
    completed += (job.items.filter(item => item.checked).length / job.items.length) | 0;
  });
  completedCount.value = completed;
}

function handleButtonClick(item, jobSection) {
  let currentForms = item.forms;
  let itemId = item.id;

  if (currentForms && Object.keys(currentForms).length > 0 && !item.checked) {
    const formsCollection = {};
    if (currentForms.formName) {
      currentForms.formName.forEach(form => {
        formsCollection[form] =
          forms.value[form]?.additionalForm || [];
      });
    }
    Object.assign(dialogState, {
      show: true,
      title: currentForms.title || "檢核確認",
      checkObject: currentForms.checkObject || "",
      checkBoxs: currentForms.checkBoxs || [],
      formButtons: currentForms.formButtons || [],
      formName: currentForms.formName || [],
      formRequired: currentForms.formRequired || false,
      time: currentForms.time || "",
      additionalForm: formsCollection,
      reminder: currentForms.reminder || ""
    });
  }
  const key = `button-${jobSection}-${itemId}`;
  if (!currentForms) {
    item.checked = item.checked ? false : true;

  } else {
    item.checked = false;
  }
  updateCompletedCount();
}

function saveDialogAndAdditionalForm(allData) {

}

function tempSaveDraft() {
  console.log("暫存草稿功能尚未實作");
}

function signDraft() {
  openSignatureDialog();
}

function openReadOnlyForm(formName) {
  readForm.value = formName;
  showReadonlyForm.value = true;
}


// Signature handling
const signatureCanvas = ref(null);
const signatureCtx = ref(null);
const isDrawing = ref(false);
const showSignatureDialog = ref(false);
let lastX = 0;
let lastY = 0;

function openSignatureDialog() {
  showSignatureDialog.value = true;
  setTimeout(() => {
    initSignatureCanvas();
  }, 100);
}

function initSignatureCanvas() {
  const canvas = signatureCanvas.value;
  if (!canvas) return;

  signatureCtx.value = canvas.getContext('2d');
  const ctx = signatureCtx.value;
  ctx.lineWidth = 2;
  ctx.lineCap = 'round';
  ctx.strokeStyle = '#000';

  canvas.addEventListener('mousedown', startDrawing);
  canvas.addEventListener('mousemove', draw);
  canvas.addEventListener('mouseup', stopDrawing);
  canvas.addEventListener('mouseout', stopDrawing);

  // Touch support
  canvas.addEventListener('touchstart', handleTouch);
  canvas.addEventListener('touchmove', handleTouchMove);
  canvas.addEventListener('touchend', stopDrawing);
}

function startDrawing(e) {
  isDrawing.value = true;
  [lastX, lastY] = [e.offsetX, e.offsetY];
}

function draw(e) {
  if (!isDrawing.value) return;
  const ctx = signatureCtx.value;
  ctx.beginPath();
  ctx.moveTo(lastX, lastY);
  ctx.lineTo(e.offsetX, e.offsetY);
  ctx.stroke();
  [lastX, lastY] = [e.offsetX, e.offsetY];
}

function handleTouch(e) {
  e.preventDefault();
  const touch = e.touches[0];
  const rect = signatureCanvas.value.getBoundingClientRect();
  const offsetX = touch.clientX - rect.left;
  const offsetY = touch.clientY - rect.top;

  isDrawing.value = true;
  [lastX, lastY] = [offsetX, offsetY];
}

function handleTouchMove(e) {
  e.preventDefault();
  if (!isDrawing.value) return;

  const touch = e.touches[0];
  const rect = signatureCanvas.value.getBoundingClientRect();
  const offsetX = touch.clientX - rect.left;
  const offsetY = touch.clientY - rect.top;

  const ctx = signatureCtx.value;
  ctx.beginPath();
  ctx.moveTo(lastX, lastY);
  ctx.lineTo(offsetX, offsetY);
  ctx.stroke();
  [lastX, lastY] = [offsetX, offsetY];
}

function stopDrawing() {
  isDrawing.value = false;
}

function useDefaultSignature() {
  if (signature.value) {
    const ctx = signatureCtx.value;
    const img = new Image();
    img.src = signature.value;
    img.onload = () => {
      ctx.clearRect(0, 0, signatureCanvas.value.width, signatureCanvas.value.height);
      ctx.drawImage(img, 0, 0, signatureCanvas.value.width, signatureCanvas.value.height);
    };
  }
}

function cancelSendSignature() {
  showSignatureDialog.value = false;
  const ctx = signatureCtx.value;
  ctx.clearRect(0, 0, signatureCanvas.value.width, signatureCanvas.value.height);
}

function saveSignatureAndSend() {
  signature.value = signatureCanvas.value.toDataURL('image/png');
  sessionStorage.setItem('signature', signature.value);
  sessionStorage.setItem('signatureTimestamp', signatureTimestamp.value);
  sessionStorage.setItem('signatureShift', shift.value);
  sessionStorage.setItem('signatureBranch', branch.value);
  sessionStorage.setItem('mealJobs', JSON.stringify(jobs.value));
  backToMealSys();
  showSignatureDialog.value = false;
}
</script>

<style scoped>
@import url('../../css/TCHG_mealLog.css');
@import url('../../css/TCHG_mealLog_MobileS.css') (max-width: 320px) and (orientation: portrait);
</style>