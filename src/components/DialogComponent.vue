<template>
  <v-dialog v-model="showDialog" max-width="500" persistent>
    <v-card>
      <v-toolbar>
        <v-toolbar-title>{{ title }}</v-toolbar-title>
        <v-btn icon="mdi-close" @click="closeDialog"></v-btn>
      </v-toolbar>
      <v-card-text>
        檢核項目{{ checkObject }}
      </v-card-text>

      <v-card-text>
        <v-checkbox v-for="checkBox in checkBoxs" :key="checkBox.id" v-model="checkBox.checked" :label="checkBox.label"
          :rules="baseCheckRules">
        </v-checkbox>
      </v-card-text>
      <v-card-text>
        <v-btn 
          outlined 
          v-for="formBtn in formButtons" 
          :key="formBtn.id" 
          @click="openFormDialog(formBtn.formName, formBtn.time)" 
          width="100%" 
          class="mt-1">
          {{ formBtn.label }}
            <template v-slot:append>
              <v-icon :color="formDones[`${formBtn.formName}-${formBtn.time}`] ? 'success' : 'grey'">mdi-check-circle</v-icon>
            </template>
        </v-btn>
      </v-card-text>
      <v-card-text>
        {{ reminder }}
      </v-card-text>


      <v-container class="d-flex justify-space-around">
        <v-btn rounded="xl" color="deep-orange-lighten-4" class="text-grey" :readonly="!allDone" @click="closeDialog" >檢核確認</v-btn>
        <v-btn v-if="props.additionalForm && !formRequired" rounded="xl" outlined @click="openSingleAdditionalForm">填寫紀錄表</v-btn>
      </v-container>
    </v-card>
  </v-dialog>
  <!-- <form-dialog-component
        v-model:showForm="showFormDialog"
        :form-config="formConfig"
        :fillFormTime="new Date().toLocaleString('zh-TW', { timeZone: 'Asia/Taipei' })"
        @submitForm="saveAdditionalForm"
    /> -->
  <form-dialog-manager 
    v-model:showForm="showFormDialog" 
    :form="form" :time="time"
    :form-config="formConfig"
    @currentformDone="updateFormDone($event,time)"
  />
</template>

<script setup>
import { ref, computed, watch, inject, onMounted } from 'vue';
import FormDialogComponent from './FormDialogComponent.vue';
import FormDialogManager from './FormDialogManager.vue';

const modifyPass = inject('modifyJobPass');

const showFormDialog = ref(false);
const formConfig = ref(null);
const allDone = ref(false);
const formDones = ref({});
const form = ref('')

const props = defineProps({
  show: Boolean,
  title: {
    type: String,
    default: 'Dialog Title'
  },
  cardText: {
    type: String,
    default: 'placeholder text for card content'
  },
  subtitle: {
    type: String,
    default: ''
  },
  checkObject: {
    type: String,
    default: 'default check object'
  },
  checkBoxs: {
    type: Array,
    default: () => [{}]
  },
  formButtons: {
    type: Array,
    default: () => [{}]
  },
  formName: {
    type: Array,
    default: () => ['']
  },
  formRequired: {
    type: Boolean,
    default: false
  },
  time: {
    type: String,
    default: ''
  },
  additionalForm: {
    type: Object,
    default: null
  },
  reminder: {
    type: String,
    default: ''
  }
});

const emit = defineEmits(['update:show', 'addtionalFormSubmit']);

const baseCheckRules = [
  (value) => !!value || '請確認'
];

onMounted(() => {
  allDone.value = false;
});

const showDialog = computed({
  get: () => props.show,
  set: (value) => emit('update:show', value)
});

watch(() => props.checkBoxs, (newVal) => {
  // allDone.value = formsAllDone && allChecks;
}, { immediate: false });

function closeDialog() {
  dbugs();
  modifyPass(props.formName, props.time, allDone.value);
  showDialog.value = false;
}

function saveAdditionalForm(submittedData) {
  emit('addtionalFormSubmit', submittedData);
}

function resetDialogDone() {
  allDone.value = false; // Reset completion state
}

watch(() => props.show, (newVal) => {
  if (newVal === true) {
    resetDialogDone();
  }
});

function dbugs() {
  console.log(`-------------\n%O\n-------------`, 
    {    
      formRequired: props.formRequired,
      formNames: props.formName,
      allCheckboxesChecked: props.checkBoxs.every(cb => cb.checked),
      formDones: { ...formDones.value },
      formDonesLength: Object.values(formDones.value).length,
      allDone: allDone.value
    }
  );
}

function updateFormDone(formName,time) {
  formDones.value[`${formName}-${time}`] = true;
  dbugs();
}

function openSingleAdditionalForm() {
  formConfig.value = props.additionalForm;
  form.value = props.formName[0];
  showFormDialog.value = true;
}

function openFormDialog(formName, time) {
  formConfig.value = props.additionalForm[formName] || null;
  form.value = formName;
  showFormDialog.value = true;
}


</script>
<style scoped>
</style>