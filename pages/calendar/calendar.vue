<template>
  <div class="weekend-row date-row">
    <div class="date-col"
         :class="getActivedClass(item, index)"
         @click="activedCurrentDay(item, index)"
         v-for="(item, index) in dateData"
         :key="`week${index}`">
      <div class='circle-svg'>
        <svg xmlns="http://www.w3.org/200/svg"
             height="100%"
             width="100%">
          <circle class="circle-detail"
                  cx="25"
                  cy="25"
                  :r="circlePai"
                  fill="none"
                  :stroke-width="item.percent === 0?0:2"
                  stroke-linecap="round"
                  :stroke="item.percent !== 113 && item.dateStr !== activeDay?'#DB3030': '#2D8DFF'"
                  :stroke-dasharray="`${item.percent},10000`"></circle>
        </svg>
      </div>
      <span :class="item.month !== activeMonth?'not-current-month': ''">
        {{item.day}}
      </span>
    </div>
  </div>
</template>

<script>
export default {
  name: "expandCalendar",
  data() {
    return {
      currentDate: "",
      activeDay: "",
      circlePai: 18
    };
  },
  props: {
    clickDay: {
      type: String,
      default: ''
    },
    activeMonth: {
      type: Number,
      default: 1
    },
    dateData: {
      type: Array,
      default: () => []
    }
  },
  watch: {
    clickDay: {
      handler(val) {
        console.log(val, this.activeDay)
        if(val && val !== '') {
          this.activeDay = val
        }
      },
      immediate: true
    }
  },
  mounted() {},
  methods: {
    getActivedClass(row, index) {
      return row.dateStr === this.activeDay ? "date-col-active" : "";
    },
    activedCurrentDay(row, index) {
      this.$emit("changeDate", { data: row });
    }
  }
};
</script>

<style scoped lang="scss">
	@import './calendar.scss'
</style>
