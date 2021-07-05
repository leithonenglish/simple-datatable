<template>
  <button :class="{ checked }" @click.stop="checked = !checked">
    <CheckIcon v-if="checked" />
  </button>
</template>

<script lang="ts">
import { defineComponent, ref, toRefs, watch } from "vue";
import CheckIcon from "../assets/check.svg";
export default defineComponent({
  name: "Checkbox",
  emits: ['change', 'update:modelValue'],
  components: {
    CheckIcon
  },
  props: {
    modelValue: {
      type: Boolean,
      default: false
    }
  },
  setup(props, { emit }) {
    const { modelValue } = toRefs(props);
    const checked = ref(modelValue.value);
    watch(modelValue, (newValue) => checked.value = newValue);
    watch(checked, (newValue) => {
      emit('change', newValue);
      emit('update:modelValue', newValue)
    });
    return { checked };
  }
});
</script>

<style lang="scss" scoped>
$highlighted-color: #0988ff;

button {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 20px;
  width: 20px;
  padding: 0;
  background: #fff;
  border-radius: 3px;
  border: 1px solid #a5a5a5;
  transition-property: background, border-color;
  transition-duration: 0.2s;
  transition-timing-function: ease-out;

  svg {
    width: 12px;
  }

  &.checked {
    background: $highlighted-color;
    border-color: $highlighted-color;

    svg {
      fill: white;
    }
  }

  &:hover {
    cursor: pointer;
    border-color: $highlighted-color;
  }
}
</style>