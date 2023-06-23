<script setup>
import { ref, watch } from 'vue';

const props = defineProps(['modelValue', 'totalPageNumber']);
const emit = defineEmits(['update:modelValue']);

const page = ref(props.modelValue);

const changePage = (e) => {
  emit('update:modelValue', e.target.value);
};

const previousPage = () => {
  emit('update:modelValue', --page.value);
};

const nextPage = () => {
  emit('update:modelValue', ++page.value);
};

const firstPage = () => {
  emit('update:modelValue', 1);
};

const lastPage = () => {
  emit('update:modelValue', props.totalPageNumber);
};

watch(
  () => props.modelValue,
  (newValue) => {
    page.value = parseInt(newValue);
  }
);
</script>

<template>
  <div class="d-flex justify-content-end gap-2 mb-3">
    <button class="btn btn-primary" @click="firstPage" :disabled="page === 1">
      <i class="fa fa-angle-double-left" aria-hidden="true"></i> 第一頁
    </button>

    <button class="btn btn-primary" @click="previousPage" :disabled="page === 1">
      <i class="fa fa-angle-left" aria-hidden="true"></i> 上一頁
    </button>

    <select class="form-select w-25" v-model="page" @change="changePage">
      <option :value="n" v-for="n in totalPageNumber" :key="n">Page {{ n }}</option>
    </select>

    <button class="btn btn-primary" @click="nextPage" :disabled="page === totalPageNumber">
      下一頁 <i class="fa fa-angle-right" aria-hidden="true"></i>
    </button>

    <button class="btn btn-primary" @click="lastPage" :disabled="page === totalPageNumber">
      最後一頁 <i class="fa fa-angle-double-right" aria-hidden="true"></i>
    </button>
  </div>
</template>
