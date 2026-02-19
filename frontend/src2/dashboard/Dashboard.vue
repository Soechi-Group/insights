<script setup lang="ts">
import { Breadcrumbs, call } from 'frappe-ui'
import { RefreshCcw } from 'lucide-vue-next'
import { provide, ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { downloadImage } from '../helpers'
import useDashboard from './dashboard'
import DashboardItem from './DashboardItem.vue'
import VueGridLayout from './VueGridLayout.vue'
import { useStorage } from '@vueuse/core'

const props = defineProps<{ name: string }>()
import bg from '@/assets/bg.jpg'

// Panggil API untuk dapatkan nama dashboard
const dashboard_name = await call('insights.api.shared.get_dashboard_name', {
  dashboard_name: props.name,
})

// Background container
const bgStyle = {
  backgroundColor: '#E6F0FF', // bersih, biru lembut
}

// Reactive waktu dan tanggal
const time = ref('')
const date = ref('')

// Dashboard state
const dashboard = useDashboard(dashboard_name)
provide('dashboard', dashboard)
dashboard.refresh()

// Router
const router = useRouter()
function openWorkbook() {
  router.push(`/workbook/${dashboard.doc.workbook}`)
}

// Download dashboard image
const dashboardContainer = ref<HTMLElement | null>(null)
async function downloadDashboardImage() {
  if (!dashboardContainer.value) return
  await downloadImage(dashboardContainer.value, `${dashboard.doc.title}.png`)
}

// Update waktu & tanggal setiap detik
function updateDateTime() {
  const now = new Date()
  time.value = now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
  date.value = now.toLocaleDateString('en-GB', { day: '2-digit', month: 'short', year: 'numeric' })
}

onMounted(() => {
  updateDateTime()
  setInterval(updateDateTime, 1000)
})

// Vertical compact grid setting
const verticalCompact = useStorage('dashboard_vertical_compact', true)
</script>

<template>
  <div class="relative flex h-full w-full overflow-hidden" :style="bgStyle">
    <div ref="dashboardContainer" class="flex-1 overflow-y-auto p-6">
      <!-- LOGO KIRI POJOK -->
      <img
        src="https://digital-sign.soechi.com/Content/Images/logo%20only.png"
        alt="Logo"
        class="absolute top-4 left-6 h-[60px] w-auto object-contain z-50"
      />

      <!-- Judul & Data UnAudited -->
      <div class="flex flex-col items-center mb-5">
        <!-- Dashboard Title -->
        <h1
          class="font-sans font-extrabold tracking-wide"
          :style="{
            fontSize: '36px',
            color: '#002145',         // navy gelap corporate
            letterSpacing: '0.04em',
            lineHeight: '1.1',
            fontFamily: `'Inter', 'Segoe UI', 'Helvetica Neue', sans-serif`,
          }"
        >
          {{ dashboard.doc.title }}
        </h1>

        <!-- Label waktu Data UnAudited (corporate style) -->
        <div
          class="mt-3 px-6 py-2 shadow-sm"
          :style="{
     // putih bersih
            border: '0px',       // border abu medium, profesional
            borderRadius: '0px',                // sharp corners, formal
            maxWidth: 'fit-content',
          }"
        >
          <span
            class="font-semibold"
            :style="{
              fontSize: '18px',               // lebih besar, jelas terbaca
              color: '#1F2937',               // abu gelap profesional
              letterSpacing: '0.02em',
              fontFamily: `'Inter', 'Segoe UI', 'Helvetica Neue', sans-serif`,
            }"
          >
            Data UnAudited : {{ time }} | {{ date }}
          </span>
        </div>
      </div>

      <!-- Dashboard Grid -->
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
