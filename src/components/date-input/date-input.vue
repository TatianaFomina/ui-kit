<template>
  <Input id="date-input"
         ref="date-input"
         :model-value="modelValue"
         :name="name"
         :disabled="disabled"
         :label="label"
         :required="required"
         :placeholder="patternStr"
         :error="error"
         @update:modelValue="onInput($event)"
  >
    <template #suffix>
      <Datepicker :model-value="modelValue"
                  :disabled="disabled"
                  :input-pattern="patternStr"
                  position="left"
                  @update:modelValue="$emit('update:modelValue', $event)"
      >
        <div
          class="pr-4 pl-3 font-medium leading-[20px] flex items-center focus:outline-none bg-transparent h-full"
          :class="[error ? 'border-burgundy' : 'border-gray-300', disabled ? 'text-gray-300 cursor-default' : 'text-gray-400']"
        >
          <CalendarIcon class="w-4 h-4" />
        </div>
      </Datepicker>
    </template>
  </Input>
</template>

<script lang='ts'>
import { defineComponent } from 'vue'
import Cleave from 'cleave.js'
import Datepicker from '/~/components/datepicker/datepicker.vue'
import CalendarIcon from './components/calendar-icon.vue'
import Input from '/~/components/input/input.vue'

const mapping: { [item: string]: string } = {
  m: 'MM',
  d: 'DD',
  y: 'YY',
  Y: 'YYYY'
}

export default defineComponent({
  name: 'DateInput',
  components: {
    Datepicker,
    CalendarIcon,
    Input
  },
  props: {
    modelValue: {
      type: String,
      default: ''
    },
    error: {
      type: String,
      default: ''
    },
    disabled: {
      type: Boolean,
      default: false
    },
    label: {
      type: String,
      default: ''
    },
    required: {
      type: Boolean,
      default: false
    },
    name: {
      type: String,
      default: ''
    },
    /**
     * Array indicating date pattern in Cleave.js style
     * ref: https://github.com/nosir/cleave.js/blob/master/doc/options.md#datepattern
     */
    datePattern: {
      type: Array,
      default: () => ['m', 'd', 'Y'],
      validator: (value: string[]): boolean => value.length === 3 && value.every(item => ['d', 'm', 'y', 'Y'].includes(item))
    },
    /**
     * Date parts delimeter
     */
    delimeter: {
      type: String,
      default: '/'
    }
  },
  emits: ['update:modelValue'],
  data() {
    return {
      cleave: null
    }
  },
  computed: {
    patternStr(): string {
      return this.datePattern.map((item: string) => mapping[item]).join(this.delimeter)
    }
  },
  mounted() {
    const inputRef = this.$refs['date-input'].inputRef

    this.cleave = new Cleave(inputRef, {
      date: true,
      datePattern: ['m', 'd', 'Y'],
      delimiter: '/',
      onValueChanged: (e: Event) => {
        this.onInput((e.target as HTMLInputElement).value)
      }
    })
  },
  methods: {
    onInput(value: string) {
      this.$emit('update:modelValue', value)
    }
  }

})

</script>
