<template>
  <div class="flex relative">
    <button aria-label="Expand date selection"
            @click="!disabled && open()"
    >
      <slot />
    </button>

    <transition enter-active-class="transition duration-150"
                enter-from-class="transform -translate-y-4 opacity-0"
                enter-to-class="opacity-100 translate-y-0"
                leave-active-class="transition duration-150"
                leave-from-class="opacity-100 transform translate-y-0"
                leave-to-class="opacity-0 -translate-y-4"
    >
      <div v-if="isExpanded"
           ref="calendar"
           v-click-away="close"
           role="dialog"
           aria-modal="true"
           :class="positionClasses"
           class="absolute top-[120%] rounded-2xl border border-gray-200 py-2 bg-white text-gray-500 space-y-2 shadow-sm outline-none w-60"
           tabindex="0"
           @keydown="onKeydown"
      >
        <div class="flex justify-between px-2">
          <button ref="firstFocusableElement"
                  class="text-gray-400 hover:text-gray-300"
                  @click="showPrevMonth"
          >
            <ChevronLeftIcon ref="leftButton"
                             class="w-4 h-4"
            />
          </button>
          <MonthPicker ref="monthpicker"
                       :date="displayedDate"
                       @showPrevYear="showPrevYear"
                       @showNextYear="showNextYear"
                       @selectMonth="selectMonth"
                       @selectYear="selectYear"
          />
          <button ref="rightButton"
                  class="text-gray-400 hover:text-gray-300"
                  @click="showNextMonth"
          >
            <ChevronRightIcon class="w-4 h-4" />
          </button>
        </div>
        <div class="grid grid-cols-7 grid-rows-1 gap-1 justify-items-center text-xs font-light uppercase border-b border-t px-2">
          <div v-for="weekDayName of weekDays"
               :key="weekDayName"
               class="w-7 text-center"
          >
            {{ weekDayName }}
          </div>
        </div>
        <div class="grid grid-cols-7 grid-rows-6 gap-1 justify-items-center text-sm px-2">
          <button v-for="(day, i) in daysOfPrevMonthCount"
                  :key="day"
                  :tabindex="isDateSelectedPrevMonth(day) ? 0 : -1"
                  class="text-gray-300 w-7 h-7 leading-7 text-center"
                  @click="selectValuePrevMonth(day)"
          >
            <p class="rounded-full hover:bg-gray-100">
              {{ firstDayOfPrevMonthDisplayed + i }}
            </p>
          </button>
          <button v-for="day in daysInMonth"
                  :key="day"
                  class="w-7 h-7 leading-7 text-center"
                  :tabindex="isDateSelected(day) ? 0 : -1"
                  @click="selectValue(day)"
          >
            <p class="rounded-full"
               :class="[isDateSelected(day) ? 'bg-sky-50' : 'hover:bg-gray-100']"
            >
              {{ day }}
            </p>
          </button>
          <button v-for="day in daysOfNextMonthCount"
                  :key="day"
                  :tabindex="isDateSelectedNextMonth(day) ? 0 : -1"
                  class="text-gray-300 w-7 h-7 leading-7 text-center"
                  @click="selectValueNextMonth(day)"
          >
            <p class="rounded-full hover:bg-gray-100">
              {{ day }}
            </p>
          </button>
        </div>
        <span class="opacity-0"
              tabindex="0"
              @focus="loopFocus"
        />
      </div>
    </transition>
  </div>
</template>

<script lang='ts'>
import { defineComponent, PropType } from 'vue'
import ChevronLeftIcon from './components/chevron-left-icon.vue'
import ChevronRightIcon from './components/chevron-right-icon.vue'
import MonthPicker from './components/monthpicker.vue'
import dayjs from 'dayjs'
import localizedFormat from 'dayjs/plugin/localizedFormat'
import customParseFormat from 'dayjs/plugin/customParseFormat'
import { directive } from 'vue3-click-away'

const positions: { [key: string]: string } = {
  right: 'left-0',
  left: 'right-0'
}

dayjs.extend(localizedFormat)
dayjs.extend(customParseFormat)

export default defineComponent({
  name: 'Datepicker',
  components: {
    ChevronLeftIcon,
    ChevronRightIcon,
    MonthPicker
  },
  directives: {
    ClickAway: directive
  },
  props: {
    modelValue: {
      type: Object as PropType<Date>,
      default: () => dayjs()
    },

    /**
     * Determines whether calendar should open on click
     */
    disabled: {
      type: Boolean,
      default: false
    },

    /**
     * Format input value is represented in
     */
    inputPattern: {
      type: String,
      default: 'L'
    },

    /**
     * Format output value is represented in
     */
    outputFormat: {
      type: String,
      default: 'L'
    },

    /**
     * Dropdown calendar position relative to its trigger
     */
    position: {
      type: String,
      default: 'right',
      validator: (value: string): boolean => ['right', 'left'].includes(value)
    }
  },
  emits: ['update:modelValue'],
  data() {
    return {
      isExpanded: false,
      weekDays: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],
      displayedDate: null as unknown as dayjs.Dayjs
    }
  },
  computed: {
    /**
     * Number of days in current month
     */
    daysInMonth(): number {
      return this.displayedDate.daysInMonth()
    },

    /**
     * Current month name
     */
    currentMonth(): string {
      return this.displayedDate.format('MMMM')
    },

    /**
     * Current year string representation
     */
    currentYear(): string {
      return this.displayedDate.format('YYYY')
    },

    /**
     * Number of days in previous month
     */
    daysOfPrevMonthCount(): number {
      return this.displayedDate.startOf('month').day()
    },

    /**
     * First of last several days of previous month displayed in calendar
     */
    firstDayOfPrevMonthDisplayed(): number {
      const daysInPrevMonth = this.displayedDate.subtract(1, 'month').daysInMonth()

      return daysInPrevMonth - this.daysOfPrevMonthCount
    },

    /**
     * Number of next month days displayed in calendar
     */
    daysOfNextMonthCount(): number {
      return 42 - this.daysInMonth - this.daysOfPrevMonthCount
    },

    /**
     * CSS classes applied to calendar dropdown according to props passed
     */
    positionClasses() {
      return positions[this.position] || positions.right
    }
  },
  methods: {
    /**
     * Returns true if day with specified index in current month in selected
     */
    isDateSelected(date: number): boolean {
      return this.displayedDate.set('date', date).isSame(dayjs(this.modelValue))
    },
    /**
     * Returns true if day with specified index in next month in selected
     */
    isDateSelectedNextMonth(date: number): boolean {
      return this.displayedDate.add(1, 'month').set('date', date).isSame(dayjs(this.modelValue))
    },
    /**
     * Returns true if day with specified index in previous month in selected
     */
    isDateSelectedPrevMonth(date: number): boolean {
      return this.displayedDate.subtract(1, 'month').set('date', date).isSame(dayjs(this.modelValue))
    },
    selectValue(date: number) {
      this.$emit('update:modelValue', this.displayedDate.set('date', date).format(this.outputFormat))
      this.isExpanded = false
    },
    selectValueNextMonth(date: number) {
      this.$emit('update:modelValue', this.displayedDate.add(1, 'month').set('date', date).format(this.outputFormat))
      this.isExpanded = false
    },
    selectValuePrevMonth(date: number) {
      this.$emit('update:modelValue', this.displayedDate.subtract(1, 'month').set('date', date).format(this.outputFormat))
      this.isExpanded = false
    },
    showPrevMonth() {
      this.displayedDate = this.displayedDate.subtract(1, 'month')
    },
    showNextMonth() {
      this.displayedDate = this.displayedDate.add(1, 'month')
    },
    showPrevYear(count = 1) {
      this.displayedDate = this.displayedDate.subtract(count, 'year')
    },
    showNextYear(count = 1) {
      this.displayedDate = this.displayedDate.add(count, 'year')
    },
    open() {
      let value = this.modelValue ? dayjs(this.modelValue, this.inputPattern) : dayjs()

      if (!value.isValid()) {
        value = dayjs()
      }
      this.displayedDate = value
      this.isExpanded = true
      this.$nextTick(() => {
        this.$refs.calendar.focus()
      })
    },
    close() {
      this.isExpanded = false
    },
    selectMonth(i: number) {
      this.displayedDate = this.displayedDate.set('month', i)
    },
    selectYear(year: number) {
      this.displayedDate = this.displayedDate.set('year', year)
    },
    onKeydown(e: KeyboardEvent) {
      const { firstFocusableElement: leftButton, rightButton, monthpicker } = this.$refs

      switch (e.code) {
        case 'Enter':
        case 'Space':
          // Don't close calendar if hitting space or enter with arrow or monthpicker buttons focused
          if (!leftButton.isSameNode(e.target) && !rightButton.isSameNode(e.target) && !monthpicker.$el.contains(e.target)) {
            this.close()
          }
          break
        case 'Escape':
          this.close()
          break
        case 'ArrowDown':
          if (!this.modelValue) {
            return
          }
          this.displayedDate = this.displayedDate.add(7, 'day')
          this.$emit('update:modelValue', this.displayedDate.format(this.outputFormat))
          break
        case 'ArrowUp':
          if (!this.modelValue) {
            return
          }
          this.displayedDate = this.displayedDate.subtract(7, 'day')
          this.$emit('update:modelValue', this.displayedDate.format(this.outputFormat))
          break
        case 'ArrowRight':
          if (!this.modelValue) {
            return
          }
          this.displayedDate = this.displayedDate.add(1, 'day')
          this.$emit('update:modelValue', this.displayedDate.format(this.outputFormat))
          break
        case 'ArrowLeft':
          if (!this.modelValue) {
            return
          }
          this.displayedDate = this.displayedDate.subtract(1, 'day')
          this.$emit('update:modelValue', this.displayedDate.format(this.outputFormat))
          break
      }
    },
    /**
     * Prevents focus from leaving dropdown element
     */
    loopFocus() {
      this.$refs.firstFocusableElement.focus()
    }
  }
})

</script>
