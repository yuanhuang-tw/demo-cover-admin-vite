<script setup>
import { ref, watch } from 'vue';

const props = defineProps(['modelValue']);
const emit = defineEmits(['update:modelValue']);

const searchString = ref(props.modelValue);

const changeSearchString = (e) => {
  emit('update:modelValue', e.target.value.trim());
};

watch(
  () => props.modelValue,
  (newValue) => {
    searchString.value = newValue;
  }
);
</script>

<template>
  <div class="form-floating mb-4">
    <input
      type="text"
      class="form-control"
      placeholder="搜尋暱稱"
      v-model="searchString"
      @input="changeSearchString"
    />
    <label for="search">搜尋暱稱</label>
  </div>
</template>
