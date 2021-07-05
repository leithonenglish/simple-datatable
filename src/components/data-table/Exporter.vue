<template>
  <div class="exporter">
    <div class="inner">
      <h1>Export Data</h1>
      <RadioButton v-model="selectedExportType" :options="exportTypes" />
      <ul>
        <li v-for="{ field, header } in columns" :key="field">
          <Checkbox
            :modelValue="isFieldSelected(field)"
            @change="onFieldSelected(field)"
          />
          {{ header }}
        </li>
      </ul>
      <div class="actions">
        <button
          :disabled="selectedFields.length < 1"
          @click.prevent="exportData"
        >
          Export
        </button>
        <button @click.prevent="onCancel" class="cancel">Cancel</button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, PropType, ref } from 'vue';
import Checkbox from '../Checkbox.vue';
import RadioButton from '../RadioButton.vue';

export default defineComponent({
  name: 'Exporter',
  emits: ['cancelled', 'exported'],
  components: {
    Checkbox,
    RadioButton
  },
  props: {
    columns: {
      type: Array as PropType<any[]>,
      required: true
    },
    data: {
      type: Array as PropType<any[]>,
      required: true
    }
  },
  setup({ data }, { emit }) {
    const exportTypes = [
      { value: 'csv', label: 'CVS' },
      { value: 'json', label: 'JSON' }
    ];
    const selectedExportType = ref('csv');
    const selectedFields = ref([] as string[]);
    const isCSV = computed(() => selectedExportType.value === 'csv');
    const selectedData  = computed(() => data.map((item: any) => {
      let obj = {};
      selectedFields.value.forEach(field => {
        obj = { ...obj, ...{ [field]: item[field] } };
      });
      return obj;
    }));
    const exportData = () => {
      const encodedUri = isCSV.value ? generateCSV() : generateJSON();
      var downloadAnchorNode = document.createElement('a');
      downloadAnchorNode.setAttribute("href", encodedUri);
      downloadAnchorNode.setAttribute("download", `data.${isCSV.value ? 'csv' : 'json'}`);
      document.body.appendChild(downloadAnchorNode);
      downloadAnchorNode.click();
      downloadAnchorNode.remove();
    };

    const generateCSV = () => {
      const fields = selectedFields.value.join(";")
      const csvRows = selectedData.value
        .map((item: any) => Object.values(item))
        .map((item: any) => item.join(";")).join("\n");
      return encodeURI(`data:text/csv;charset=utf-8,${fields}\n${csvRows}`);
    }

    const generateJSON = () => {
      const dataString = JSON.stringify(selectedData.value);
      const csvContent = `data:text/json;charset=utf-8,${dataString}`;
      return encodeURI(csvContent);
    }

    const onFieldSelected = (field: string) => {
      const index = selectedFields.value.indexOf(field);
      if (index > -1) {
        selectedFields.value.splice(index, 1);
      } else {
        selectedFields.value.push(field);
      }
    }

    const isFieldSelected = (field: string) => {
      return selectedFields.value.indexOf(field) > -1;
    }

    const onCancel = () => {
      selectedFields.value = [];
      emit('cancelled');
    }

    return {
      exportTypes,
      selectedExportType,
      selectedFields,
      exportData,
      isFieldSelected,
      onFieldSelected,
      onCancel
    }
  },
})
</script>

<style lang="scss" scoped>
.exporter {
  position: absolute;
  z-index: 100;
  right: -2px;
  top: 0;
  bottom: 0;
  left: 0;
  background: rgba(#000, 0.1);

  .inner {
    display: flex;
    flex-direction: column;
    position: absolute;
    right: 0;
    top: 0;
    bottom: 0;
    width: 300px;
    background: #fff;

    ::v-deep(.radio-btn) {
      button {
        border-radius: 0;

        &:first-child {
          border-left: none;
        }

        &:last-child {
          border-right: none;
        }
      }
    }

    h1 {
      flex: 0 0 auto;
      padding: 0.5em 1em;
      margin: 0;
      font-size: 1.5em;
    }

    ul {
      flex: 1 1 auto;
      list-style: none;
      margin: 0;
      padding: 0;
      overflow-y: auto;

      li {
        display: flex;
        justify-content: flex-start;
        align-items: center;
        padding: 1em 1.5em;
        border-bottom: 1px solid #e9e9e9;

        button {
          margin-right: 10px;
        }

        &:last-child {
          margin: 0;
        }
      }
    }

    .actions {
      display: flex;
      flex: 0 1 auto;
      padding: 1em;
      justify-content: center;
      align-items: center;
      border-top: 1px solid #ddd;

      button {
        padding: 0.5em 1em;
        margin: 0 0.2em;
        min-width: 80px;
        font-family: inherit;
        font-weight: bold;
        font-size: 0.9rem;
        color: #1f1f1f;
        border: none;
        border-radius: 2px;
        background: rgba(green, 0.2);
        transition: background 0.3s ease-out;

        &:disabled {
          background: #e9e9e9;
          color: #acacac
        }

        &:not(:disabled):hover {
          cursor: pointer;
          background: rgba(green, 0.5);
        }

        &.cancel {
          background: none;
          color: rgba(red, 0.7);

          &:hover {
            color: rgba(red, 0.9);
          }
        }
      }
    }
  }
}
</style>