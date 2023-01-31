<template>
	<div>
		<Dialog
			:options="{ title: 'Add New Ticket Type', size: 'sm' }"
			v-model="open"
		>
			<template #body-content>
				<div class="space-y-4">
					<div class="space-y-1">
						<Input
							label="Name"
							type="text"
							placeholder="Bug"
							v-model="title"
						/>
					</div>
					<div class="w-full space-y-1">
						<div>
							<span class="block text-sm leading-4 text-gray-700">
								Priority
							</span>
						</div>
						<Autocomplete
							:value="selectedPriority"
							@change="
								(item) => {
									if (!item) {
										return
									}
									selectedPriority = item.value
								}
							"
							:resourceOptions="{
								method: 'frappe.client.get_list',
								inputMap: (query) => {
									return {
										doctype: 'Ticket Priority',
										pluck: 'name',
										filters: [
											['name', 'like', `%${query}%`],
										],
									}
								},
								responseMap: (res) => {
									return res.map((d) => {
										return {
											label: d.name,
											value: d.name,
										}
									})
								},
							}"
						/>
					</div>
					<div class="flex flex-col">
						<div class="mb-2 block text-sm leading-4 text-gray-700">
							Description
						</div>
						<TextEditor
							:class="'bg-gray-100'"
							ref="textEditor"
							:editor-class="'min-h-[4rem] overflow-y-auto max-h-[73vh] px-3 max-w-full'"
							:content="description"
							:starterkit-options="{
								heading: { levels: [2, 3, 4, 5, 6] },
							}"
							@change="
								(val) => {
									description = val
								}
							"
						>
						</TextEditor>
					</div>

					<div class="flex float-right space-x-2">
						<Button
							appearance="primary"
							@click="
								() => {
									addTicketType()
									close()
									this.$router.go()
								}
							"
							class="mr-auto"
							>Add</Button
						>
					</div>
				</div>
			</template>
		</Dialog>
	</div>
</template>

<script>
import { Input, Dialog, ErrorMessage, TextEditor } from "frappe-ui"
import { computed, ref, inject } from "vue"
import Autocomplete from "@/components/global/Autocomplete.vue"
export default {
	name: "newTicketTypeDialog",
	props: {
		modelValue: {
			type: Boolean,
			required: true,
		},
		title: {
			type: String,
		},
	},
	setup(props, { emit }) {
		let open = computed({
			get: () => props.modelValue,
			set: (val) => {
				emit("update:modelValue", val)
				if (!val) {
					emit("close")
				}
			},
		})
		const selectedPriority = ref("")
		return {
			open,
			selectedPriority,
		}
	},
	components: {
		Dialog,
		Input,
		Autocomplete,
		TextEditor,
	},
	data(props) {
		return {
			name: props.title,
			priority: "",
			description: "",
		}
	},
	methods: {
		addTicketType() {
			const inputParams = {
				name: this.title,
				priority: this.selectedPriority,
				description: this.description,
			}
			this.$resources.newTicketType.submit({
				doc: {
					doctype: "Ticket Type",
					...inputParams,
				},
			})
		},
		close() {
			this.name = ""
			this.priority = ""
			this.description = ""
			this.$emit("close")
		},
	},
	resources: {
		newTicketType() {
			return {
				method: "frappe.client.insert",
				onSuccess: (doc) => {
					this.$router.go()
				},
			}
		},
	},
}
</script>
