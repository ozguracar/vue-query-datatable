<template>
  <div class="table-wrapper">
    <div class="table-title">
      <h6>{{ opts.title }}</h6>
      <div class="table-filter">
        <button class="bg-none table-menu" @click="mobileMenu = !mobileMenu">
          <i class="ri-menu-line"></i>
        </button>
        <ul v-if="mobileMenu" class="list-propery">
          <li v-if="opts.visibility.undoButton && undoQuery">
            <button class="button-item" @click="undo">
              {{ locale("Undo") }} <i class="ri-arrow-go-back-fill"></i>
            </button>
          </li>
          <!-- <li
            v-if="
              opts.visibility.forwardButton && queryIndex < undoQueries.length
            "
          >
            <button class="button-item" @click="reset('forward')">
              Forward <i class="ri-arrow-go-forward-fill"></i>
            </button>
          </li> -->
          <li v-if="opts.visibility.resetButton">
            <button class="button-item" @click="reset">
              {{ locale("Reset") }} <i class="ri-refresh-line"></i>
            </button>
          </li>
          <li
            v-if="
              opts.visibility.exportCurrentList || opts.visibility.exportAllData
            "
            class="exports-box"
          >
            <button
              @click="exports = !exports"
              :class="exports ? 'text-success' : ''"
              class="button-item"
            >
              {{ locale("Exports") }} <i class="ri-file-chart-line"></i>
            </button>
            <div
              v-if="exports"
              @mouseleave="exports = false"
              class="exports-box-submenu"
            >
              <button
                v-if="opts.visibility.exportCurrentList"
                @click="json2excel"
                class="button-item"
              >
                <i class="ri-file-chart-line"></i>
                {{ locale("Export Current Data") }}
              </button>
              <button
                v-if="opts.visibility.exportAllData"
                @click="json2excel('all')"
                class="button-item"
              >
                <i class="ri-file-chart-line"></i
                >{{ locale("Export All Data") }}
              </button>
            </div>
          </li>
          <li v-if="opts.visibility.filter">
            <button
              :class="filter ? 'text-success' : ''"
              class="button-item"
              @click="filter = !filter"
            >
              {{ locale("Filters") }} <i class="ri-sound-module-line"></i>
            </button>
            <ul v-if="filter" @mouseleave="filter = false" class="filters-box">
              <li @click="filter = false" class="filter-close">
                <i class="ri-close-circle-line"></i>
              </li>
              <li v-if="opts.visibility.dates" class="date">
                <span class="filter-title">{{ locale("Date") }}: </span>
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
              <li
                v-if="opts.visibility.datepicker"
                class="date position-relative date-range"
              >
                <span class="filter-title d-flex"
                  >{{ locale("Date Range") }}:
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
                        :placeholder="locale('Start Date')"
                        readonly
                        v-on="inputEvents.start"
                      />
                      <input
                        :value="
                          range.end
                            ? new Date(range.end).toLocaleString()
                            : null
                        "
                        :placeholder="locale('Finish Date')"
                        readonly
                        v-on="inputEvents.end"
                      />
                    </template>
                  </DatePicker>
                </span>
              </li>
              <li v-if="opts.visibility.selectShownRow" class="date">
                <span class="filter-title"
                  >{{ locale("How Many Rows") }}:
                </span>
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
              <li v-if="opts.visibility.columns" class="date">
                <span class="filter-title"
                  >{{ locale("Select Columns") }}:
                </span>
                <button class="bg-none selector" @click="selectAllHeads">
                  {{ locale("Select All") }}
                </button>
                <button
                  v-if="getFilterStatus.reverse"
                  class="bg-none selector default-select"
                  @click="reserveHeads"
                >
                  {{ locale("Reverse") }}
                </button>
                <ul class="date-columns-group">
                  <li
                    v-for="head in opts.heads"
                    :key="'filter-head-' + head.title"
                  >
                    <button
                      class="btn"
                      :class="head.visibility ? 'btn-success' : 'btn-default'"
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
                <span class="filter-title">{{ custom.title }} </span>
                <ul class="date-columns-group">
                  <li
                    v-for="val in custom.values"
                    :key="'custom-filter-' + custom.key + '-' + val.key"
                  >
                    <button
                      class="btn"
                      :class="
                        custom.selected === val.key
                          ? 'btn-success'
                          : 'btn-default'
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
          <li v-if="opts.visibility.search" class="search-box">
            <button v-if="!search" class="button-item" @click="search = true">
              {{ locale("Search") }} <i class="ri-search-line"></i>
            </button>
            <div v-else class="search">
              <input
                v-model="searchText"
                type="text"
                :placeholder="locale('Search')"
              />
              <button class="bg-none">
                <i class="ri-close-line text-danger" @click="closeSearch"></i>
                <i
                  v-if="opts.searchWithButton"
                  class="ri-search-line"
                  @click="setSearch"
                ></i>
              </button>
            </div>
            <ul v-if="searchData.length && search" class="filters-box">
              <li
                v-for="(data, index) in searchData"
                :key="'search-' + index"
                class="date"
              >
                <slot
                  name="search"
                  :data="data"
                  :index="index"
                  :pushQuery="pushQuery"
                  :removeQuery="removeQuery"
                  :closeSearch="closeSearch"
                ></slot>
              </li>
            </ul>
          </li>
        </ul>
      </div>
    </div>
    <div class="table-body">
      <div class="table-responsive">
        <Loader v-if="loading" />
        <table v-if="getRows.length">
          <thead v-if="opts.visibility.heads">
            <tr>
              <th
                v-for="(head, name) in getHeads"
                :key="head.title"
                :width="head.width"
              >
                {{ head.title }}
                <span
                  v-if="head.sortable"
                  class="sort-icon"
                  @click="sortTable(name)"
                >
                  <i
                    v-if="
                      sort.key !== name || !sort.type || sort.type === 'asc'
                    "
                    class="ri-arrow-drop-up-fill ri-xl"
                  ></i>
                  <i
                    v-if="
                      sort.key !== name || !sort.type || sort.type === 'desc'
                    "
                    class="ri-arrow-drop-down-fill ri-xl"
                  ></i>
                </span>
              </th>
            </tr>
          </thead>
          <tbody>
            <slot name="before-tr"></slot>
            <tr v-for="(row, index) in getRows" :key="row[opts.rowId]">
              <td
                v-for="(value, keyName) in getHeads"
                :key="opts.rowId + keyName"
              >
                <slot
                  :name="'column-' + keyName"
                  :column="row[keyName]"
                  :row="row"
                  :index="index"
                  >{{ row[keyName] }}</slot
                >
              </td>
            </tr>
            <slot name="after-tr"></slot>
          </tbody>
        </table>
        <div v-else class="text-center table-blank">
          {{ locale("There is no data") }}
        </div>
      </div>
    </div>
    <div v-if="opts.visibility.footer" class="table-bottom">
      <div v-if="opts.visibility.shownRow" class="left">
        <span class="rows-number">{{ locale("Number of rows") }}:</span>
        <select v-model="shownRow">
          <option
            v-for="n in opts.shownRowNumbers"
            :key="'shownRow' + n"
            :value="n"
          >
            {{ n }}
          </option>
        </select>
        <span v-if="opts.visibility.totalRows" class="of-rows">
          of {{ data.length }}
        </span>
      </div>
      <div v-if="opts.visibility.pagination" class="center">
        <ul>
          <li v-if="getStartPage.number > 1" class="text-cascade-grey">...</li>
          <li
            v-for="n in getStartPage.total"
            :key="'page' + n"
            :class="
              n + getStartPage.number - 1 === page
                ? 'active'
                : 'text-cascade-grey'
            "
            class="cursor-pointer"
            @click="page = n + getStartPage.number - 1"
          >
            {{ n + getStartPage.number - 1 }}
          </li>
          <li v-if="page + 5 < totalPages" class="text-cascade-grey">...</li>
        </ul>
      </div>
      <div v-if="opts.visibility.pagination" class="right">
        <span
          :class="page > 1 ? 'text-hoki-grey' : 'text-cascade-grey'"
          @click="page > 1 ? page-- : ''"
          ><i class="ri-arrow-left-s-fill"></i>
          {{ locale("Previus Page") }}</span
        >
        <span
          :class="page < totalPages ? 'text-hoki-grey' : 'text-cascade-grey'"
          @click="page < totalPages ? page++ : ''"
          >{{ locale("Next Page") }} <i class="ri-arrow-right-s-fill"></i
        ></span>
      </div>
    </div>
  </div>
</template>
<script>
import DatePicker from "v-calendar/lib/components/date-picker.umd";
import * as FileSaver from "file-saver";
import * as XLSX from "xlsx";
import Loader from "./Loader";
import "../assets/fonts/remixicon.css";
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
    searchData: {
      type: Array,
      default: () => {
        return [];
      },
    },
    options: {
      type: Object,
      default: () => {
        return {};
      },
    },
    texts: {
      type: Object,
      default: () => {
        return {};
      },
    },
    loading: {
      type: Boolean,
      default: () => {
        return false;
      },
    },
  },
  data() {
    return {
      DatePickerVersion: 0,
      mobileMenu: true,
      exports: false,
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
          undoButton: true,
          forwardButton: true,
          resetButton: true,
          exportCurrentList: true,
          exportAllData: true,
          filter: true,
          dates: true,
          datepicker: true,
          selectShownRow: true,
          columns: true,
          search: true,
          heads: true,
          footer: true,
          shownRow: true,
          totalRows: true,
          pagination: true,
        },
        shownRowNumbers: [10, 20, 30, 40, 50],
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
            key: null,
            type: null,
            step: 0,
          },
        },
      },
      undoQuery: null,
      opts: {},
      query: { ...this.$route.query },
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
        if (this.opts.heads[this.sort.key].type === "date") {
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
        } else if (this.opts.heads[this.sort.key].type === "string") {
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
      if (this.query[this.opts.queryPrefix + "-search"]) {
        data = data.filter((x) => {
          let check = false;
          Object.entries(this.opts.heads).forEach(([key, value]) => {
            if (
              value.type === "string" &&
              x[key]
                .toLowerCase()
                .includes(
                  this.query[this.opts.queryPrefix + "-search"].toLowerCase()
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
      }
    },
    shownRow(val, oldVal) {
      this.pushQuery("shownRow", val);
      this.page = Math.floor((oldVal * (this.page - 1)) / val) + 1;
    },
    page(val) {
      this.pushQuery("page", val);
    },
    searchText(val) {
      if (this.opts.customSearch) {
        if (!this.opts.searchWithButton) {
          this.$emit("search", val);
        }
      } else if (!this.opts.searchWithButton) {
        this.pushQuery("search", val);
        this.page = 1;
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
        if (!val.start || !val.end) {
          this.removeQuery("startDate");
          this.removeQuery("endDate");
        } else {
          this.pushQuery("startDate", val.start);

          this.pushQuery("endDate", val.end);
          this.page = 1;
        }
      },
    },
    sort: {
      deep: true,
      immediate: true,
      handler(val) {
        if (!val.key) {
          this.removeQuery("sort");
          this.removeQuery("sortType");
        } else {
          this.pushQuery("sort", val.key);
          this.pushQuery("sortType", val.type ? val.type : "asc");
        }
      },
    },
    query: {
      deep: true,
      immediate: true,
      handler() {
        if (this.compareQuery(this.$route.query, this.query)) {
          return;
        }
        if (this.timeout) {
          clearTimeout(this.timeout);
        }
        this.reload();
        this.timeout = setTimeout(() => {
          if (this.firstTime) {
            const query = { ...this.$route.query, ...this.query };
            this.$router.push({
              query,
            });
            Object.entries(query).forEach(([key, value]) => {
              if (!this.query[key]) {
                this.pushQuery(key, value, false);
              }
            });
            this.$emit("refresh", query);
          } else {
            this.firstTime = true;
          }
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
    Object.entries(this.opts.defaults).forEach(([key, value]) => {
      if (value && typeof value === "object") {
        Object.entries(value).forEach(([key2, value2]) => {
          if (this[key][key2] != value2) {
            this[key][key2] = value2;
          }
        });
        return;
      }
      if (this[key] != value) {
        this[key] = value;
      }
    });
    if (!this.firstTime) {
      Object.entries({ ...this.$route.query, ...this.query }).forEach(
        ([key, value]) => {
          if (!this.query[key]) {
            this.pushQuery(key, value, false);
          }
        }
      );
    }
    setTimeout(() => {
      this.firstTime = true;
    }, 2000);
  },
  methods: {
    locale(text) {
      return this.texts[text] || text;
    },
    compareQuery(o1, o2) {
      for (let p in o1) {
        // eslint-disable-next-line no-prototype-builtins
        if (o1.hasOwnProperty(p)) {
          if (o1[p] !== o2[p]) {
            return false;
          }
        }
      }
      for (let p in o2) {
        // eslint-disable-next-line no-prototype-builtins
        if (o2.hasOwnProperty(p)) {
          if (o1[p] !== o2[p]) {
            return false;
          }
        }
      }
      return true;
    },
    formatData(data) {
      this.$emit("formatData", {
        type: "all",
        data: JSON.parse(JSON.stringify(data)),
        heads: this.getHeads,
        exportData: this.exportExcel,
      });
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
          this.$emit("getAllData", {
            type: "all",
            heads: this.getHeads,
            set: this.formatData,
          });
        } else {
          this.$emit("formatData", {
            type: "all",
            data: JSON.parse(JSON.stringify(this.data.rows)),
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
          defaults: {
            ...this.defaultOptions.defaults,
            ...this.options.defaults,
            range: {
              ...this.defaultOptions.defaults.range,
              ...this.options.defaults.range,
            },
            sort: {
              ...this.defaultOptions.defaults.sort,
              ...this.options.defaults.sort,
            },
          },
        })
      );
      opts.customFilter.forEach((element) => {
        element.selected = null;
      });
      this.opts = opts;
      this.opts.customFilter.forEach((element) => {
        if (this.query[element.key]) {
          element.selected = this.query[element.key];
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
      if (this.opts.customSearch) {
        this.$emit("search", this.searchText);
      } else if (this.searchText !== this.query.search) {
        this.pushQuery("search", this.searchText);
      }
    },
    reload() {
      const query = {};
      Object.keys(this.query).forEach((el) => {
        if (el.startsWith(this.opts.queryPrefix + "-")) {
          query[
            el.substring((this.opts.queryPrefix + "-").length)
          ] = this.query[el];
        }
      });
      if (
        query.shownRow &&
        this.opts.shownRowNumbers.includes(Number(query.shownRow))
      ) {
        if (this.shownRow !== Number(query.shownRow)) {
          this.shownRow = Number(query.shownRow);
        }
      } else {
        this.shownRow = Math.min(...this.opts.shownRowNumbers);
      }
      if (query.page && Number(query.page) <= this.totalPages) {
        if (this.page !== Number(query.page)) {
          this.page = Number(query.page);
        }
      } else {
        this.page = 1;
      }
      if (query.startDate) {
        if (this.range.start !== Number(query.startDate)) {
          this.range.start = +query.startDate;
        }
      }
      if (query.endDate) {
        if (this.range.end !== Number(query.endDate)) {
          this.range.end = +query.endDate;
        }
      }
      if (query.sort) {
        if (this.sort.key !== query.sort) {
          if (
            !this.opts.heads[query.sort] ||
            !this.opts.heads[query.sort].sortable
          ) {
            this.removeQuery("sort");
            this.removeQuery("sortType");
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
        this.searchText = query.search;
        this.search = true;
      } else if (this.searchText) {
        this.search = true;
      }
      this.opts.customFilter.forEach((element) => {
        if (!this.query[element.key]) {
          element.selected = null;
        } else {
          element.selected = this.query[element.key];
        }
      });
    },
    undo() {
      Object.keys(this.query).forEach((element) => {
        if (
          element.startsWith(this.opts.queryPrefix + "-") ||
          this.opts.customFilter.find((x) => x.key === element)
        ) {
          this.query[element] = undefined;
        }
      });
      Object.entries(this.undoQuery).forEach(([key, value]) => {
        this.pushQuery(key, value, false);
      });
      this.undoQuery = null;
    },
    reset() {
      this.undoQuery = { ...this.query };
      Object.keys(this.query).forEach((element) => {
        if (
          element.startsWith(this.opts.queryPrefix + "-") ||
          this.opts.customFilter.find((x) => x.key === element)
        ) {
          this.query[element] = undefined;
        }
      });
    },
    setCustomFilter(custom, value) {
      if (this.query[custom.key]) {
        if (this.query[custom.key] === value.key) {
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
      if (this.query[this.opts.queryPrefix + "-" + name] == value) {
        return;
      }
      this.$set(
        this.query,
        prefix ? this.opts.queryPrefix + "-" + name : name,
        value
      );
      if (name !== "shownRow" && name !== "page") {
        this.page = 1;
      }
    },
    removeQuery(name, prefix = true) {
      if (!this.query[prefix ? this.opts.queryPrefix + "-" + name : name]) {
        return;
      }
      this.query[
        prefix ? this.opts.queryPrefix + "-" + name : name
      ] = undefined;
      if (name !== "shownRow" && name !== "page") {
        this.page = 1;
      }
    },
  },
};
</script>
<style scoped>
html,
body,
div,
span,
applet,
object,
iframe,
h1,
h2,
h3,
h4,
h5,
h6,
p,
blockquote,
pre,
a,
abbr,
acronym,
address,
big,
cite,
code,
del,
dfn,
img,
ins,
kbd,
q,
s,
samp,
small,
strike,
sub,
sup,
tt,
var,
dl,
dt,
dd,
ol,
ul,
li,
fieldset,
form,
label,
legend,
input,
select,
textarea,
button,
table,
caption,
tbody,
tfoot,
thead,
tr,
th,
td,
article,
aside,
canvas,
details,
embed,
figure,
figcaption,
footer,
header,
hgroup,
menu,
nav,
output,
ruby,
section,
summary,
time,
mark,
audio,
video {
  margin: 0;
  padding: 0;
  border: 0;
  font-size: 100%;
  vertical-align: baseline;
}

article,
aside,
details,
figcaption,
figure,
footer,
header,
hgroup,
menu,
nav,
section {
  display: block;
}

body {
  line-height: 1;
}

ol,
ul {
  list-style: none;
}

blockquote,
q {
  quotes: none;
}

blockquote:before,
blockquote:after,
q:before,
q:after {
  content: "";
  content: none;
}

table {
  border-collapse: collapse;
  border-spacing: 0;
}

a {
  text-decoration: none;
}

img {
  border: 0;
}

:focus {
  outline: 0;
}

.clear {
  clear: both;
}

*,
::after,
::before {
  box-sizing: border-box;
}

.table-wrapper {
  width: 100%;
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  -webkit-box-direction: normal;
  -webkit-box-orient: vertical;
  -moz-box-direction: normal;
  -moz-box-orient: vertical;
  flex-direction: column;
  -webkit-flex-direction: column;
  -ms-flex-direction: column;
  align-items: flex-start;
  -webkit-box-align: start;
  -moz-box-align: start;
  -ms-flex-align: start;
  font-family: var(--vd-standart-font);
  font-size: 14px;
}

.table-responsive {
  width: 100%;
  overflow-x: auto;
}

.table-wrapper button {
  font-family: var(--vd-standart-font);
}

.table-wrapper .text-success {
  color: var(--vd-success) !important;
}

.table-wrapper .bg-none {
  background: none;
}
.table-wrapper .selector {
  padding: 0 5px;
  cursor: pointer;
}
.table-wrapper input {
  font-family: var(--vd-standart-font);
}

.table-wrapper.table-wrapper-single .table-title {
  padding-left: 0;
  padding-right: 0;
}

.table-title {
  width: 100%;
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  -webkit-box-pack: justify;
  -moz-box-pack: justify;
  -ms-flex-pack: justify;
  justify-content: space-between;
  -webkit-justify-content: space-between;
  padding: 20px 30px;
}

.table-title h6 {
  font-weight: 500;
  color: var(--vd-flat-black);
}

.table-title.table-title-single {
  -webkit-box-pack: end;
  -moz-box-pack: end;
  -ms-flex-pack: end;
  justify-content: end;
  -webkit-justify-content: end;
  border: 1px solid var(--vd-grey);
  border-bottom-width: 0;
}

.table-title.table-title-single .table-menu {
  top: -5px;
}

.table-menu {
  display: none;
}

.table-title .table-filter {
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  position: relative;
}

.table-title .table-filter .exports-box {
  position: relative;
}

.table-title .table-filter .exports-box .exports-box-submenu {
  position: absolute;
  width: 200px;
  padding: 15px;
  background: var(--vd-white);
  border-radius: 10px;
  position: absolute;
  top: 35px;
  right: 0;
  flex-direction: column;
  justify-content: left;
  align-items: end;
  box-shadow: 0px 0px 10px 0px var(--vd-shadow);
  -webkit-box-shadow: 0px 0px 10px 0px var(--vd-shadow);
  -moz-box-shadow: 0px 0px 10px 0px var(--vd-shadow);
  z-index: 1;
}

.table-title .table-filter .exports-box .exports-box-submenu button {
  margin-bottom: 10px;
}

.table-title .table-filter .exports-box .exports-box-submenu button:last-child {
  margin-bottom: 0;
}

.table-title .table-filter .exports-box .exports-box-submenu button i {
  margin-right: 5px;
}

.table-title .table-filter ul {
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  -webkit-box-align: center;
  -moz-box-align: center;
  -ms-flex-align: center;
  align-items: center;
  -webkit-box-pack: center;
  -moz-box-pack: center;
  -ms-flex-pack: center;
  justify-content: center;
  -webkit-justify-content: center;
}

.table-title .table-filter ul li {
  margin-right: 20px;
}

.table-title .table-filter ul li:last-child {
  margin-right: 0;
}

.table-title .table-filter ul li.reports-box-button {
  position: relative;
}

.table-title .table-filter ul li.close-list {
  text-align: right;
  display: none;
}

.table-title .table-filter ul li .filter-title {
  font-size: 13px;
  color: var(--vd-black);
  margin-right: 5px;
}

.table-title .table-filter ul li .filter-title.d-flex {
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
}

.table-title .table-filter ul li .button-item {
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  -webkit-box-align: center;
  -moz-box-align: center;
  -ms-flex-align: center;
  align-items: center;
  transition: 1s all;
  -webkit-transition: 1s all;
  -moz-transition: 1s all;
  -ms-transition: 1s all;
  -o-transition: 1s all;
  background: none;
  color: var(--vd-cascade-grey);
  font-family: var(--vd-standart-font);
  font-weight: 400;
  height: 20px;
  font-size: 14px;
  cursor: pointer;
}

.table-title .table-filter ul li .button-item i {
  margin-left: 5px;
}

.table-title .table-filter ul li .search {
  position: relative;
  height: 20px;
}

.table-title .table-filter ul li .search input {
  height: 100%;
  background: var(--vd-grey);
  font-size: 12px;
  font-family: var(--vd-standart-font);
  font-weight: 500;
  color: var(--vd-black);
  padding: 0 10px;
}

.table-title .table-filter ul li .search input::-webkit-input-placeholder {
  color: var(--vd-dark-blue);
}

.table-title .table-filter ul li .search input::-moz-placeholder {
  color: var(--vd-dark-blue);
}

.table-title .table-filter ul li .search input:-ms-input-placeholder {
  color: var(--vd-dark-blue);
}

.table-title .table-filter ul li .search input:-moz-placeholder {
  color: var(--vd-dark-blue);
}

.table-title .table-filter ul li .search input::placeholder {
  color: var(--vd-dark-blue);
}

.table-title .table-filter ul li .search button {
  position: absolute;
  right: 10px;
  top: 2px;
  height: 100%;
  color: var(--vd-danger);
  cursor: pointer;
}

.table-title .table-filter ul li .filters-box {
  width: 500px;
  padding: 30px;
  background: var(--vd-white);
  border-radius: 10px;
  position: absolute;
  top: 35px;
  right: -30px;
  -webkit-box-direction: normal;
  -webkit-box-orient: vertical;
  -moz-box-direction: normal;
  -moz-box-orient: vertical;
  -webkit-flex-direction: column;
  -ms-flex-direction: column;
  flex-direction: column;
  -webkit-box-pack: left;
  -moz-box-pack: left;
  -ms-flex-pack: left;
  -webkit-justify-content: left;
  justify-content: left;
  -webkit-box-align: end;
  -moz-box-align: end;
  -ms-flex-align: end;
  -webkit-align-items: end;
  align-items: end;
  box-shadow: 0px 0px 10px 0px var(--vd-shadow);
  -webkit-box-shadow: 0px 0px 10px 0px var(--vd-shadow);
  -moz-box-shadow: 0px 0px 10px 0px var(--vd-shadow);
  z-index: 1;
  background: var(--vd-white);
}

.table-title .table-filter ul li .filters-box .filter-close {
  display: none;
}

.table-title .table-filter ul li .filters-box.reports-box {
  width: 300px;
  right: 0;
}

.table-title .table-filter ul li .filters-box li {
  width: 100%;
  border-bottom: 1px solid var(--vd-light-info);
  padding-bottom: 10px;
  margin-bottom: 10px;
  text-align: left;
}

.table-title .table-filter ul li .filters-box li:last-child {
  border-bottom: none;
}

.table-title .table-filter ul li .filters-box li.date.date-range {
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  position: relative;
}

.table-title .table-filter ul li .filters-box li.date.date-range input {
  flex: auto;
}

.table-title
  .table-filter
  ul
  li
  .filters-box
  li.date.date-range
  .date-range-clear {
  position: absolute;
  right: 0;
  top: calc(50% - 10px);
  color: var(--vd-danger);
  font-size: 17px;
}

.table-title .table-filter ul li .filters-box li.date .date-columns-group {
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  -webkit-flex-direction: row;
  -ms-flex-direction: row;
  flex-direction: row;
  -webkit-box-pack: left;
  -moz-box-pack: left;
  -ms-flex-pack: left;
  -webkit-justify-content: left;
  justify-content: left;
  overflow-x: scroll;
  margin-top: 10px;
  padding-bottom: 10px;
}

.table-title .table-filter ul li .filters-box li.date .date-columns-group .btn {
  padding: 5px 10px;
  background: none;
  border-radius: 30px;
  font-size: 13px;
  border: 2px solid transparent;
  color: var(--vd-cascade-grey);
  cursor: pointer;
}
.table-title .table-filter ul li .filters-box li.date .date-columns-group .btn {
  padding: 5px 10px;
  background: none;
  border-radius: 30px;
  font-size: 13px;
  border: 2px solid transparent;
  color: var(--vd-cascade-grey);
  cursor: pointer;
}
.table-title
  .table-filter
  ul
  li
  .filters-box
  li.date
  .date-columns-group
  .btn.btn-success {
  border-color: var(--vd-success);
}

.table-title
  .table-filter
  ul
  li
  .filters-box
  li.date
  .date-columns-group
  .btn.btn-default {
  border-color: var(--vd-grey-v-2);
}

.table-title
  .table-filter
  ul
  li
  .filters-box
  li.date
  .date-columns-group::-webkit-scrollbar-track {
  -webkit-box-shadow: none;
  box-shadow: none;
  background-color: var(--vd-grey-v-2);
  border-radius: 10px;
  opacity: 0;
}

.table-title
  .table-filter
  ul
  li
  .filters-box
  li.date
  .date-columns-group::-webkit-scrollbar {
  width: 3px;
  height: 4px;
  background-color: var(--vd-grey-v-2);
  opacity: 0;
}

.table-title
  .table-filter
  ul
  li
  .filters-box
  li.date
  .date-columns-group::-webkit-scrollbar-thumb {
  border-radius: 10px;
  height: 4px;
  background-color: var(--vd-cascade-grey);
}

.table-title .table-filter ul li .filters-box li.date .date-columns-group li {
  -webkit-box: none;
  -moz-box: none;
  -webkit-flex: none;
  -ms-flex: none;
  flex: none;
  width: inherit;
  border: none;
  margin-bottom: 0;
  padding-bottom: 0;
  margin-right: 0;
}

.table-title
  .table-filter
  ul
  li
  .filters-box
  li.date
  .date-columns-group
  li
  button {
  margin-right: 10px;
}

.table-title .table-filter ul li .filters-box li.date .date-picker-filter {
  z-index: 1;
  display: flex;
}

.table-title .table-filter ul li .filters-box li.date button.default-select {
  font-family: var(--vd-standart-font);
  color: var(--vd-dark-blue);
  font-size: 13px;
  margin-right: 10px;
  font-weight: 500;
  cursor: pointer;
}

.table-title
  .table-filter
  ul
  li
  .filters-box
  li.date
  button.default-select:last-child {
  margin-right: 0;
}

.table-body {
  width: 100%;
  position: relative;
}

.table-body .table-blank {
  min-height: 60vh;
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  -webkit-box-align: center;
  -moz-box-align: center;
  -ms-flex-align: center;
  -webkit-align-items: center;
  align-items: center;
  -webkit-box-pack: center;
  -moz-box-pack: center;
  -ms-flex-pack: center;
  -webkit-justify-content: center;
  justify-content: center;
  text-align: center;
  font-family: var(--vd-standart-font);
  font-weight: 400;
  color: var(--black);
  font-size: 16px;
}

.table-body table {
  width: 100%;
  text-align: left;
}

.table-body table th,
td {
  padding: 20px 30px;
  font-family: var(--vd-standart-font);
  font-weight: 500;
  border: 1px solid var(--vd-grey);
  color: var(--vd-flat-black);
  font-size: 13px;
}

.table-body table thead {
  background-color: var(--vd-grey-v-4);
}

.table-body table thead tr th {
  color: var(--vd-dark-blue);
  font-size: 14px;
}

.table-body table thead tr th .sort-icon {
  width: 30px;
  display: inline-table;
  vertical-align: middle;
}

.table-body table thead tr th .sort-icon i {
  line-height: 0.35;
  display: block;
}

.table-body table body {
  background-color: var(--vd-white);
}

.table-body table body tr {
  transition: 1s all;
  -webkit-transition: 1s all;
  -moz-transition: 1s all;
  -ms-transition: 1s all;
  -o-transition: 1s all;
}

.table-body table body tr:hover {
  background-color: var(--vd-grey-v-4);
}

.table-body table body tr td {
  border: 1px solid var(--vd-grey);
  color: var(--vd-flat-black);
  font-size: 13px;
}

.table-body table body tr td.table-button-group {
  vertical-align: middle;
  text-align: right;
}

.table-bottom {
  width: 100%;
  padding: 20px 30px;
  background-color: var(--vd-grey-v-4);
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  -webkit-box-pack: justify;
  -moz-box-pack: justify;
  -ms-flex-pack: justify;
  -webkit-justify-content: space-between;
  justify-content: space-between;
}

.table-bottom .left {
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  -webkit-box-align: center;
  -moz-box-align: center;
  -ms-flex-align: center;
  -webkit-align-items: center;
  align-items: center;
}

.table-bottom .left .rows-number {
  color: var(--vd-cascade-grey);
  font-size: 13px;
}

.table-bottom .left select {
  background: none;
  font-family: var(--vd-standart-font);
  margin-left: 5px;
  font-weight: 500;
  font-size: 13px;
  border: 1px solid var(--vd-grey-v-2);
  color: var(--vd-flat-black);
  border-radius: 5px;
  margin-right: 5px;
}

.table-bottom .left .of-rows {
  color: var(--vd-cascade-grey);
  font-size: 13px;
}

.table-bottom .center {
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
}

.table-bottom .center ul {
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  -webkit-flex-wrap: wrap;
  -ms-flex-wrap: wrap;
  flex-wrap: wrap;
  -webkit-box-pack: center;
  -moz-box-pack: center;
  -ms-flex-pack: center;
  -webkit-justify-content: center;
  justify-content: center;
}

.table-bottom .center ul li {
  margin-right: 10px;
  cursor: pointer;
  font-weight: 500;
  font-family: var(--vd-standart-font);
  color: var(--vd-cascade-grey);
  font-size: 14px;
}

.table-bottom .center ul li.active {
  color: var(--vd-dark-blue);
}

.table-bottom .right {
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  -webkit-box-pack: center;
  -moz-box-pack: center;
  -ms-flex-pack: center;
  -webkit-justify-content: center;
  justify-content: center;
  -webkit-box-align: center;
  -moz-box-align: center;
  -ms-flex-align: center;
  -webkit-align-items: center;
  align-items: center;
}

.table-bottom .right span {
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  -webkit-box-pack: center;
  -moz-box-pack: center;
  -ms-flex-pack: center;
  -webkit-justify-content: center;
  justify-content: center;
  -webkit-box-align: center;
  -moz-box-align: center;
  -ms-flex-align: center;
  -webkit-align-items: center;
  align-items: center;
  cursor: pointer;
  font-size: 12px;
  font-family: var(--standart-font);
  font-weight: 500;
  margin-right: 10px;
  color: var(--vd-dark-blue);
}

.table-bottom .right span:hover {
  opacity: 0.7;
}

.table-bottom .right span:last-child {
  margin-right: 0;
}

@media screen and (max-width: 1600px) {
  .table-body table th,
  td {
    padding: 13px 8px;
  }
  .table-body table thead tr th {
    font-size: 13px;
  }
  .table-body table body tr td {
    font-size: 12px;
  }
}

@media screen and (max-width: 1030px) {
  .table-title {
    padding: 20px;
    position: relative;
    z-index: 2;
  }
  .table-menu {
    display: block;
    position: absolute;
    right: 0;
    top: 5px;
  }
  .table-title .table-filter {
    border-radius: 5px;
    justify-content: center;
    width: 100%;
    position: absolute;
    right: 20px;
    top: 20px;
  }
  .table-title .table-filter ul.list-propery {
    position: absolute;
    right: 0;
    top: 30px;
    display: block;
    background-color: var(--vd-white);
    box-shadow: 0px 0px 14px 0px var(--vd-shadow);
    -webkit-box-shadow: 0px 0px 14px 0px var(--vd-shadow);
    -moz-box-shadow: 0px 0px 14px 0px var(--vd-shadow);
    display: block;
  }
  .table-title .table-filter ul li {
    margin-right: 0;
    padding: 10px 20px;
    border-bottom: 1px solid var(--vd-grey);
  }
  .table-title .table-filter ul li.search-box {
    position: relative;
  }
  .table-title .table-filter ul li.close-list {
    display: block;
  }
  .table-title .table-filter ul li .button-item {
    font-size: 12px;
  }
  .table-title .table-filter ul li .search {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
  }
  .table-title .table-filter ul li .search input {
    width: 190px;
    height: 45px;
    position: absolute;
    color: var(--vd-dark-blue);
    right: 0;
    top: 0;
  }
  .table-title .table-filter ul li .search button {
    top: 14px;
  }
  .table-title .table-filter ul li .filters-box {
    width: 300px;
    right: 0;
    top: 0;
    padding: 10px;
  }
  .table-title .table-filter ul li .filters-box li {
    padding: 0;
  }
  .table-title .table-filter ul li .filters-box li.date.date-range {
    -webkit-box-pack: center;
    -moz-box-pack: center;
    -ms-flex-pack: center;
    -webkit-justify-content: center;
    justify-content: center;
    -webkit-box-align: center;
    -moz-box-align: center;
    -ms-flex-align: center;
    -webkit-align-items: center;
    align-items: center;
  }
  .table-title
    .table-filter
    ul
    li
    .filters-box
    li.date
    .date-columns-group
    li
    button {
    font-size: 10px;
  }
  .table-title .table-filter ul li .filters-box li.date button.default-select {
    margin-right: 5px;
  }
  .table-body table th,
  td {
    padding: 10px 8px;
    font-size: 12px;
  }
  .table-body table thead tr th {
    font-size: 13px;
  }
  .table-body table body tr td {
    font-size: 10px;
  }
  .table-title .table-filter ul li .filters-box .filter-close {
    display: block;
    text-align: right;
    color: var(--vd-danger);
    font-size: 20px;
  }
  .table-title .table-filter ul li .filter-title.d-flex {
    -webkit-box-direction: normal;
    -webkit-box-orient: vertical;
    -moz-box-direction: normal;
    -moz-box-orient: vertical;
    -webkit-flex-direction: column;
    -ms-flex-direction: column;
    flex-direction: column;
    width: 100%;
  }
  .table-title .table-filter ul li .filters-box li.date.date-range input {
    flex: 1;
    width: 50%;
  }
  .table-title .table-filter ul li .filters-box li.date .date-picker-filter {
    padding: 0;
    margin-top: 5px;
  }
  .table-title
    .table-filter
    ul
    li
    .filters-box
    li.date
    .date-columns-group
    .btn {
    font-size: 10px;
    padding: 5px 10px;
  }
}

@media screen and (max-width: 640px) {
  .table-body table th,
  td {
    min-width: 200px;
  }
  .table-body table th {
    font-size: 12px;
  }
  .table-body table td {
    font-size: 11px;
  }
  .table-bottom {
    -webkit-box-direction: normal;
    -webkit-box-orient: vertical;
    -moz-box-direction: normal;
    -moz-box-orient: vertical;
    -webkit-flex-direction: column;
    -ms-flex-direction: column;
    flex-direction: column;
  }
  .table-bottom .left {
    -webkit-box-pack: center;
    -moz-box-pack: center;
    -ms-flex-pack: center;
    -webkit-justify-content: center;
    justify-content: center;
    margin-bottom: 10px;
  }
  .table-bottom .center {
    -webkit-box-pack: center;
    -moz-box-pack: center;
    -ms-flex-pack: center;
    -webkit-justify-content: center;
    justify-content: center;
    text-align: center;
    margin: 0 0 5px 0;
  }
  .table-bottom .center ul li {
    padding: 5px 10px;
    background-color: var(--vd-white);
    border-radius: 3px;
    margin-bottom: 5px;
    min-width: 35px;
  }
}

@media screen and (max-width: 350px) {
  .table-bottom .right span {
    font-size: 10px;
  }
}

.date-picker-filter {
  top: 0px !important;
  padding: 0 5px;
}

.date-picker-filter input {
  color: #95a5a6;
}
</style>