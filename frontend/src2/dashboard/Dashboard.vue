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

<div class="relative flex h-full w-full overflow-hidden":style="bgStyle">
<div ref="dashboardContainer" class="flex-1 overflow-y-auto p-4">
  <!-- LOGO KIRI POJOK -->
<img
  src="https://digital-sign.soechi.com/Content/Images/logo%20only.png"
  alt="Logo"
  class="
    absolute
    top-4 left-6
    h-[50px]
    w-auto
    object-contain
    z-50
  "
/>




<div class="flex justify-center mb-3">
<div class="flex flex-col items-center leading-tight">
<h1
  class="
    text-[30px]
    font-bold
    text-blue-800
    tracking-tight
    z-50
  "
>
  {{ dashboard.doc.title }}
</h1>


  <div
    class="mt-1 flex items-center justify-center
           bg-white/40 backdrop-blur-md
           shadow-sm
           px-4 h-8 border border-white/30"
  >
    <span class="text-xs font-bold text-blue-700 tracking-wide">
      <span class="text-blue-600">Data UnAudited :</span>
      &nbsp;{{ time }} | {{ date }}
    </span>
  </div>
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
