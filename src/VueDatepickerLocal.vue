<template>
<div class="datepicker" :class="{'datepicker-range':range,'datepicker__clearable':clearable&&text&&!disabled}">
  <input readonly :value="text" :class="[show ? 'focus' : '', inputClass]" :disabled="disabled" :placeholder="placeholder" :name="name" v-if="type!=='inline'"/>
  <a class="datepicker-close" @click.stop="cls"></a>
  <transition name="datepicker-anim">
    <div class="datepicker-popup" :class="[popupClass,{'datepicker-inline':type==='inline'}]" tabindex="-1" v-if="show||type==='inline'">
      <template v-if="range">
        <vue-datepicker-local-calendar v-model="dates[0]" :left="true"></vue-datepicker-local-calendar>
        <vue-datepicker-local-calendar v-model="dates[1]" :right="true"></vue-datepicker-local-calendar>
      </template>
      <template v-else>
        <vue-datepicker-local-calendar v-model="dates[0]"></vue-datepicker-local-calendar>
      </template>
      <div v-if="showButtons" class="datepicker__buttons">
        <button @click.prevent.stop="cancel" class="datepicker__button-cancel">{{this.local.cancelTip}}</button>
        <button @click.prevent.stop="submit" class="datepicker__button-select">{{this.local.submitTip}}</button>
      </div>
    </div>
  </transition>
</div>
</template>

<script>
import VueDatepickerLocalCalendar from './VueDatepickerLocalCalendar.vue'
export default {
  name: 'VueDatepickerLocal',
  components: { VueDatepickerLocalCalendar },
  props: {
    name: [String],
    inputClass: [String],
    popupClass: [String],
    value: [Date, Array, String],
    disabled: [Boolean],
    type: {
      type: String,
      default: 'normal'
    },
    rangeSeparator: {
      type: String,
      default: '~'
    },
    clearable: {
      type: Boolean,
      default: false
    },
    placeholder: [String],
    disabledDate: {
      type: Function,
      default: () => {
        return false
      }
    },
    format: {
      type: String,
      default: 'YYYY-MM-DD'
    },
    local: {
      type: Object,
      default () {
        return {
          dow: 1, // Monday is the first day of the week
          hourTip: '选择小时', // tip of select hour
          minuteTip: '选择分钟', // tip of select minute
          secondTip: '选择秒数', // tip of select second
          yearSuffix: '年', // format of head
          monthsHead: '1月_2月_3月_4月_5月_6月_7月_8月_9月_10月_11月_12月'.split('_'), // months of head
          months: '一月_二月_三月_四月_五月_六月_七月_八月_九月_十月_十一月_十二月'.split('_'), // months of panel
          weeks: '一_二_三_四_五_六_日'.split('_'), // weeks
          cancelTip: '取消', // default text for cancel button
          submitTip: '确定' // default text for submit button
        }
      }
    },
    showButtons: {
      type: Boolean,
      default: false
    },
    dateRangeSelect: [Function]
  },
  data () {
    return {
      show: false,
      dates: this.vi(this.value)
    }
  },
  computed: {
    range () {
      return this.dates.length === 2
    },
    text () {
      const val = this.value
      const txt = this.dates.map(date => this.tf(date)).join(` ${this.rangeSeparator} `)
      if (Array.isArray(val)) {
        return val.length > 1 ? txt : ''
      } else {
        return val ? txt : ''
      }
    }
  },
  watch: {
    value (val) {
      this.dates = this.vi(this.value)
    }
  },
  methods: {
    get () {
      return Array.isArray(this.value) ? this.dates : this.dates[0]
    },
    cls () {
      this.$emit('clear')
      this.$emit('input', this.range ? [] : '')
    },
    vi (val) {
      if (Array.isArray(val)) {
        return val.length > 1 ? val.map(item => new Date(item)) : [new Date(), new Date()]
      } else {
        return val ? new Array(new Date(val)) : [new Date()]
      }
    },
    ok (leaveOpened) {
      const $this = this
      $this.$emit('input', $this.get())
      !leaveOpened && !$this.showButtons && setTimeout(() => {
        $this.show = $this.range
      })
    },
    tf (time, format) {
      const year = time.getFullYear()
      const month = time.getMonth()
      const day = time.getDate()
      const hours24 = time.getHours()
      const hours = hours24 % 12 === 0 ? 12 : hours24 % 12
      const minutes = time.getMinutes()
      const seconds = time.getSeconds()
      const milliseconds = time.getMilliseconds()
      const dd = t => ('0' + t).slice(-2)
      const map = {
        YYYY: year,
        MM: dd(month + 1),
        MMM: this.local.months[month],
        MMMM: this.local.monthsHead[month],
        M: month + 1,
        DD: dd(day),
        D: day,
        HH: dd(hours24),
        H: hours24,
        hh: dd(hours),
        h: hours,
        mm: dd(minutes),
        m: minutes,
        ss: dd(seconds),
        s: seconds,
        S: milliseconds
      }
      return (format || this.format).replace(/Y+|M+|D+|H+|h+|m+|s+|S+/g, str => map[str])
    },
    dc (e) {
      this.show = this.$el.contains(e.target) && !this.disabled
    },
    submit () {
      this.$emit('confirm', this.get())
      this.show = false
    },
    cancel () {
      this.$emit('cancel')
      this.show = false
    }
  },
  mounted () {
    document.addEventListener('click', this.dc, true)
  },
  beforeDestroy () {
    document.removeEventListener('click', this.dc, true)
  }
}
</script>

<style>
.datepicker {
  display: inline-block;
  position: relative;
  background-color: rgba(0, 32, 76, .9);
  border-radius: 15px;
}

.datepicker:before {
  content: '';
  display: block;
  position: absolute;
  width: 14px;
  height: 100%;
  top: 0;
  right: 25px;
  background: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAA4AAAAOCAYAAAAfSC3RAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyJpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENTNiAoV2luZG93cykiIHhtcE1NOkluc3RhbmNlSUQ9InhtcC5paWQ6OTc5RDlGN0VENEZBMTFFOEE1NkZFQ0QxRTBGMjYwOTEiIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6OTc5RDlGN0ZENEZBMTFFOEE1NkZFQ0QxRTBGMjYwOTEiPiA8eG1wTU06RGVyaXZlZEZyb20gc3RSZWY6aW5zdGFuY2VJRD0ieG1wLmlpZDo5NzlEOUY3Q0Q0RkExMUU4QTU2RkVDRDFFMEYyNjA5MSIgc3RSZWY6ZG9jdW1lbnRJRD0ieG1wLmRpZDo5NzlEOUY3REQ0RkExMUU4QTU2RkVDRDFFMEYyNjA5MSIvPiA8L3JkZjpEZXNjcmlwdGlvbj4gPC9yZGY6UkRGPiA8L3g6eG1wbWV0YT4gPD94cGFja2V0IGVuZD0iciI/Ppns8tcAAAEfSURBVHjajJI9SgRBEIWrx5HdwF8wEDZTwUsIgqHCJgYr7g3MFCMPYGTsDURMBDXbTDE2VlBjQVRkgh1GdPxKX8Ow7IgPvqmaol53ddOhLIozM1uGSftVacMVFN/hMmD0xkN4hA7s1xj34ATmYCvhk4HHF+hZvXrq+fH4jrcki1BAHybgA0Zk+IRReIUxaMBdItO2ClNa0fNUNFSbgSbsuCepGSuHNZGplun/3rRSqZWq2oAb0VWtq/+We+KOz7CgkeO5/pRfzhdxXc1HMK6x4k6xlmsS12mqZB4O1GCVWM3jcXbjGV0P0B64iLbIh42a6in5uOewqbrHC+WdAY/fR4hP7gqO4Ulj+wSzaoy1qFVYCXrkSzBt/9MbXH8LMACNxU68IFMi+gAAAABJRU5ErkJggg==') no-repeat 50% 50%;
}

.datepicker-close {
  display: none;
  position: absolute;
  width: 34px;
  height: 100%;
  top: 0;
  right: 0;
  cursor: pointer;
}

.datepicker-close:before {
  display: block;
  content: '';
  position: absolute;
  width: 16px;
  height: 16px;
  left: 50%;
  top: 50%;
  margin-left: -8px;
  margin-top: -8px;
  text-align: center;
  background: #ccc;
  color: #fff;
  border-radius: 50%;
  background:#ccc url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAA4AAAAOCAYAAAAfSC3RAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyJpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENTNiAoV2luZG93cykiIHhtcE1NOkluc3RhbmNlSUQ9InhtcC5paWQ6OTc5RDlGN0VENEZBMTFFOEE1NkZFQ0QxRTBGMjYwOTEiIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6OTc5RDlGN0ZENEZBMTFFOEE1NkZFQ0QxRTBGMjYwOTEiPiA8eG1wTU06RGVyaXZlZEZyb20gc3RSZWY6aW5zdGFuY2VJRD0ieG1wLmlpZDo5NzlEOUY3Q0Q0RkExMUU4QTU2RkVDRDFFMEYyNjA5MSIgc3RSZWY6ZG9jdW1lbnRJRD0ieG1wLmRpZDo5NzlEOUY3REQ0RkExMUU4QTU2RkVDRDFFMEYyNjA5MSIvPiA8L3JkZjpEZXNjcmlwdGlvbj4gPC9yZGY6UkRGPiA8L3g6eG1wbWV0YT4gPD94cGFja2V0IGVuZD0iciI/Ppns8tcAAAEfSURBVHjajJI9SgRBEIWrx5HdwF8wEDZTwUsIgqHCJgYr7g3MFCMPYGTsDURMBDXbTDE2VlBjQVRkgh1GdPxKX8Ow7IgPvqmaol53ddOhLIozM1uGSftVacMVFN/hMmD0xkN4hA7s1xj34ATmYCvhk4HHF+hZvXrq+fH4jrcki1BAHybgA0Zk+IRReIUxaMBdItO2ClNa0fNUNFSbgSbsuCepGSuHNZGplun/3rRSqZWq2oAb0VWtq/+We+KOz7CgkeO5/pRfzhdxXc1HMK6x4k6xlmsS12mqZB4O1GCVWM3jcXbjGV0P0B64iLbIh42a6in5uOewqbrHC+WdAY/fR4hP7gqO4Ulj+wSzaoy1qFVYCXrkSzBt/9MbXH8LMACNxU68IFMi+gAAAABJRU5ErkJggg==') no-repeat 50% 50%;
}

.datepicker__clearable:hover:before {
  display: none;
}
.datepicker__clearable:hover .datepicker-close{
  display: block;
}

.datepicker-close:hover:before{
  background-color: #afafaf;
}

.datepicker>input {
  color: white;
  transition: all 200ms ease;
  border: 1px solid #006aff;
  background-color: rgba(0, 106, 255, .2);
  height: 30px;
  box-sizing: border-box;
  outline: none;
  padding: 0 34px 0 12px;
  font-size: 14px;
  width: 100%;
  padding:0 25px;
  user-select: none;
  border-radius: 15px;
  -ms-user-select: none;
  -moz-user-select: none;
  -webkit-user-select: none;
  position: relative;
  z-index: 999;
}

.datepicker>input::-webkit-input-placeholder {
    color: white;
}

.datepicker>input.focus {
  /* border-color: #3bb4f2; */
  -webkit-box-shadow: 0 0 5px rgba(59, 180, 242, .3);
  box-shadow: 0 0 5px rgba(59, 180, 242, .3);
}

.datepicker>input:disabled {
  cursor: not-allowed;
  background-color: #ebebe4;
  border-color: #e5e5e5;
  -webkit-box-shadow: none;
  box-shadow: none;
}

.datepicker-popup {
  position: absolute;
  transition: all 200ms ease;
  opacity: 1;
  transform: scaleY(1);
  transform-origin: center top;
  font-size: 12px;
  background: #fff;
  /* border: 1px solid #d9d9d9; */
  box-shadow: 0 1px 6px rgba(99, 99, 99, 0.2);
  /* margin-top: 2px; */
  outline: 0;
  border-radius: 15px;
  /* padding: 0 25px; */
  text-align: center;
  overflow: hidden;
  z-index: 998;
  width: 100%;
  padding-top: 30px;
  top: -1px;
  background-color: rgba(0, 32, 76, .9);
}

.datepicker-inline{
  position: relative;
  margin-top: 0;
}

.datepicker-range {
  min-width: 325px
}

.datepicker-range .datepicker-popup{
  width: 403px;
}

.datepicker-bottom {
  float: left;
  width: 100%;
  text-align: right;
}

.datepicker-btn {
  padding: 5px 10px;
  background: #1284e7;
  color: #fff;
  border-radius: 2px;
  display: inline-block;
  cursor: pointer;
}
.datepicker-anim-enter-active {
    transform-origin: 0 0;
    animation: datepicker-anim-in .2s cubic-bezier(.23, 1, .32, 1)
}

.datepicker-anim-leave-active {
    transform-origin: 0 0;
    animation: datepicker-anim-out .2s cubic-bezier(.755, .05, .855, .06)
}

.datepicker__buttons {
  display: block;
  text-align: right;
}

.datepicker__buttons button {
  display: inline-block;
  font-size: 13px;
  border: none;
  cursor: pointer;
  margin: 10px 0 0 5px;
  padding: 5px 15px;
  color: #ffffff;
}

.datepicker__buttons .datepicker__button-select {
  background: #1284e7;
}

.datepicker__buttons .datepicker__button-cancel {
  background: #666;
}

@keyframes datepicker-anim-in {
    0% {
        opacity: 0;
        transform: scaleY(.8)
    }
    to {
        opacity: 1;
        transform: scaleY(1)
    }
}

@keyframes datepicker-anim-out {
    0% {
        opacity: 1;
        transform: scaleY(1)
    }
    to {
        opacity: 0;
        transform: scaleY(.8)
    }
}
</style>
