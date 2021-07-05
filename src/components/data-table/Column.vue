<script lang="ts">
import { defineComponent } from "vue";
export default defineComponent({
  name: "Column",
  props: {
    field: {
      type: String,
    },
    header: {
      type: String,
      required: true,
    },
    sortable: {
      type: Boolean,
      default: false,
    },
    wrap: {
      type: Boolean,
      default: true,
    },
    fixed: {
      type: Boolean,
      default: false,
    },
    width: {
      type: String,
    },
  },
  render() {
    return null;
  },
});
</script>