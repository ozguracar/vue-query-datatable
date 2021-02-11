<template>
  <div class="gd-row">
    <div class="gd-lg-12">
      <div class="card p-0">
        <div class="table-wrapper">
          <div class="table-title">
            <h6 class="font-poppins text-link font-weight-500 m-0">
              {{ opts.title }}
            </h6>
            <div class="table-filter">
              <ul>
                <li v-if="undoQuery">
                  <button class="button-item" @click="undo">
                    Undo <i class="ri-arrow-go-back-fill"></i>
                  </button>
                </li>
                <li>
                  <button
                    :class="undoQuery ? 'text-success' : ''"
                    class="button-item"
                    @click="reset"
                  >
                    Reset <i class="ri-refresh-line"></i>
                  </button>
                </li>
                <li>
                  <button @click="json2excel" class="button-item">
                    Export Current <i class="ri-file-chart-line"></i>
                  </button>
                  <button @click="json2excel('all')" class="button-item">
                    Export All <i class="ri-file-chart-line"></i>
                  </button>
                </li>
                <li>
                  <button
                    :class="filter ? 'text-success' : ''"
                    class="button-item"
                    @click="filter = !filter"
                  >
                    Filters <i class="ri-sound-module-line"></i>
                  </button>
                  <ul v-if="filter" class="filters-box">
                    <li class="date">
                      <span
                        class="mr-10 text-dark-blue font-size-13 font-weight-500 font-poppins"
                        >Date:</span
                      >
                      <button
                        v-for="(value, name) in opts.dates"
                        :key="name"
                        :class="selectedDate === name ? 'text-success' : ''"
                        class="bg-none default-select"
                        @click="selectedDate = name"
                      >
                        {{ name }}
                      </button>
                    </li>
                    <li class="date position-relative date-range">
                      <span
                        class="mr-10 text-dark-blue font-size-13 font-weight-500 font-poppins"
                        >Date Range:
                        <DatePicker
                          :key="DatePickerVersion"
                          v-model="dateRange"
                          class="date-picker-filter"
                          is-range
                        >
                          <template
                            v-slot="{ inputEvents }"
                            class="font-size-13 font-poppins"
                          >
                            <input
                              :value="
                                range.start
                                  ? new Date(range.start).toLocaleString()
                                  : null
                              "
                              placeholder="Start Date"
                              readonly
                              v-on="inputEvents.start"
                            />
                            <input
                              :value="
                                range.end
                                  ? new Date(range.end).toLocaleString()
                                  : null
                              "
                              placeholder="Finish Date"
                              readonly
                              v-on="inputEvents.end"
                            />
                          </template>
                        </DatePicker>
                      </span>
                    </li>
                    <li class="date">
                      <span
                        class="mr-10 text-dark-blue font-size-13 font-weight-500 font-poppins"
                        >How Many Rows :</span
                      >
                      <button
                        v-for="n in opts.shownRowNumbers"
                        :key="'filtershownRow' + n"
                        class="bg-none default-select"
                        :class="n === shownRow ? 'text-success' : ''"
                        @click="shownRow = n"
                      >
                        {{ n }}
                      </button>
                    </li>
                    <li class="date">
                      <span
                        class="mr-10 text-dark-blue font-size-13 font-weight-500 font-poppins"
                        >Select Columns:</span
                      >
                      <button
                        class="bg-none default-select text-success"
                        @click="selectAllHeads"
                      >
                        Select All
                      </button>
                      <button
                        v-if="getFilterStatus.reverse"
                        class="bg-none default-select"
                        @click="reserveHeads"
                      >
                        Reverse
                      </button>
                      <ul class="date-columns-group">
                        <li
                          v-for="head in opts.heads"
                          :key="'filter-head-' + head.title"
                        >
                          <button
                            class="btn btn-sm rounded"
                            :class="
                              head.visibility
                                ? 'btn-success-outline'
                                : 'btn-default-outline'
                            "
                            @click="
                              getFilterStatus.selectable || !head.visibility
                                ? (head.visibility = !head.visibility)
                                : ''
                            "
                          >
                            {{ head.title }}
                          </button>
                        </li>
                      </ul>
                    </li>
                    <li
                      v-for="custom in opts.customFilter"
                      :key="'query' + custom.key"
                      class="date"
                    >
                      <span
                        class="mr-10 text-dark-blue font-size-13 font-weight-500 font-poppins"
                        >{{ custom.title }}</span
                      >
                      <ul class="date-columns-group">
                        <li
                          v-for="val in custom.values"
                          :key="'custom-filter-' + custom.key + '-' + val.key"
                        >
                          <button
                            class="btn btn-sm rounded"
                            :class="
                              custom.selected === val.key
                                ? 'btn-success-outline'
                                : ''
                            "
                            @click="setCustomFilter(custom, val)"
                          >
                            {{ val.name }}
                          </button>
                        </li>
                      </ul>
                    </li>
                  </ul>
                </li>
                <li>
                  <button
                    v-if="!search"
                    class="button-item"
                    @click="search = true"
                  >
                    Search <i class="ri-search-line"></i>
                  </button>
                  <div v-if="search" class="search">
                    <input
                      v-model="searchText"
                      type="text"
                      placeholder="Search"
                    />
                    <button class="bg-none">
                      <i
                        class="ri-close-line text-danger"
                        @click="closeSearch"
                      ></i>
                      <i
                        v-if="opts.searchWithButton"
                        class="ri-search-line"
                        @click="setSearch"
                      ></i>
                    </button>
                  </div>
                </li>
              </ul>
            </div>
          </div>
          <div class="table-body">
            <div class="table-responsive">
              <Loader v-if="opts.loading" />
              <table v-if="getRows.length">
                <thead>
                  <tr>
                    <th v-for="(head, name) in getHeads" :key="head.title">
                      {{ head.title }}
                      <span
                        v-if="head.sortable"
                        class="sort-icon"
                        @click="sortTable(name)"
                      >
                        <i
                          v-if="
                            sort.key !== name ||
                            !sort.type ||
                            sort.type === 'asc'
                          "
                          class="ri-arrow-drop-up-fill ri-xl"
                        ></i>
                        <i
                          v-if="
                            sort.key !== name ||
                            !sort.type ||
                            sort.type === 'desc'
                          "
                          class="ri-arrow-drop-down-fill ri-xl"
                        ></i>
                      </span>
                    </th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="row in getRows" :key="row[opts.rowId]">
                    <td
                      v-for="(value, keyName) in getHeads"
                      :key="opts.rowId + keyName"
                    >
                      <slot
                        :name="'column-' + keyName"
                        :column="row[keyName]"
                        :row="row"
                        >{{ row[keyName] }}</slot
                      >
                    </td>
                  </tr>
                </tbody>
              </table>
              <div v-else class="text-center table-blank xs-font-size-13">
                There is no data
              </div>
            </div>
          </div>
          <div class="table-bottom">
            <div class="left">
              <span class="font-size-13 font-poppins text-hoki-grey"
                >Number of rows:</span
              >
              <select v-model="shownRow">
                <option
                  v-for="n in opts.shownRowNumbers"
                  :key="'shownRow' + n"
                  :value="n"
                >
                  {{ n }}
                </option>
              </select>
              <span
                v-if="opts.visibility.totalRows"
                class="font-size-13 xs-font-size-12 font-poppins text-hoki-grey"
              >
                of {{ opts.totalRows }}
              </span>
            </div>
            <div class="center">
              <ul>
                <li v-if="getStartPage.number > 1" class="text-cascade-grey">
                  ...
                </li>
                <li
                  v-for="n in getStartPage.total"
                  :key="'page' + n"
                  :class="
                    n + getStartPage.number - 1 === page
                      ? 'text-dark-blue'
                      : 'text-cascade-grey'
                  "
                  class="cursor-pointer"
                  @click="page = n + getStartPage.number - 1"
                >
                  {{ n + getStartPage.number - 1 }}
                </li>
                <li v-if="page + 5 < totalPages" class="text-cascade-grey">
                  ...
                </li>
              </ul>
            </div>
            <div class="right">
              <span
                :class="page > 1 ? 'text-hoki-grey' : 'text-cascade-grey'"
                @click="page > 1 ? page-- : ''"
                ><i class="ri-arrow-left-s-fill"></i> Previus Page</span
              >
              <span
                :class="
                  page < totalPages ? 'text-hoki-grey' : 'text-cascade-grey'
                "
                @click="page < totalPages ? page++ : ''"
                >Next Page <i class="ri-arrow-right-s-fill"></i
              ></span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import DatePicker from "v-calendar/lib/components/date-picker.umd";
import * as FileSaver from "file-saver";
import * as XLSX from "xlsx";
import Loader from "./Loader";
import "../assets/style.css";
class IWorkBook {
  constructor(sheetNames, sheets) {
    this.SheetNames = sheetNames;
    this.Sheets = sheets;
  }
}
export default {
  name: "Index",
  auth: "guest",
  components: { Loader, DatePicker },
  props: {
    data: {
      type: Object,
      default: () => {
        return { length: 0, rows: [] };
      },
    },
    options: {
      type: Object,
      default: () => {
        return {};
      },
    },
  },
  data() {
    return {
      DatePickerVersion: 0,
      filter: false,
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
        key: null,
        type: null,
        step: 0,
      },
      defaultOptions: {
        mode: "static", // static,ajax
        title: null,
        rowId: "id",
        dateKey: "createdAt",
        visibility: {
          totalRows: true,
          numberOfRows: true,
          pagination: true,
        },
        shownRowNumbers: [10, 20, 30, 40, 50],
        loading: false,
        heads: {},
        dates: {
          Today: 0,
          "3D": 3,
          "1W": 7,
          "1M": 30,
          "1Y": 365,
        },
        customFilter: [],
        searchWithButton: false,
        queryPrefix: "vd",
        exportExcel: {
          status: true,
          currentList: true,
          allData: true,
        },
      },
      undoQuery: null,
      opts: {},
    };
  },
  computed: {
    getFilterStatus() {
      const l = Object.keys(this.opts.heads).length;
      const h = Object.keys(this.getHeads).length;
      return {
        reverse: l - h > 0,
        selectable: l - h < l - 1,
      };
    },
    dateRange: {
      get() {
        if (!this.range.start || !this.range.end) {
          return { start: null, end: null };
        }
        return {
          start: new Date(this.range.start),
          end: new Date(this.range.end),
        };
      },
      set(val) {
        val.start.setHours(0, 0, 0, 0);
        val.end.setHours(23, 59, 59, 999);
        this.range = { start: val.start.getTime(), end: val.end.getTime() };
      },
    },
    getHeads() {
      const heads = {};
      Object.keys(this.opts.heads).forEach((el) => {
        if (this.opts.heads[el].visibility) {
          heads[el] = this.opts.heads[el];
        }
      });
      return heads;
    },
    getRows() {
      if (this.opts.mode === "ajax") {
        return this.data.rows;
      }

      let data = [...this.data.rows];
      if (this.range.start) {
        data = data.filter(
          (x) => new Date(x[this.opts.dateKey]).getTime() >= this.range.start
        );
      }
      if (this.range.end) {
        data = data.filter(
          (x) => new Date(x[this.opts.dateKey]).getTime() <= this.range.end
        );
      }
      if (this.sort.key) {
        if (this.getHeads[this.sort.key].type === "date") {
          data.sort((a, b) => {
            if (
              new Date(a[this.sort.key]).getTime() >
              new Date(b[this.sort.key]).getTime()
            ) {
              return 1;
            }
            if (
              new Date(a[this.sort.key]).getTime() <
              new Date(b[this.sort.key]).getTime()
            ) {
              return -1;
            }
            return 0;
          });
        } else if (this.getHeads[this.sort.key].type === "string") {
          data.sort((a, b) => {
            if (
              a[this.sort.key].toLowerCase() > b[this.sort.key].toLowerCase()
            ) {
              return 1;
            }
            if (
              a[this.sort.key].toLowerCase() < b[this.sort.key].toLowerCase()
            ) {
              return -1;
            }
            return 0;
          });
        } else {
          data.sort((a, b) => {
            return a - b;
          });
        }

        if (this.sort.type === "desc") {
          data.reverse();
        }
      }
      if (this.$route.query[this.opts.queryPrefix + "-search"]) {
        data = data.filter((x) => {
          let check = false;
          Object.entries(this.opts.heads).forEach(([key, value]) => {
            if (
              value.type === "string" &&
              x[key]
                .toLowerCase()
                .includes(
                  this.$route.query[
                    this.opts.queryPrefix + "-search"
                  ].toLowerCase()
                )
            ) {
              check = true;
            }
          });
          return check;
        });
      }

      return data.slice(
        (this.page - 1) * this.shownRow,
        this.page * this.shownRow
      );
    },
    totalPages() {
      return Math.ceil(this.data.length / this.shownRow);
    },
    getStartPage() {
      if (this.totalPages < 11) {
        return {
          total: this.totalPages,
          number: 1,
        };
      } else if (this.page < 6) {
        return {
          total: 11,
          number: 1,
        };
      } else if (this.page + 5 > this.totalPages) {
        return {
          total: 11,
          number: this.totalPages - 10,
        };
      } else {
        return {
          total: 11,
          number: this.page - 5,
        };
      }
    },
  },
  watch: {
    "opts.shownRowNumbers"(val) {
      if (!val.includes(this.shownRow)) {
        this.shownRow = Math.min(...val);
        this.pushQuery("shownRow", this.shownRow);
      }
    },
    shownRow(val, oldVal) {
      const query = { ...this.$route.query };
      query[this.opts.queryPrefix + "-shownRow"] = val;
      query[this.opts.queryPrefix + "-page"] =
        Math.floor((oldVal * (this.page - 1)) / val) + 1;
      this.$router.push({ query }).catch(() => {});
    },
    page(val) {
      this.pushQuery("page", val);
    },
    searchText(val) {
      if (!this.opts.searchWithButton) {
        this.pushQuery("search", val);
      }
    },
    selectedDate(val) {
      const date = new Date(
        Date.now() - this.opts.dates[val] * 24 * 60 * 60 * 1000
      );
      date.setHours(0, 0, 0, 0);
      this.range.start = date.getTime();
      const today = new Date();
      today.setHours(23, 59, 59, 999);
      this.range.end = today.getTime();
      this.DatePickerVersion++;
    },
    range: {
      deep: true,
      immediate: true,
      handler(val) {
        const query = { ...this.$route.query };
        if (!val.start || !val.end) {
          delete query[this.opts.queryPrefix + "-startDate"];
          delete query[this.opts.queryPrefix + "-endDate"];
        } else {
          query[this.opts.queryPrefix + "-startDate"] = val.start;
          query[this.opts.queryPrefix + "-endDate"] = val.end;
        }
        this.$router.push({ query }).catch(() => {});
      },
    },
    sort: {
      deep: true,
      immediate: true,
      handler(val) {
        const query = { ...this.$route.query };
        if (!val.key) {
          delete query[this.opts.queryPrefix + "-sort"];
          delete query[this.opts.queryPrefix + "-sortType"];
        } else {
          query[this.opts.queryPrefix + "-sort"] = val.key;
          query[this.opts.queryPrefix + "-sortType"] = val.type;
        }
        this.$router.push({ query }).catch(() => {});
      },
    },
    "$route.query": {
      deep: true,
      immediate: true,
      handler() {
        if (this.timeout) {
          clearTimeout(this.timeout);
        }
        this.timeout = setTimeout(() => {
          this.reload();
          this.timeout = setTimeout(() => {
            this.$emit("refresh", this.$route.query);
          }, 250);
        }, 250);
      },
    },
    options: {
      deep: true,
      immediate: true,
      handler() {
        this.setOpts();
      },
    },
  },
  created() {
    this.setOpts();
    this.reload();
  },
  methods: {
    formatData(data) {
      this.$emit("formatData", {
        type: "all",
        data: JSON.parse(JSON.stringify(data)),
        heads: this.getHeads,
        exportData: this.exportExcel,
      });
      this.opts.loading = false;
    },
    json2excel(type) {
      if (type !== "all") {
        this.$emit("formatData", {
          type: "current",
          data: JSON.parse(JSON.stringify(this.getRows)),
          heads: this.getHeads,
          exportData: this.exportExcel,
        });
      } else {
        if (this.opts.mode === "ajax") {
          this.opts.loading = true;
          this.$emit("getAllData", {
            type: "all",
            heads: this.getHeads,
            set: this.formatData,
          });
        } else {
          this.$emit("formatData", {
            type: "all",
            data: JSON.parse(JSON.stringify(this.data)),
            heads: this.getHeads,
            exportData: this.exportExcel,
          });
        }
      }
    },
    exportExcel(opts) {
      const { data = [], name = "excel", formateDate = "dd/mm/yyyy" } = opts;

      const fileNames = [];
      const sheets = {};

      const ws = XLSX.utils.json_to_sheet(data, { dateNF: formateDate });

      fileNames.push(name);
      sheets[name] = ws;

      const wb = new IWorkBook(fileNames, sheets);
      console.log(wb);
      const wbout = XLSX.write(wb, {
        bookType: "xlsx",
        bookSST: true,
        type: "binary",
      });
      const buf = new ArrayBuffer(wbout.length);
      const view = new Uint8Array(buf);

      for (let i = 0; i !== wbout.length; ++i) {
        view[i] = wbout.charCodeAt(i) & 0xff;
      }
      FileSaver.saveAs(
        new Blob([buf], { type: "application/octet-stream" }),
        name + ".xlsx"
      );
    },
    setOpts() {
      const opts = JSON.parse(
        JSON.stringify({
          ...this.defaultOptions,
          ...this.options,
          visibility: {
            ...this.defaultOptions.visibility,
            ...this.options.visibility,
          },
          exportExcel: {
            ...this.defaultOptions.exportExcel,
            ...this.options.exportExcel,
          },
        })
      );
      opts.customFilter.forEach((element) => {
        element.selected = null;
      });
      this.opts = opts;
      this.opts.customFilter.forEach((element) => {
        if (this.$route.query[element.key]) {
          element.selected = this.$route.query[element.key];
        }
      });
    },
    sortTable(name) {
      if (this.sort.key === name) {
        if (this.sort.step === 2) {
          this.sort = {
            key: null,
            type: null,
            step: 0,
          };
          return;
        }
        if (this.sort.type === "asc") {
          this.sort = {
            key: name,
            type: "desc",
            step: this.sort.step + 1,
          };
        } else {
          this.sort = {
            key: name,
            type: "asc",
            step: this.sort.step + 1,
          };
        }
      } else {
        this.sort = {
          key: name,
          type: "asc",
          step: 1,
        };
      }
    },
    selectAllHeads() {
      Object.values(this.opts.heads).forEach((value) => {
        value.visibility = true;
      });
    },
    reserveHeads() {
      Object.values(this.opts.heads).forEach((value) => {
        value.visibility = !value.visibility;
      });
    },
    closeSearch() {
      this.searchText = null;
      this.search = false;
      this.removeQuery("search");
    },
    setSearch() {
      if (this.searchText !== this.$route.query.search) {
        this.pushQuery("search", this.searchText);
      }
    },
    reload() {
      const query = {};
      Object.keys(this.$route.query).forEach((el) => {
        if (el.startsWith(this.opts.queryPrefix + "-")) {
          query[
            el.substring((this.opts.queryPrefix + "-").length)
          ] = this.$route.query[el];
        }
      });
      if (
        query.shownRow &&
        this.opts.shownRowNumbers.includes(Number(query.shownRow))
      ) {
        if (this.shownRow !== Number(query.shownRow)) {
          this.start = true;
          this.shownRow = Number(query.shownRow);
        }
      } else {
        this.shownRow = Math.min(...this.opts.shownRowNumbers);
        if (!query.shownRow) {
          this.start = true;
          this.pushQuery("shownRow", this.shownRow);
        }
      }
      if (query.page && Number(query.page) <= this.totalPages) {
        if (this.page !== Number(query.page)) {
          this.start = true;
          this.page = Number(query.page);
        }
      } else {
        this.page = 1;
        if (!query.page) {
          this.start = true;
          this.pushQuery("page", this.page);
        }
      }
      if (query.startDate) {
        if (this.range.start !== Number(query.startDate)) {
          this.start = true;
          this.range.start = +query.startDate;
        }
      }
      if (query.endDate) {
        if (this.range.end !== Number(query.endDate)) {
          this.start = true;
          this.range.end = +query.endDate;
        }
      }
      if (query.sort) {
        if (this.sort.key !== query.sort) {
          this.start = true;
          if (
            !this.getHeads[query.sort] ||
            !this.getHeads[query.sort].sortable
          ) {
            const query = { ...this.$route.query };
            delete query[this.opts.queryPrefix + "-sort"];
            delete query[this.opts.queryPrefix + "-sortType"];
            this.$router.push({ query }).catch(() => {});
          } else {
            this.sort = {
              key: query.sort,
              type: query.sortType ? query.sortType : "asc",
              step: 1,
            };
          }
        }
      }
      if (query.search) {
        this.start = true;
        this.searchText = query.search;
        this.search = true;
      } else {
        this.start = true;
        this.searchText = null;
      }
      if (!this.start) {
        this.start = true;
        this.$emit("refresh", this.$route.query);
      }
    },
    reset() {
      this.undoQuery = JSON.parse(JSON.stringify(this.$route.query));
      Object.keys(this.undoQuery).forEach((element) => {
        if (!element.startsWith(this.opts.queryPrefix + "-")) {
          delete this.undoQuery[element];
        }
      });
      const query = JSON.parse(JSON.stringify(this.$route.query));
      Object.keys(query).forEach((element) => {
        if (element.startsWith(this.opts.queryPrefix + "-")) {
          delete query[element];
        }
      });
      this.$router.push({ query }).catch(() => {});
    },
    undo() {
      const query = JSON.parse(JSON.stringify(this.$route.query));
      Object.keys(query).forEach((element) => {
        if (element.startsWith(this.opts.queryPrefix + "-")) {
          delete query[element];
        }
      });
      this.$router
        .push({ query: { ...query, ...this.undoQuery } })
        .catch(() => {});
      this.undoQuery = null;
    },
    setCustomFilter(custom, value) {
      if (this.$route.query[custom.key]) {
        if (this.$route.query[custom.key] === value.key) {
          custom.selected = null;
          this.removeQuery(custom.key, false);
        } else {
          custom.selected = value.key;
          this.pushQuery(custom.key, value.key, false);
        }
      } else {
        custom.selected = value.key;
        this.pushQuery(custom.key, value.key, false);
      }
    },
    pushQuery(name, value, prefix = true) {
      // eslint-disable-next-line eqeqeq
      if (this.$route.query[this.opts.queryPrefix + "-" + name] == value) {
        return;
      }
      const query = { ...this.$route.query };
      if (prefix) {
        query[this.opts.queryPrefix + "-" + name] = value;
      } else {
        query[name] = value;
      }
      this.$router.push({ query }).catch(() => {});
    },
    removeQuery(name, prefix = true) {
      const query = { ...this.$route.query };
      if (prefix) {
        delete query[this.opts.queryPrefix + "-" + name];
      } else {
        delete query[name];
      }
      this.$router.push({ query }).catch(() => {});
    },
  },
};
</script>
