<template>
  <label class="text-sm space-x-3 font-medium flex items-center"
         :class="[disabled ? 'text-gray-300' : 'text-gray-500']"
  >
    <div class="relative w-4 h-4 focus-within:ring ring-sky-50 rounded-full bg-gray-50 border border-gray-300"
         :class="[disabled ? 'cursor-default' : 'cursor-pointer']"
    >
      <input type="radio"
             :name="name"
             class="absolute inset-0 opacity-0 outline-none"
             :disabled="disabled"
             :required="required"
             @keydown.space="selectValue"
      >

      <div class="absolute inset-0.5 rounded-full outline-none transition"
           :class="[!disabled && modelValue && 'bg-sky-400', disabled && modelValue && 'bg-sky-100']"
           @click="selectValue"
      />

    </div>
    <span @click="selectValue">
      {{ label }}
    </span>
  </label>
</template>

<script lang='ts'>
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'RadioButton',
  props: {
    modelValue: {
      type: Boolean,
      default: false
    },
    label: {
      type: String,
      default: ''
    },
    name: {
      type: String,
      default: ''
    },
    disabled: {
      type: Boolean,
      default: false
    },
    required: {
      type: Boolean,
      default: false
    }
  },
  emits: ['update:modelValue'],
  methods: {
    selectValue() {
      if (this.disabled) {
        return
      }
      if (!this.modelValue) {
        this.$emit('update:modelValue', true)
      }
    }
  }
})

</script>
