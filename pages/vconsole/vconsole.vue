<template>
	<view>
		<Calendar
			@changeDate="changeDateMethods"
		></Calendar>
	</view>
</template>

<script>
	import Calendar from '../calendar/main.vue'
	export default {
		components:{
			Calendar
		},
		data() {
			return {
				
			}
		},
		methods: {
			changeDateMethods(row) {
				console.log(row)
			}
		}
	}
</script>

<style>

</style>
