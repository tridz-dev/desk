<template>
	<div class="flex flex-col px-4">
		<ListManager
			ref="listManager"
			:options="{
				cache: ['Agents', 'Desk'],
				doctype: 'Agent',
				fields: ['user as email', 'user.full_name as full_name'],
				limit: 20,
				
			}"
		>
			<template #body="{ manager }">
				<ListViewer
					:options="{
						name: 'Agent',
						base: '12',
						listTitle: 'Agents',
						presetFilters: true,
						fields: {
							full_name: { label: 'Name', width: '4' },
							email: {
								label: 'Email',
								width: '2',
							},
						},
					}"
					class="text-base h-[calc(100vh-9.5rem)] pt-4"
					@add-item="
						() => {
							showNewAgentDialog = true
						}
					"
				>
					<template #field-full_name="{ row }">
						<router-link
							:to="{
								path: `/frappedesk/settings/agents/${row.email}`,
							}"
						>
							{{ `${row.full_name}` }}
						</router-link>
					</template>
					<template #bulk-actions="{ selectedItems }">
						<div class="flex flex-row space-x-2">
							<Button
								@click="
									() => {
										$resources.deleteTeam
											.submit({
												doctype: 'Agent Group',
												name: Object.keys(
													selectedItems
												),
											})
											.then(() => {
												manager.unselect()
												manager.reload()
											})
									}
								"
								>Delete</Button
							>
						</div>
					</template>
				</ListViewer>
			</template>
		</ListManager>
		<AddNewAgentsDialog
			:show="showNewAgentDialog"
			@close="
				() => {
					showNewAgentDialog = false
					$refs.listManager.manager.reload()
					$router.go() // TODO: this is a hack
				}
			"
		/>
	</div>
</template>
<script>
import { inject, ref } from "vue"
import AgentList from "@/components/desk/settings/agents/AgentList.vue"
import ListManager from "@/components/global/ListManager.vue"
import AddNewAgentsDialog from "@/components/desk/global/AddNewAgentsDialog.vue"
import ListViewer from "@/components/global/ListViewer.vue"

export default {
	name: "Agents",
	components: {
		AgentList,
		ListManager,
		AddNewAgentsDialog,
		ListViewer,
	},
	setup() {
		const viewportWidth = inject("viewportWidth")
		const showNewAgentDialog = ref(false)
		return {
			viewportWidth,
			showNewAgentDialog,
		}
	},
	mounted() {
		this.$event.emit("set-selected-setting", "Agents")
		this.$event.emit("show-top-panel-actions-settings", "Agents")

		this.$event.on("show-new-agent-dialog", () => {
			this.showNewAgentDialog = true
		})
		this.$event.on("delete-selected-agents", () => {
			this.$resources.bulk_delete_agents.submit({
				items: Object.keys(
					this.$refs.listManager.manager.selectedItems
				),
				doctype: "Agent",
			})
		})
	},
	unmounted() {
		this.$event.off("show-new-agent-dialog")
		this.$event.off("delete-selected-agents")
	},
	resources: {
		bulk_delete_agents() {
			return {
				method: "frappedesk.api.doc.delete_items",
				onSuccess: () => {
					this.$router.go()
					// this.$refs.listManager.manager.reload()
					// this.$toast({
					// 	title: 'Agents deleted',
					// 	customIcon: 'circle-check',
					// 	appearance: 'success'
					// })
					// this.$event.emit('show-top-panel-actions-settings', 'Agents')
				},
				onError: (err) => {
					this.$refs.listManager.manager.reload()
					this.$toast({
						title: "Error while deleting agents",
						text: err,
						customIcon: "circle-check",
						appearance: "success",
					})
					this.$event.emit(
						"show-top-panel-actions-settings",
						"Agents"
					)
				},
			}
		},
	},
}
</script>
