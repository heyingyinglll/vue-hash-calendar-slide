/**
* @Description:    日期、时间选择器
* @Author:         TSY
* @CreateDate:     2019/5/17 16:22
* @Email:          t@tsy6.com
*/
<template>
    <div>
        <div class="hash-calendar" :class="{'calendar_inline': model === 'inline'}" v-show="isShowDatetimePicker"
             :style="{'height': `${model === 'inline' ? calendarContentHeight : undefined}px`}" @click="close">
            <div class="calendar_content" :style="{'height': `${calendarContentHeight}px`, 'background-color': `${mainBackgroundColor}`}" @click.stop>
                <calendar ref="calendar" v-if="pickerType !== 'time'" :show="isShowCalendar" v-bind="bindFields" @height="heightChange"
                          :default-date="defaultDatetime" @touchstart="touchStart" @touchmove="touchMove" @touchend="touchEnd" @slidechange="slideChange"
                          @change="dateChange" @click="dateClick"></calendar>
            </div>
        </div>
    </div>

</template>

<script>
import Calendar from './Calendar.vue'
import TimePicker from './TimePicker.vue'
import {formatDate} from '../utils/util'
import languageUtil from '../language'

export default {
  props: {
    visible: {// 是否显示日历组件
      type: Boolean,
      default: false
    },
    pickerType: {// 选择器类型 datetime：日期+时间   date：日期   time：时间
      type: String,
      default: 'datetime'
    },
    showTodayButton: {// 是否显示返回今日按钮
      type: Boolean,
      default: true
    },
    defaultDatetime: {// 默认时间
      type: Date,
      default() {
        return new Date()
      }
    },
    format: null, // 确认选择之后，返回的日期格式
    model: {
      type: String,
      default: 'inline'
    },
    // 日期下面的标记
    markDate: {
      type: Array,
      default: () => []
    },
    // 禁用的日期
    disabledDate: {
      type: Function,
      default: () => {
        return false
      }
    },
    // 使用的语言包
    lang: {
      type: String,
      default: 'CN'
    },
    mainBackgroundColor: {
      type: String,
      default: '#0e8ee9'
    }
  },
  components: {
    TimePicker,
    Calendar
  },
  name: 'VueHashCalendar',
  data() {
    return {
      language: {}, // 使用的语言包
      checkedDate: {
        year: new Date().getFullYear(),
        month: new Date().getMonth(),
        day: new Date().getDate(),
        hours: new Date().getHours(),
        minutes: new Date().getMinutes()
      }, // 被选中的日期
      isShowCalendar: false, // 是否显示日历选择控件
      calendarContentHeight: 0, // 日历组件高度
      calendarTitleHeight: 0, // 日历组件标题高度
      firstTimes: true// 第一次触发
    }
  },
  mounted() {
    if (this.model === 'inline') {
      this.isShowDatetimePicker = true
    }

    this.language = languageUtil[this.lang.toUpperCase()]
  },
  watch: {
    defaultDatetime(val) {
      if (!(val instanceof Date)) {
        throw new Error(`The calendar component's defaultDate must be date type!`)
      }
    },
    pickerType: {
      handler(val) {
        if (val === 'time') {
          this.showTime()
        }
      },
      immediate: true
    },
    checkedDate: {
      handler() {
        let date = new Date(`${this.checkedDate.year}/${this.checkedDate.month + 1}/${this.checkedDate.day} ${this.checkedDate.hours}:${this.checkedDate.minutes}`)
        if (this.format) {
          date = formatDate(date, this.format, this.lang)
        }
        this.$emit('change', date)
      },
      deep: true
    },
    visible: {
      handler(val) {
        this.isShowCalendar = val
      },
      immediate: true
    }
  },
  computed: {
    isShowDatetimePicker: {// 是否显示日期控件
      get() {
        return this.visible
      },
      set(val) {
        this.$emit('update:visible', val)
      }
    },
    bindFields() {
      let fs = {...this.$props, ...this.$attrs}
      return fs
    }
  },
  methods: {
    today() {
      if (this.disabledDate(new Date())) return

      this.$refs.calendar.today()
    },
    dateChange(date) {
      date.hours = this.checkedDate.hours
      date.minutes = this.checkedDate.minutes
      this.checkedDate = date
    },
    dateClick(date) {
      date.hours = this.checkedDate.hours
      date.minutes = this.checkedDate.minutes
      this.checkedDate = date

      let fDate = new Date(`${this.checkedDate.year}/${this.checkedDate.month + 1}/${this.checkedDate.day} ${this.checkedDate.hours}:${this.checkedDate.minutes}`)
      if (this.format) {
        fDate = formatDate(fDate, this.format, this.lang)
      }
      this.$emit('click', fDate)
    },
    timeChange(date) {
      date.year = this.checkedDate.year
      date.month = this.checkedDate.month
      date.day = this.checkedDate.day
      this.checkedDate = date
    },
    confirm() { // 确认选择时间
      let date = new Date(`${this.checkedDate.year}/${this.checkedDate.month + 1}/${this.checkedDate.day} ${this.checkedDate.hours}:${this.checkedDate.minutes}`)
      if (this.format) {
        date = formatDate(date, this.format, this.lang)
      }
      this.$emit('confirm', date)
      if (this.model === 'dialog') {
        this.close()
      }
    },
    show() {
      this.isShowDatetimePicker = true
    },
    close() {
      this.isShowDatetimePicker = false
    },
    fillNumber(val) { // 小于10，在前面补0
      return val > 9 ? val : '0' + val
    },
    formatDate(time, format) {
      return formatDate(time, format, this.lang)
    },
    showCalendar() { // 显示日历控件
      this.isShowCalendar = true
    },
    showTime() { // 显示时间选择控件
      this.isShowCalendar = false
    },
    heightChange(height) { // 高度变化
      if (!this.firstTimes && this.model === 'dialog') return

      this.calendarContentHeight = height
      this.$emit('height', height)
    },
    touchStart(event) { // 监听手指开始滑动事件
      this.$emit('touchstart', event)
    },
    touchMove(event) { // 监听手指开始滑动事件
      this.$emit('touchmove', event)
    },
    touchEnd(event) { // 监听手指开始滑动事件
      this.$emit('touchend', event)
    },
    slideChange(direction) { // 滑动方向改变
      this.$emit('slidechange', direction)
    }
  }
}
</script>

<style lang="stylus" scoped>
    @import "../style/common.styl"

    .hash-calendar {
        width 100vw
        z-index 999
        color #ffffff
        display table
    }

    .calendar_inline {
        position relative
        width 100%
        height auto
        background none
        height px2vw(710px)
        z-index 1
    }

    .calendar_content {
        position absolute
        width 100%
        left 0
        display flex
        padding-bottom px2vw(26px)
        flex-wrap wrap
        /*background main-color*/
        color #ffffff
        height px2vw(710px)
        overflow hidden
    }

    .calendar_title {
        position absolute
        width 100%
        left 0
        top 0
        background bg-color
        borderBottom()
        display flex
        align-items center
        justify-content space-between
        z-index 1
    }

    .calendar_title_date {
        color vice-font-color
        background white
        padding px2vw(30px) px2vw(15px)
    }

    .calendar_title_date_active {
        color main-font-color
        font-weight bold
    }

    .calendar_title_date_time {
        margin-left px2vw(20px)
    }

    .calendar_confirm {
        color main-color
        margin-right px2vw(34px)
    }

    .today_disable {
        color disabled-font-color
    }
</style>
