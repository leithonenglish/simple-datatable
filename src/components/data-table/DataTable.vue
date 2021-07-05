<template>
  <div class="data-table">
    <slot name="header"></slot>
    <slot></slot>
    <div v-if="hasGlobalSearch" class="global-actions">
      <div class="search-holder">
        <SearchIcon />
        <input v-model="globalSearchText" type="text" placeholder="Keyword search" />
      </div>
      <div class="search-fields">
        <span @click="isSearchFieldsOpen = !isSearchFieldsOpen">
          {{ searchFieldsText }}
        </span>
        <ul v-if="isSearchFieldsOpen" v-click-outside="onClickOutsideSearch">
          <li v-for="{ field, header } in columns" :key="field">
            <Checkbox :modelValue="isSearchFieldSelected(field)" @change="onSearchFieldSelected(field)"/>
            {{ header }}
          </li>
        </ul>
      </div>
      <button @click.prevent="isExporterOpen = true">
        <ExportIcon />
      </button>
    </div>
    <div :class="['table-holder', { full: fullWidth }]"  @scroll="onTableScroll">
      <table v-if="value && value.length > 0">
        <thead>
          <tr>
            <th v-if="selectable" :class="['selector fixed', { scrolled: isScrolled }]"></th>
            <th
              v-for="{ header, sortable, field } in columns" :key="header"
              :class="{ sortable }"
              @click="onColumnClick(field, sortable)"
            >
              <div class="holder">
                <span>{{ header }}</span>
                <div class="actions">
                  <div v-if="sortable" :class="['sort', isSorting(field) ? currentSort.order : '']">
                    <transition
                      enter-active-class="fadeInUp"
                      leave-active-class="fadeOutUp"
                      mode="out-in"
                    >
                      <DescIcon v-if="isSorting(field)" />
                      <SortIcon v-else />
                    </transition>
                  </div>
                </div>
              </div>
            </th>
          </tr>
        </thead>
        <tbody>
          <template v-for="(item, index) in rowData" :key="index">
            <tr
              :class="{ selected: isRowSelected(item), 'can-expand': canExpand }"
              @click="onRowClick(item)"
            >
              <td v-if="selectable" :class="['selector fixed', { scrolled: isScrolled }]">
                <Checkbox @change="onRowSelected(item)" />
              </td>
              <td
                v-for="{ field, header, wrap, width, children } in columns"
                :key="header + field"
              >
                <span
                  :class="{ 'no-wrap': !wrap }"
                  :style="{ width: width || 'auto' }"
                >
                  <template v-if="!children">
                    {{ item[field] }}
                  </template>
                  <component v-else :is="children.default" :item="item" />
                </span>
              </td>
            </tr>
            <tr v-if="isExpanded(item) && canExpand" class="expander">
              <td v-if="selectable" :class="['selector fixed', { scrolled: isScrolled }]">
                <ExpandedIcon class="expanded-icon" />
              </td>
              <td :colspan="columns.length">
                <div>
                  <ExpandedIcon v-if="!selectable" class="expanded-icon" />
                  <slot name="expander" :data="item"></slot>
                </div>
              </td>
            </tr>
          </template>
        </tbody>
      </table>
    </div>
    <div class="info">
      <div class="detail">
        <h1>{{ pageInfo }}</h1>
      </div>
      <Paginator
        :page="currentPage"
        :rows="currentRows"
        :size="value.length"
        @change="currentPage = $event"
      />
      <select class="rows-ddl" @change="onRowsChanged">
        <option v-for="amount in rowsPerPageOptions" :key="amount">
          {{ amount }}
        </option>
      </select>
    </div>
    <Exporter
      v-if="isExporterOpen"
      :columns="columns"
      :data="dataForExport"
      @cancelled="isExporterOpen = false"
    />
    <slot name="footer"></slot>
  </div>
</template>
<script lang="ts">
import { ref, defineComponent, computed, PropType, onMounted } from "vue";
import Paginator from './Paginator.vue';
import Exporter from './Exporter.vue';
import Checkbox from '../Checkbox.vue';
import SearchIcon from "../../assets/search.svg";
import DescIcon from "../../assets/down-arrow.svg";
import SortIcon from "../../assets/sort.svg";
import ExportIcon from "../../assets/export.svg";
import ExpandedIcon from "../../assets/left-round-angle-arrow.svg";

type ColumnProps = {
  header: string;
  field: string;
  sortable: boolean | string;
  wrap: boolean | string;
  width: string | undefined
  children: any
};

type Sort = {
  field: string;
  order: string;
};

export default defineComponent({
  name: "DataTable",
  components: {
    Paginator,
    Checkbox,
    Exporter,
    SearchIcon,
    DescIcon,
    SortIcon,
    ExportIcon,
    ExpandedIcon
  },
  props: {
    value: {
      type: Array,
      required: true,
    },
    rows: {
      type: Number,
      default: 10,
    },
    globalSearchFields: {
      type: Array as PropType<Array<string>>,
      default: []
    },
    removableSort: {
      type: Boolean,
      default: false,
    },
    selectable: {
      type: Boolean,
      default: false,
    },
    fullWidth: {
      type: Boolean,
      default: true,
    },
    page: {
      type: Number,
      default: 1
    },
    rowsPerPageOptions: {
      type: Array as PropType<number[]>,
      default: [10, 20, 30]
    }
  },
  setup: (
    { value, rows, globalSearchFields, removableSort, page },
    { slots }
  ) => {
    const currentSort = ref(null as Sort | null);
    const globalSearchText = ref("");
    const searchFields = ref(globalSearchFields);
    const isSearchFieldsOpen = ref(false);
    const isExporterOpen = ref(false);
    const currentPage = ref(page as number);
    const currentRows = ref(rows as number);
    const expandedRows = ref([] as any[]);
    const isScrolled = ref(false);
    const selectedItems = ref([] as Array<any>);
    const children = computed(() => (slots.default ? slots.default() : []));
    const canExpand = computed(() => !!slots.expander);
    const dataForExport = computed(() => selectedItems.value.length > 0 ? selectedItems.value : value);
    const columns = computed<Array<ColumnProps>>(() =>
      children.value
        .filter(({ type }) => type && type.name === "Column")
        .map((child) => {
          const { field, header, sortable, wrap, width } =
            child.props as ColumnProps;
          return {
            field,
            header,
            wrap: wrap || typeof wrap === "undefined",
            sortable: sortable || sortable == "",
            width,
            children: child.children
          };
        })
    );
    const hasGlobalSearch = computed(() => !!globalSearchFields);

    const searchFieldsText = computed(() => {
      if (searchFields.value.length === 0) {
        return 'Please select fields';
      } else if (searchFields.value.length > 1) {
        return 'Multiple fields';
      } else {
        const column = columns.value.find(({ field }) => field === searchFields.value[0]);
        return column ? column.header : 'None';
      }
    });

    // generate data for table based on sorts or page
    const rowData = computed(() => {
      let data = [...value];
      if (hasGlobalSearch.value && globalSearchText.value) {
        data = data.filter((row: any) => {
          if (searchFields.value && searchFields.value.length > 0) {
            let matched = false;
            searchFields.value?.forEach((field) => {
              const cell = row[field];
              if (typeof cell === "string") {
                matched =
                  matched ||
                  cell
                    .toLowerCase()
                    .includes(globalSearchText.value.toLowerCase());
              }
            });
            return matched;
          }
          return true;
        });
      }
      sortData(data);
      const start = currentPage.value < 2 ? 0 : ((currentPage.value - 1) * currentRows.value);
      const end = start + currentRows.value;
      return data.slice(start, end);
    });
    const pageInfo = computed(() => {
      return `Showing ${((currentPage.value - 1) * currentRows.value) + 1} to ${currentPage.value * currentRows.value} of ${value.length}`
    });
    const sortData = (data: any[]) => {
      if (currentSort.value) {
        const { field, order } = currentSort.value;
        data.sort((a, b) => {
          if (order === "desc") {
            return a[field] > b[field] ? 1 : -1;
          }
          return a[field] < b[field] ? 1 : -1;
        });
      }
    };
    const isSorting = (field: string) => {
      return currentSort.value?.field === field;
    };
    const isRowSelected = (item: any[]) => selectedItems.value.indexOf(item) > -1;
    const isSearchFieldSelected = (field: string) => {
      return searchFields.value.indexOf(field) > -1
    };
    const isExpanded = (item: any) => {
      return expandedRows.value.indexOf(item) > -1;
    }
    const onTableScroll = (event: Event) => {
      isScrolled.value = (event.target as HTMLDivElement).scrollLeft > 0;
    };
    const onRowClick = (item: any) => {
      if (canExpand.value) {
        const rowIndex = expandedRows.value.indexOf(item);
        if (rowIndex > - 1) {
          expandedRows.value.splice(rowIndex, 1);
        } else {
          expandedRows.value.push(item)
        }
      }
    }
    const onRowSelected = (item: any) => {
      const itemIndex = selectedItems.value.indexOf(item);
      if (itemIndex > -1) {
        selectedItems.value.splice(itemIndex, 1);
      } else {
        selectedItems.value.push(item);
      }
    };
    const onRowsChanged = (event: Event) => {
      const rows = parseInt((event.target as HTMLInputElement).value);
      const maxPage = Math.ceil(value.length / rows);
      currentRows.value = rows;
      if (currentPage.value > maxPage) {
        currentPage.value = maxPage;
      }
    };

    const onColumnClick = (field: string, sortable: boolean) => {
      if (sortable) {
        if (isSorting(field) && currentSort.value) {
          const { order } = currentSort.value;
          if (removableSort && order === "asc") {
            currentSort.value = null;
          } else {
            currentSort.value.order = order === "asc" ? "desc" : "asc";
          }
        } else {
          currentSort.value = { field, order: "desc" };
        }
      }
    };

    const onSearchFieldSelected = (field: string) => {
      const itemIndex = searchFields.value.indexOf(field);
      if (itemIndex > -1) {
        searchFields.value.splice(itemIndex, 1);
      } else {
        searchFields.value.push(field);
      }
    };

    const onClickOutsideSearch = () => isSearchFieldsOpen.value = false;
    return {
      columns,
      rowData,
      hasGlobalSearch,
      currentSort,
      globalSearchText,
      pageInfo,
      currentRows,
      currentPage,
      canExpand,
      dataForExport,
      searchFieldsText,
      isExporterOpen,
      isSorting,
      isSearchFieldSelected,
      isScrolled,
      isSearchFieldsOpen,
      isRowSelected,
      isExpanded,
      onColumnClick,
      onRowClick,
      onRowSelected,
      onRowsChanged,
      onTableScroll,
      onSearchFieldSelected,
      onClickOutsideSearch
    };
  },
});
</script>

<style lang="scss" scoped>
.data-table {
  position: relative;
  display: flex;
  flex-direction: column;
  margin: 2em;

  .global-actions {
    padding: 1em;
    border: 1px solid #e9ecef;
    border-bottom: none;
    background: #f8f9fa;
    display: flex;
    justify-content: flex-end;
    width: 100%;
    box-sizing: border-box;

    > button {
      background: #fff;
      border: 1px solid #b5b8bb;
      border-radius: 4px;
      margin-left: 1em;
      width: 40px;
      transition-property: border-color, background;
      transition-duration: 0.3s;
      transition-timing-function: cubic-bezier(0.075, 0.82, 0.165, 1);

      svg {
        width: 20px;
        margin-top: 2px;
        fill: #3f3c3c;
        transition: fill 0.2s ease-out;
      }

      &:hover {
        background: lightblue;
        border-color: royalblue;
        cursor: pointer;
      }
    }

    .search-fields {
      position: relative;
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 0 1em;
      border: 1px solid #b5b8bb;
      background: #fff;

      span {
        cursor: pointer;
        user-select: none;
      }

      ul {
        position: absolute;
        z-index: 100;
        top: 37px;
        left: 0;
        margin: 0;
        padding: 0;
        min-width: 150px;
        background: #fff;
        border: 1px solid #b5b8bb;
        box-shadow: rgba(#000, 0.1) 0 4px 6px;

        li {
          display: flex;
          padding: 0.5em 1em;

          button {
            margin-right: 10px;
          }
        }
      }
    }

    .search-holder {
      position: relative;

      svg {
        position: absolute;
        width: 20px;
        top: 50%;
        left: 10px;
        transform: translateY(-50%);
      }

      input {
        padding: 0 1em 0 3em;
        height: 35px;
        width: 150px;
        border: 1px solid #b5b8bb;
        letter-spacing: 2px;
        transition-property: border-color, width;
        transition-duration: 0.3s;
        transition-timing-function: cubic-bezier(0.075, 0.82, 0.165, 1);

        &:focus {
          width: 200px;
          outline: none;
          border-color: royalblue;
        }
      }
    }
  }

  .table-holder {
    position: relative;
    width: auto;
    overflow-x: auto;
    border: 1px solid #e9ecef;

    &.full {
      width: 100%;
      > table {
        min-width: 100%;
      }
    }
  }

  table {
    table-layout: auto;
    border-collapse: collapse;
    background: #fff;

    thead {
      tr {
        th {
          padding: 1em 1.5em;
          text-align: left;
          border-bottom: 1px solid #e9ecef;
          font-weight: bold;
          font-size: 0.875rem;
          background: #f8f9fa;
          transition: background 0.3s ease-in-out;

          &.sortable {
            cursor: pointer;

            &:hover {
              background: #f1f5ff;
            }
          }

          &.fixed {
            position: sticky;
            z-index: 100;
          }

          &.scrolled {
            &::after {
              content: "";
              position: absolute;
              right: 0;
              top: 0;
              bottom: 0;
              border-right: 1px solid #e9ecef;
              box-shadow: rgba(#000, 0.1) 0 5px 6px;
            }
          }

          &.selector {
            left: 0;
          }

          .holder {
            display: flex;
            justify-content: space-between;
            align-items: center;

            span {
              user-select: none;
            }

            .actions {
              display: flex;
              justify-content: center;
              align-items: center;

              .sort {
                svg {
                  animation-duration: 0.3s;
                  animation-fill-mode: both;
                  width: 16px;
                  transition: transform 0.3s cubic-bezier(0.075, 0.82, 0.165, 1);
                }

                &.asc {
                  svg {
                    transform: rotate(180deg);
                  }
                }
              }
            }
          }
        }
      }
    }

    tbody {
      tr {
        &.can-expand {
          cursor: pointer;
        }
        td {
          padding: 1em 1.5em;
          text-align: left;
          border-bottom: 1px solid #e9ecef;

          span {
            display: block;
            &.no-wrap {
              min-width: 100%;
              white-space: nowrap;
              overflow: hidden;
              text-overflow: ellipsis;
            }
          }

          &.fixed {
            position: sticky;
            z-index: 100;
          }

          &.scrolled {
            &::after {
              content: "";
              position: absolute;
              right: 0;
              top: 0;
              bottom: 0;
              border-right: 1px solid #e9ecef;
              box-shadow: rgba(#000, 0.1) 0 5px 6px;
            }
          }

          &.selector {
            left: 0;
            background: #fff;
          }
        }

        &.selected {
          td {
            background: #edf6ff;
          }
        }

        &.expander {
          .expanded-icon {
            width: 24px;
            transform: scaleX(-1);
          }

          td {
            > div {
              display: flex;

              .expanded-icon {
                margin-right: 20px;
              }
            }
          }
        }

        &:not(.expander):hover {
          td {
            background: #f6f8fa;
          }
        }
      }
    }
  }

  .info {
    display: flex;
    justify-content: center;
    align-items: center;
    background: #fff;
    padding: 1em 2em;

    .detail {
      h1 {
        font-size: 0.9rem;
      }
    }

    ::v-deep(.paginator) {
      margin: 0 2em;
    }

    .rows-ddl {
      height: 30px;
      min-width: 50px;
      padding: 0 5px;
      font-family: inherit;
    }
  }
}
.fadeInUp {
  animation-name: fadeInUp;
}
.fadeOutUp {
  animation-name: fadeOutUp;
}
@keyframes fadeInUp {
  from {
    opacity: 0;
    -webkit-transform: translate3d(0, 100%, 0);
    transform: translate3d(0, 100%, 0);
  }

  to {
    opacity: 1;
    -webkit-transform: translate3d(0, 0, 0);
    transform: translate3d(0, 0, 0);
  }
}
@keyframes fadeOutUp {
  from {
    opacity: 1;
  }

  to {
    opacity: 0;
    -webkit-transform: translate3d(0, -100%, 0);
    transform: translate3d(0, -100%, 0);
  }
}
</style>