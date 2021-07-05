<template>
  <div class="radio-btn">
    <button
      v-for="{ value, label } in options"
      :key="value"
      :class="{ selected: selectedValue === value }"
      @click.stop="selectedValue = value"
    >
      {{ label }}
    </button>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType, ref, toRefs, watch } from "vue";
type RadioButtonOptionsType = {
  value: any;
  label: string;
}
export default defineComponent({
  name: "RadioButton",
  emits: ['change', 'update:modelValue'],
  props: {
    modelValue: {},
    options: {
      type: Array as PropType<RadioButtonOptionsType[]>,
      required: true
    }
  },
  setup(props, { emit }) {
    const { modelValue } = toRefs(props);
    const selectedValue = ref(modelValue.value);
    watch(modelValue, (newValue) => selectedValue.value = newValue);
    watch(selectedValue, (newValue) => {
      emit('change', newValue);
      emit('update:modelValue', newValue)
    });
    return { selectedValue };
  }
});
</script>

<style lang="scss" scoped>
$highlighted-color: #0988ff;
div {
  display: flex;
  justify-content: space-evenly;
  align-items: center;

  button {
    background: #fff;
    border: 1px solid #bdbdbd;
    border-width: 1px 0 1px 1px;
    padding: 0.65em 1em;
    min-width: 80px;
    width: 100%;
    font-weight: bold;
    font-family: inherit;
    transition-property: background, border-color;
    transition-duration: 0.2s;
    transition-timing-function: ease-out;

    &:first-child {
      border-radius: 3px 0 0 3px;
    }

    &:last-child {
      border-right-width: 1px;
      border-radius: 0 3px 3px 0;
    }

    &.selected {
      background: $highlighted-color;
      color: #fff;
      border: 1px solid $highlighted-color;
    }

    &:not(.selected):hover {
      cursor: pointer;
      background: rgba($highlighted-color, 0.3);
    }
  }
}
</style>