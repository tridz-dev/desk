<template>
	<div>
		<Datepicker
			range
			v-model="selectedDates"
			@change="onDateChange"
			:enable-time-picker="false"
			format="MMM dd"
		/>
	</div>
</template>

<script>
import Datepicker from "@vuepic/vue-datepicker"

export default {
    components: {
        Datepicker,
    },
    computed: {
        startDate() {
            return new Date(new Date().setDate(new Date().getDate() - 7));
        },
        endDate() {
            return new Date();
        },
        selectedDates() {
            return [this.startDate, this.endDate]
        }
    },
    methods: {
        onDateChange() {
            this.$emit("date-change", this.selectedDates)
			console.log("here")
        },
    },
}
</script>
