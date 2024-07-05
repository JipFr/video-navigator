<script setup lang="ts">
defineProps<{
	src: string;
}>();
</script>

<template>
	<img :src="src" alt="Segment" class="w-full h-full object-cover" />
</template>
