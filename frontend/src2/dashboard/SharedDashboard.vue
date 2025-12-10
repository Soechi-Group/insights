<script setup lang="ts">
import { provide, ref,onMounted } from 'vue'
import useDashboard from './dashboard'
import DashboardItem from './DashboardItem.vue'
import VueGridLayout from './VueGridLayout.vue'
import { call } from 'frappe-ui'
import bg from '@/assets/bg.jpg'
const bgStyle = {
  backgroundImage: `url(${bg})`,
  backgroundSize: 'cover',
  backgroundPosition: 'center',
  backgroundRepeat: 'no-repeat'
}
const props = defineProps<{ dashboard_name: string }>()
const dashboard_name = await call('insights.api.shared.get_dashboard_name', {
	dashboard_name: props.dashboard_name,
})
const time = ref("");
const date = ref("");

function updateDateTime() {
  const now = new Date();
  time.value = now.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
  date.value = now.toLocaleDateString("en-GB", { day: "2-digit", month: "short", year: "numeric" });
}

onMounted(() => {
  updateDateTime();
  setInterval(updateDateTime, 1000);
});

const dashboard = useDashboard(dashboard_name)
provide('dashboard', dashboard)
const dashboardContainer = ref<HTMLElement | null>(null)
</script>
	

<template>
<div class="relative flex h-full w-full overflow-hidden":style="bgStyle">
<div ref="dashboardContainer" class="flex-1 overflow-y-auto p-4">
  <div class="flex items-center gap-4 mb-3 ml-2">
  <div class="flex flex-col leading-tight">
    <span class="text-lg font-semibold text-gray-800 tracking-wide">
 	<Breadcrumbs
			:items="[
				{
				 label: dashboard.doc.title, route: `${dashboard.doc.name}` },
			]"
		/>
    </span>
    <span class="text-xs text-gray-600 italic mt-1">
      Data UnAudited
    </span>
  </div>
  <div class="flex items-center justify-center
              bg-white/40 backdrop-blur-md
              shadow-sm
              px-4 h-8 border border-white/30">
    <span class="text-xs font-medium text-gray-700 tracking-wide">
        {{ time }} | {{ date }}
    </span>
  </div>
</div>
	<div class="relative flex h-full w-full overflow-hidden">
		<div class="flex-1 overflow-y-auto p-4">
			<VueGridLayout
				v-if="dashboard.doc.items.length > 0"
				class="h-fit w-full"
				:cols="20"
				:disabled="true"
				:modelValue="dashboard.doc.items.map((item) => item.layout)"
			>
				<template #item="{ index }">
					<DashboardItem :index="index" :item="dashboard.doc.items[index]" />
				</template>
			</VueGridLayout>
		</div>
	</div>
	 </div>
	 </div>
</template>

	