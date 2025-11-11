<template>
	<div
		class="flex h-full flex-col justify-between transition-all duration-300 ease-in-out border-r border-white/10"
		:class="isSidebarCollapsed ? 'w-14' : 'w-60'"
		style="background: linear-gradient(180deg, rgba(30,60,114,0.95) 0%, rgba(42,82,152,0.95) 100%);
			   backdrop-filter: blur(8px);"
	>
		<!-- Bagian Atas -->
		<div class="flex flex-col overflow-hidden">
			<UserDropdown
				class="p-3 text-white"
				:isCollapsed="isSidebarCollapsed"
			/>
			<div class="flex flex-col overflow-y-auto px-1">
				<template v-for="link in links" :key="link.label">
					<SidebarLink
						v-if="!link.hidden"
						class="my-1 rounded-md px-3 py-2 text-sm text-white/80 
							   hover:text-white hover:bg-white/10 transition-all duration-200 ease-in-out"
						:icon="link.icon"
						:label="link.label"
						:to="link.to"
						:isCollapsed="isSidebarCollapsed"
						@click="link.onClick"
					/>
				</template>
			</div>
		</div>

		<!-- Bagian Bawah -->
		<div class="pb-3 px-2">
			<TrialBanner
				v-if="is_fc_site"
				:is-sidebar-collapsed="isSidebarCollapsed"
			/>
			<SidebarLink
				:label="isSidebarCollapsed ? 'Expand' : 'Collapse'"
				:isCollapsed="isSidebarCollapsed"
				@click="isSidebarCollapsed = !isSidebarCollapsed"
				class="flex items-center justify-center rounded-md bg-white/10 py-2 text-sm text-white/90 hover:bg-white/20 transition"
			>
				<template #icon>
					<span class="grid h-5 w-6 flex-shrink-0 place-items-center">
						<PanelRightOpen
							class="h-5 w-5 text-white transition-transform duration-300 ease-in-out"
							:class="{ 'rotate-180': isSidebarCollapsed }"
							stroke-width="1.5"
						/>
					</span>
				</template>
			</SidebarLink>
		</div>
	</div>

	<Settings v-model="showSettingsDialog" />
</template>


<script setup lang="ts">
import { useStorage } from '@vueuse/core'
import {
	Book,
	Database,
	DatabaseZap,
	LayoutGrid,
	PanelRightOpen,
	SettingsIcon,
} from 'lucide-vue-next'
import { computed, ref } from 'vue'
import useSettings from '../settings/settings'
import Settings from '../settings/Settings.vue'
import SidebarLink from './SidebarLink.vue'
import UserDropdown from './UserDropdown.vue'
import { TrialBanner } from 'frappe-ui/frappe'

const isSidebarCollapsed = useStorage('insights:sidebarCollapsed', false)
const showSettingsDialog = ref(false)

const settings = useSettings()
const is_fc_site = window.is_fc_site

const links = ref([
	{
		label: 'Dashboards',
		icon: LayoutGrid,
		to: 'DashboardList',
	},
	{
		label: 'Workbooks',
		icon: Book,
		to: 'WorkbookList',
	},
	{
		label: 'Data Sources',
		icon: Database,
		to: 'DataSourceList',
	},
	{
		label: 'Data Store',
		icon: DatabaseZap,
		to: 'DataStoreList',
		hidden: computed(() => !settings.doc.enable_data_store),
	},
	{
		label: 'Settings',
		icon: SettingsIcon,
		to: 'Settings',
		onClick: () => (showSettingsDialog.value = true),
	},
])
</script>
