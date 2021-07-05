<template>
  <DataTable
    :value="deals"
    :rows="10"
    :globalSearchFields="['name', 'issuer', 'industry']"
    removableSort
    selectable
    :rowsPerPageOptions="[10, 15, 20, 30, 40, 50]"
  >
    <Column field="name" header="Name" sortable :wrap="false" width="350px" />
    <Column field="issuer" header="Issuer" sortable :wrap="false" width="350px" />
    <Column field="industry" header="Industry" sortable :wrap="false" width="200px" />
    <Column field="agent" header="Agent" width="200px" />
    <Column field="source" header="Source" />
    <Column field="type" header="Type" />
    <template #expander="{ data: {docs} }">
      <DataTable
        :value="docs"
        :rows="5"
        removableSort
        :fullWidth="false"
        :style="{ padding: 0, alignItems: 'flex-start' }"
      >
        <Column field="name" header="Name" sortable :wrap="false" width="350px" />
        <Column
          field="size"
          header="File Size"
          sortable
          :wrap="false" width="350px"
          #default="{ item: { size } }"
        >
          {{ `${(size/1024).toFixed(2)}MB` }}
        </Column>
        <Column field="status" header="Status" sortable :wrap="false" width="350px" />
      </DataTable>
    </template>
  </DataTable>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { DataTable, Column } from "./components/data-table";
import { data as deals_json } from "./assets/deals_dataset.json";
import { data as docs_json } from "./assets/docs_dataset.json";

export default defineComponent({
  name: "App",
  components: {
    DataTable,
    Column,
  },
  setup() {
    // format data
    const { Holdings, Industries, ClientIssuers, Agents, Sources, DealTypes } =
      deals_json;
    const deals = Holdings.map(
      ({ DealName, IssuerId, IndustryId, AgentId, SourceId, TypeId, Id }) => {
        const { IssuerName } =
          ClientIssuers.find((issuer) => issuer.IssuerId === IssuerId) || {};
        const { IndustryName } =
          Industries.find((industry) => industry.Id === IndustryId) || {};
        const { CompanyName } =
          Agents.find((agent) => agent.Id === AgentId) || {};
        const { SourceName } =
          Sources.find((source) => source.Id === SourceId) || {};
        const { TypeName } = DealTypes.find((type) => type.Id === TypeId) || {};
        const docs = docs_json.docs
          .filter((({ holding_id }) => holding_id === Id))
          .map(({ name, blob_size, status_name }) => ({ name, size: blob_size, status: status_name }))
        return {
          name: DealName,
          issuer: IssuerName,
          industry: IndustryName,
          agent: CompanyName,
          source: SourceName,
          type: TypeName,
          docs
        };
      }
    );
    return {
      deals,
    };
  },
});
</script>

<style>
html,
body {
  min-height: 100%;
  background: #f4f9fc;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
