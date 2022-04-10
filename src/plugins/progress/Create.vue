<script setup>

defineProps({
  space: Object,
  preview: Boolean,
  modelValue: Object
});

const emit = defineEmits(['update']);
const update = form => {
  emit('update', { key: 'progress', form });
};
</script>

<template>
  <ProgressConfig> TEST</ProgressConfig>
</template>
