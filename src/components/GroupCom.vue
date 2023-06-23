<script setup>
defineProps(['groups', 'activeGroup']);
const emit = defineEmits(['chooseGroup']);

function chooseGroup(group) {
  emit('chooseGroup', group);
}
</script>

<template>
  <div class="row mb-3">
    <div class="btn-group">
      <a
        v-for="item in groups"
        :key="item"
        href=""
        class="btn btn-outline-primary"
        :class="{ active: activeGroup === item }"
        @click.prevent="chooseGroup(item)"
        >{{ item }}</a
      >
    </div>
  </div>
</template>
