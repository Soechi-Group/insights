<script setup lang="ts">
import { Breadcrumbs, call } from 'frappe-ui'
import { RefreshCcw } from 'lucide-vue-next'
import { provide, ref,onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { downloadImage } from '../helpers'
import useDashboard from './dashboard'
import DashboardItem from './DashboardItem.vue'
import VueGridLayout from './VueGridLayout.vue'
import { useStorage } from '@vueuse/core'
const props = defineProps<{ name: string }>()
import bg from '@/assets/bg.jpg'
const dashboard_name = await call('insights.api.shared.get_dashboard_name', {
	dashboard_name: props.name,
})

const bgStyle = {
  backgroundImage: `url(${bg})`,
  backgroundSize: 'cover',
  backgroundPosition: 'center',
  backgroundRepeat: 'no-repeat'
}
const time = ref("");
const date = ref("");
const dashboard = useDashboard(dashboard_name)
provide('dashboard', dashboard)
dashboard.refresh()

const router = useRouter()
function openWorkbook() {
	router.push(`/workbook/${dashboard.doc.workbook}`)
}

const dashboardContainer = ref<HTMLElement | null>(null)
async function downloadDashboardImage() {
	if (!dashboardContainer.value) return
	await downloadImage(dashboardContainer.value, `${dashboard.doc.title}.png`)
}

function updateDateTime() {
  const now = new Date();
  time.value = now.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
  
  // Format: 10 Oct 2025
  date.value = now.toLocaleDateString("en-GB", { day: "2-digit", month: "short", year: "numeric" });
}

onMounted(() => {
  updateDateTime();
  setInterval(updateDateTime, 1000);
});

const verticalCompact = useStorage('dashboard_vertical_compact', true)
</script>

<template>

	<!-- <header class="flex h-12 items-center justify-between border-b py-2.5 pl-5 pr-2">
		<Breadcrumbs
			:items="[
				{ label: 'Dashboards', route: '/dashboards' },
				{ label: dashboard.doc.title, route: `/dashboards/${dashboard.doc.name}` },
			]"
		/>
		<div class="flex items-center gap-2">
			<Button variant="outline" @click="() => dashboard.refresh(true)" label="Refresh">
				<template #prefix>
					<RefreshCcw class="h-4 w-4 text-black-700" stroke-width="1.5" />
				</template>
			</Button>
			<Dropdown
				placement="left"
				:button="{ icon: 'more-vertical', variant: 'outline' }"
				:options="[
					{
						label: 'Export as PNG 2',
						variant: 'outline',
						icon: 'download',
						onClick: downloadDashboardImage,
					},
					{
						label: 'Open Workbook',
						variant: 'outline',
						icon: 'external-link',
						onClick: openWorkbook,
					},
				]"
			/>
		</div>
	</header> -->




	 <div class="relative flex h-full w-full overflow-hidden":style="bgStyle">
 
    
    <div ref="dashboardContainer" class="flex-1 overflow-y-auto p-4">
      
      <!-- Card Jam & Tanggal di atas grid -->
  <div class="flex items-center gap-4 mb-3 ml-2">

  <!-- Title + Subtitle -->
  <div class="flex flex-col leading-tight">
    <span class="text-lg font-semibold text-gray-800 tracking-wide">
 	<Breadcrumbs
			:items="[
				{ label: 'Dashboards', route: '/dashboards' },
				{ label: dashboard.doc.title, route: `/dashboards/${dashboard.doc.name}` },
			]"
		/>
    </span>
    <span class="text-xs text-gray-600 italic mt-1">
      Data UnAudited
    </span>
  </div>

  <!-- Time Box (no rounded) -->
  <div class="flex items-center justify-center
              bg-white/40 backdrop-blur-md
              shadow-sm
              px-4 h-8 border border-white/30">
    <span class="text-xs font-medium text-gray-700 tracking-wide">
      {{ time }} | {{ date }}
    </span>
  </div>

</div>


      <VueGridLayout
        v-if="dashboard.doc.items.length > 0"
        class="h-fit w-full"
        :cols="20"
        :disabled="true"
        :verticalCompact="verticalCompact"
        :modelValue="dashboard.doc.items.map((item) => item.layout)"
      >
        <template #item="{ index }">
          <DashboardItem :index="index" :item="dashboard.doc.items[index]" />
        </template>
      </VueGridLayout>
    </div>
  </div>
</template>
