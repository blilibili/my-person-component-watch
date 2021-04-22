<template>
  <view class="calendar-container">
	<view class="calendar-bg-month" v-show="isExpand">
		{{currentDate.month < 10?`0${currentDate.month}`: currentDate.month}}
	</view>  
    <view class="calendar-date">
      {{currentDate.year}}年{{currentDate.month}}月
    </view>
    <view class="weekend-row">
      <view class="weekend-col"
           v-for="(item, index) in weekendList"
           :key="`week${index}`">
        {{item}}
      </view>
    </view>

    <!--未展开-->
    <swiper class="my-swipe my-not-expand-swipe"
           ref="notExpandRef"
		   :disable-touch="true"
           v-show="!isExpand"
		   :duration="0"
		   :current="currentWeekIndex"
           :loop="false"
           :show-indicators="false"
           @change="changeWeek"
    >
      <template v-for="(parent, key) in lineWeekDataList">
        <swiper-item
            :key="`swipe${key}`">
          <ExpandCalendar :dateData="parent"
                          :click-day="activeDay"
                          :activeMonth="currentMonth+1"
                          ref="notExpandCalendarRef"
                          @changeDate="changeDateMethods"></ExpandCalendar>

        </swiper-item>
      </template>
    </swiper>

    <!--展开-->
    <swiper class="my-swipe my-expand-swipe"
           ref="expandRef"
           :loop="false"
		   :duration="0"
		   :current="currentMonth"
           v-show="isExpand"
           :show-indicators="false"
           @change="changeMonth"
    >
      <template v-for="(parent, key) in allDateList">
        <swiper-item
            :key="`swipe${key}`">
          <ExpandCalendar :dateData="parent.data"
                          :click-day="activeDay"
                          :activeMonth="currentMonth+1"
                          ref="expandCalendarRef"
                          @changeDate="changeDateMethods"></ExpandCalendar>

        </swiper-item>
      </template>

    </swiper>

	<view class="icon-footer-container"
	         @click="updateExpandStatus">
	      <text v-if="!isExpand">展开</text>
		  <text v-if="isExpand">收起</text>
	   
	    </view>

	  </view>
</template>


<script>
	import CalendarObj from './date.js'
	import ExpandCalendar from './calendar.vue'
	export default {
		components:{
			ExpandCalendar
		},
		data() {
			return {
				activeObj: {},
				cacheWeekIndex: 0,
				cacheMonthIndex: 0,
				currentWeekIndex: 0,
				lineWeekDataList: [],
				currentMonth: 0,
				currentDate: "",
				activeDay: "",
				currentDateList: [],
				allDateList: [],
				lineCurrentDateList: [],
				weekendList: ["周日", "周一", "周二", "周三", "周四", "周五", "周六"],
				circlePai: 18,
				isExpand: false
			}
		},
		props: {
			format: {
				type: Object,
				default: 'YYYY-MM-DD'
			}
		},
		created() {
			const opt = {
			prev: "上个月",
			  current: "当前",
			  next: "下个月"
			};
			let count = 0;
			const cacheDateLen = 12; // 前后缓存十个
			let weekIndex = 0;
			const calendar = CalendarObj({
			  // 获取的每个日历数据的最小单元
			  // 这里可以自定义想要的数据

			  // callback 参数为最终的日历josn 数据
			  callback: calendarJson => {
				// console.log("json", calendarJson);
				this.currentDateList = calendarJson;
				const circleLength = Math.floor(2 * Math.PI * this.circlePai);
				this.currentDateList.forEach((item, key) => {
				  count++;
				  // 初始化percent
				  item.weekIndex = weekIndex;
				  if(item.week === 6) {
					weekIndex++;
				  }
				  item.percent = 0;

				  if (item.isToday) {
					this.currentWeekIndex = item.weekIndex
					this.cacheWeekIndex = item.weekIndex
					this.activeDay = item.dateStr
					this.currentMonth = parseInt(item.month, 10) - 1;
					this.cacheMonthIndex = parseInt(item.month, 10) - 1;
					this.currentDate = item;
					this.activeObj = Object.assign({}, item)
					item.percent = (circleLength * 100) / 100;
					// this.calculateLineDate(calendarJson);
				  }
				});

				this.allDateList.push({
				  data: calendarJson
				});
			  }
			  // 设置 dateStr 输出格式
			}).format(this.format);
			
			for (let i = 0; i < cacheDateLen; i++) {
			      let month = i;
			      let year = 2021;
			      calendar.toDate({ year, month }).paint();
			}
			
			// 计算周 先聚合，在计算
			const weekObj = {}
			const weekPlatList = []
			// console.log(this.allDateList)
			this.allDateList.forEach((parent, parentIndex) => {
			  parent.data.forEach((item, key) => {
			    weekObj[`week-${item.weekIndex}`] = []
			  })
			  parent.data.forEach((item, key) => {
			    weekObj[`week-${item.weekIndex}`].push(item)
			  })
			})
						
			Object.keys(weekObj).forEach((parent) => {
			  // 判断是否有第一个重复
			  const filterList = weekPlatList.filter((result) => {
			    return result.dateStr === weekObj[parent][0].dateStr
			  })
			  if(filterList.length === 0) {
			    weekPlatList.push(weekObj[parent])
			  }
			})
			this.lineWeekDataList = weekPlatList
		},
		methods: {
			changeWeek(index) {
				this.cacheWeekIndex = index;
				const {month} = this.activeObj
				this.currentMonth = parseInt(month, 10) - 1
				this.currentDate.month = month
			},
			changeMonth(ev) {
				let index = ev.detail.current
			  // 切换 默认选中第一天
			  this.cacheMonthIndex = index;
			  this.currentMonth = index;
			  this.currentDate.month = index + 1;
			  // 默认选中一号
			  const monthTmpList = this.allDateList[index].data
			  for(let i = 0;i < monthTmpList.length;i++) {
			    if(monthTmpList[i].day === 1) {
			      // 选中的同一月 不默认选中
			      if(this.activeObj.month === monthTmpList[i].month) {
			        break
			      } else {
			        this.activeObj = monthTmpList[i]
			        this.activeDay = monthTmpList[i].dateStr
					this.$emit("changeDate", monthTmpList[i]);
			        break
			      }
			    }
			  }
			},
			updateExpandStatus() {
				this.isExpand = !this.isExpand;
				setTimeout(() => {
				  const weekIndex = this.activeObj.weekIndex
				  const monthIndex = this.activeObj.month - 1
				  // 展开月日历
					this.currentMonth = this.activeObj.month - 1
					this.currentWeekIndex = this.activeObj.weekIndex
				})
			},
			
			changeDateMethods(row) {
			  this.activeDay = row.data.dateStr;
			  this.activeObj = Object.assign({}, row.data)
			  this.$emit("changeDate", row);
			},
		}
	}
</script>

<style scoped lang="scss">
	@import './calendar.scss'
</style>
