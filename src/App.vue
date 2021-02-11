<template>
  <div>
    <Datatable
      :data="data"
      :options="options"
      :loading="loading"
      @formatData="formatData($event)"
      @getAllData="getAllData"
      @refresh="refreshData($event)"
    >
      <template #column-name="{ column, row }">
        {{ row.surname }} {{ column }}
      </template>
      <template #column-createdAt="{ column }">
        {{ new Date(column).toLocaleString() }}
      </template>
    </Datatable>
  </div>
</template>
<script>
// eslint-disable-next-line vue/no-parsing-error
import data from "./assets/data.json";
import Datatable from "./components/Datatable.vue";
export default {
  components: { Datatable },
  data() {
    return {
      data: {
        length: data.length,
        rows: data.docs,
      },
      loading: false,
      options: {
        heads: {
          name: {
            title: "Name",
            visibility: true,
            sortable: true,
            type: "string",
          },
          surname: {
            title: "Surname",
            visibility: true,
            sortable: false,
            type: "string",
          },
          email: {
            title: "Email",
            visibility: true,
            sortable: true,
            type: "string",
          },
          createdAt: {
            title: "Register Date",
            visibility: true,
            sortable: true,
            type: "date",
          },
        },
        customFilter: [
          {
            title: "Status",
            key: "status",
            values: [
              { name: "Rejected", key: "rejected" },
              { name: "Process", key: "process" },
              { name: "Pending", key: "pending" },
              { name: "Completed", key: "completed" },
              { name: "Waiting", key: "waiting" },
            ],
          },
          {
            title: "Type",
            key: "type",
            values: [
              { name: "Rejected", key: "rejected" },
              { name: "Process", key: "process" },
              { name: "Pending", key: "pending" },
              { name: "Completed", key: "completed" },
              { name: "Waiting", key: "waiting" },
            ],
          },
        ],
      },
    };
  },
  methods: {
    formatData({ data, heads, exportData, type }) {
      const newData = data.map((el) => {
        const d = {};
        Object.entries(heads).forEach(([key, value]) => {
          d[value.title] = el[key];
        });
        return d;
      });
      exportData({
        data: newData,
        name: type === "all" ? "all" : "current",
        formateDate: "dd/mm/yyyy",
      });
    },
    getAllData({ heads, set }) {
      console.log(heads);
      this.loading = true;
      setTimeout(() => {
        set(data.docs);
        this.loading = false;
      }, 2000);
    },
    refreshData(query) {
      console.log(query);
      // startDate
      // endDate
      // sort
      // sortType
      // shownRow
      // page
      // search
    },
  },
};
</script>
