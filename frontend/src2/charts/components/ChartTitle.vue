<script setup>
const props = defineProps({
	title: { type: String, required: false },
})
</script>

<template>
	<div id="Chart_Title" class="px-4 py-2 font-medium leading-6 text-lg text-black-8100">
		{{ title }}
	</div>
</template>
