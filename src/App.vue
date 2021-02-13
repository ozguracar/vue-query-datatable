<template>
  <div>
    <Datatable
      :data="data"
      :options="options"
      :loading="loading"
      :searchData="searchData"
      @formatData="formatData($event)"
      @getAllData="getAllData"
      @refresh="refreshData($event)"
      @search="search($event)"
    >
      <template #search="{ data, pushQuery, closeSearch }">
        <span @click="pushQuery('user', data.id)"
          >{{ data }} <span @click="closeSearch">x</span></span
        >
      </template>
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
      searchData: [],
      texts: {
        Reset: "@",
        Exports: "@",
        Filters: "@",
        "Number of rows": "@",
        "Previus Page": "@",
        "Next Page": "@",
        Undo: "@",
        "Export Current Data": "@",
        "Export All Data": "@",
        Date: "@",
        "Date Range": "@",
        "How Many Rows": "@",
        "Select Columns": "@",
        "Select All": "@",
        "Start Date": "@",
        "Finish Date": "@",
        Search: "@",
      },
      options: {
        mode: "static",
        defaults: {
          search: false,
          searchText: null,
          shownRow: 10,
          page: 1,
          selectedDate: null,
          range: {
            start: null,
            end: null,
          },
          sort: {
            key: "createdAt",
            type: "asc",
            step: 1,
          },
        },
        heads: {
          name: {
            title: "Name",
            visibility: true,
            sortable: true,
            type: "string",
            width: "20%",
          },
          surname: {
            title: "Surname",
            visibility: true,
            sortable: false,
            type: "string",
            width: "20%",
          },
          email: {
            title: "Email",
            visibility: true,
            sortable: true,
            type: "string",
            width: "20%",
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
        customSearch: true,
      },
    };
  },
  methods: {
    search(val) {
      if (!val) {
        this.searchData = [];
        return;
      }
      this.searchData = data.docs.filter((x) => {
        let check = false;
        Object.entries(this.options.heads).forEach(([key, value]) => {
          if (value.type === "string" && x[key].toLowerCase().includes(val)) {
            check = true;
          }
        });
        return check;
      });
    },
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
    refreshData() {
      // console.log(query);
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
