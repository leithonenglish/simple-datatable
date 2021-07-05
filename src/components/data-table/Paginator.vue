<template>
  <div class="paginator">
    <button
      :disabled="!canGoBack"
      class="first"
      @click.prevent="currentPage = 1"
    >
      <FastForwardIcon />
    </button>
    <button
      :disabled="!canGoBack"
      class="previous"
      @click.prevent="currentPage = currentPage - 1"
    >
      <NextIcon />
    </button>
    <div class="pages">
      <button
        v-for="number in pages"
        :key="number"
        :class="{ selected: number === currentPage }"
        @click="currentPage = number"
      >
        {{ number }}
      </button>
    </div>
    <button
      :disabled="!canGoForward"
      class="next"
      @click.prevent="currentPage = currentPage + 1"
    >
      <NextIcon />
    </button>
    <button
      :disabled="!canGoForward"
      class="last"
      @click.prevent="currentPage = maxPageNumber"
    >
      <FastForwardIcon />
    </button>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, ref, toRef, toRefs, watch } from "vue";
import NextIcon from "../../assets/next.svg";
import FastForwardIcon from "../../assets/fast-forward.svg";

type PaginatorProps = {
  page: number;
  size: number;
  rows: number;
}

export default defineComponent({
  name: "Pagiator",
  components: {
    NextIcon,
    FastForwardIcon
  },
  emits: ['change'],
  props: {
    page: {
      type: Number,
      default: 1
    },
    size: {
      type: Number,
      required: true
    },
    rows: {
      type: Number,
      required: true
    },
  },
  setup(props, { emit }) {
    const { size, rows, page } = toRefs<PaginatorProps>(props);
    const currentPage = ref(props.page);
    const canGoBack = computed(() => currentPage.value > 1);
    const canGoForward = computed(() => currentPage.value < maxPageNumber.value);
    const maxPageNumber = computed(() => Math.ceil(size.value / rows.value));
    const pages = computed(() => {
      const count = 5;
      const pageNumbers = [...Array(maxPageNumber.value)].map((_,i) => i+1);
      if (currentPage.value < count) {
        return pageNumbers.slice(0, count);
      }
      return pageNumbers.slice(currentPage.value - 3, currentPage.value + count - 3);
    });
    watch(page, (newValue) => currentPage.value = newValue);
    watch(currentPage, (newValue: number) =>  emit('change', newValue));

    return {
      currentPage,
      canGoBack,
      canGoForward,
      pages,
      maxPageNumber
    };
  }
});
</script>

<style lang="scss" scoped>
.paginator {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;

  > button {
    background: transparent;
    border: none;

    svg {
      width: 16px;
      transition: fill 0.3s ease-out;
    }

    &.first,
    &.previous {
      svg {
        transform: rotate(180deg);
      }
    }

    &:disabled {
      cursor: default;
      svg {
        fill: #e9e9e9;
      }
    }

    &:not(:disabled):hover {
      cursor: pointer;

      svg {
        fill: royalblue;
      }
    }
  }

  > div {
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 0 1.5em;

    button {
      background: #fff;
      border: none;
      height: 30px;
      min-width: 30px;
      margin: 0 2px;
      font-size: 1rem;
      font-family: inherit;
      border-radius: 5px;

      &:first-child {
        margin-left: 0;
      }

      &:last-child {
        margin-right: 0;
      }

      &.selected {
        background: #cfe6ff;
      }

      &:not(.selected):hover {
        cursor: pointer;
        background: #e9e9e9;
      }
    }
  }
}
</style>