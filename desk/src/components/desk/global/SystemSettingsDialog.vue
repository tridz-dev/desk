<template>
	<div>
		<Dialog
			:options="{
				actions: [
					{
						label: 'Finish Setup',
						appearance: 'primary',
						handler: ({ finish }) => {
							updateHelpdeskName();
						},
					},
				],
			}"
			v-model="open"
		>
			<template #body-title>
				<h2 class="mb-2 text-3xl space-y-2 font-bold">
					Welcome, let's make helpdesk your own!
				</h2>
			</template>
			<template #body-content>
				<div class="flex flex-col">
					<div class="flex flex-col space-y-1 mb-2">
						<span class="block mb-2 text-base leading-4 text-gray-700"
							>Choose helpdesk name</span
						>
						<Input
							class="text-xl text-gray-900"
							v-model="name"
							placeholder="eg:frappedesk, helpdesk"
							type="text"
						/>
					</div>
					<div class="space-y-1 my-2">
						<span class="block mb-2 text-base leading-4 text-gray-700"
							>Select default country and language</span
						>
						<div class="flex flex-row gap-4 mb-2">
							<div class="mb-4">
								<span class="mb-3 block text-sm leading-4 text-gray-700"
									>Country</span
								>
								<div class="w-28">
									<Autocomplete
										:value="country != null ? country : selectedCountry"
										:resourceOptions="{
											url: 'frappedesk.extends.client.get_list',
											inputMap: (query) => {
												return {
													doctype: 'Country',
													pluck: 'name',
													filters: [['name', 'like', `%${query}%`]],
												};
											},
											responseMap: (res) => {
												return res.map((d) => {
													return {
														label: d.name,
														value: d.name,
													};
												});
											},
										}"
										@change="
											(item) => {
												country = null;
												selectedCountry = item.value;
											}
										"
									/>
								</div>
							</div>
							<div>
								<span class="mb-3 block text-sm leading-4 text-gray-700"
									>Language</span
								>
								<div class="w-28">
									<Autocomplete
										:value="language != null ? language : selectedLanguage"
										:resourceOptions="{
											url: 'frappedesk.extends.client.get_list',
											inputMap: (query) => {
												return {
													doctype: 'Language',
													pluck: 'name',
													filters: [['name', 'like', `%${query}%`]],
												};
											},
											responseMap: (res) => {
												return res.map((d) => {
													return {
														label: d.name,
														value: d.name,
													};
												});
											},
										}"
										@change="
											(item) => {
												language = null;
												selectedLanguage = item.value;
											}
										"
									/>
								</div>
							</div>
						</div>
					</div>
					<div class="flex flex-col">
						<h2 class="mb-1 text-base text-gray-800">
							We have created a support inbox for you.
						</h2>
						<div class="text-lg font-bold">{{ user }}</div>
						<div class="text-base text-gray-700">
							All email sent to this address will become tickets in Helpdesk.
						</div>
						<Button
							@click="$router.push('/frappedesk/settings/emails/new')"
							class="w-36 mt-4"
							icon-right="external-link"
							>Change Email</Button
						>
					</div>
				</div>
			</template>
		</Dialog>
	</div>
</template>

<script>
import { Dialog, Button } from "frappe-ui";
import Autocomplete from "@/components/global/Autocomplete.vue";
import { ref } from "vue";

export default {
	name: "SystemSettingsDialog",
	components: {
		Dialog,
		Autocomplete,
	},
	computed: {
		user() {
			let doc = this.$resources.getUser.data;
			return doc.email;
		},
	},
	data() {
		return {
			name: "",
		};
	},
	setup() {
		let open = ref(true);
		let country = ref("");
		let currency = ref("");
		let language = ref("");
		let selectedCountry = ref("");
		let selectedCurrency = ref("");
		let selectedLanguage = ref("");

		return {
			open,
			country,
			currency,
			language,
			selectedCountry,
			selectedCurrency,
			selectedLanguage,
		};
	},

	methods: {
		updateSystemSettings(event) {
			return this.$resources.systemSettings.fetch;
		},
		updateHelpdeskName() {
			return this.$resources.setHelpdeskName.submit({ name: this.name });
		},
	},

	resources: {
		systemSettings() {
			return {
				url: "frappedesk.api.settings.get_system_settings",
				onSuccess(res) {
					this.country = res.country;
					this.language = res.language;
				},
				auto: true,
			};
		},

		getUser() {
			return {
				url: "frappedesk.api.settings.last_created_user",
				onSuccess(res) {
					console.log(res);
				},
				auto: true,
			};
		},
	},

	setHelpdeskName() {
		// set helpdesk name in Frappe Desk Settings
		return {
			url: "frappedesk.api.settings.update_helpdesk_name",
			onSuccess: (res) => {
				document.title = `Frappe Desk ${res ? ` | ${res}` : ""}`;
				this.$toast({
					title: "Helpdesk name updated!!",
					icon: "check",
					iconClasses: "text-green-500",
				});
			},
			onError: (err) => {
				this.$toast({
					title: "Something went wrong, updating helpdesk name",
					text: "Please try again later.",
					icon: "x",
					iconClasses: "text-red-500",
				});
			},
			auto: true,
		};
	},
	beforeMount() {
		this.updateSystemSettings();
	},
};
</script>
