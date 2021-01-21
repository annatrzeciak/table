<template>
  <div class="table-content">
    <b-alert :show="errorMessage.length" variant="danger">
      {{ errorMessage }}
    </b-alert>
    <div class=" form-inline" v-if="search">
      <b-form-input
        type="search"
        v-model="searchValue"
        placeholder="Search..."
        class="mb-3"
      />
      <span class="mx-3 mb-3">
        in
      </span>

      <b-form-select
        class="mb-3"
        v-model="selected"
        :options="searchOptions"
      ></b-form-select>
    </div>
    <b-table
      v-model="visibleItems"
      @filtered="onFiltered"
      v-if="items"
      striped
      hover
      small
      show-empty
      :items="items"
      :fields="fields"
      :per-page="pagination ? perPage : 0"
      :current-page="currentPage"
      :filter="selected"
      :filter-function="filterTable"
      responsive="lg"
    >
      <template #cell(email)="data">
        <a :href="`mailto:${data.value.toLowerCase()}`">{{ data.value }}</a>
      </template>
    </b-table>
    <b-pagination
      class="mt-auto"
      v-if="items && pagination"
      align="center"
      v-model="currentPage"
      :total-rows="totalItems"
      :per-page="perPage"
      aria-controls="my-table"
    ></b-pagination>
  </div>
</template>

<script>
export default {
  name: "Table",
  props: {
    endpoint: { type: String, required: true },
    search: { type: Boolean, default: false },
    sorting: { type: Boolean, default: false },
    pagination: { type: Boolean, default: false },
    properties: { type: String, default: "" }
  },
  data: () => ({
    visibleItems: [],
    data: [],
    errorMessage: "",
    totalItems: 0,
    perPage: 3,
    currentPage: 1,
    searchValue: "",
    selected: "all"
  }),
  computed: {
    fields() {
      const fields = [];
      if (this.propertiesToShow) {
        this.propertiesToShow.forEach(prop => {
          if (prop === this.emailPropertyName) {
            fields.push({
              key: "email",
              label: this.formatToLabel(prop),
              sortable: this.sorting
            });
          } else {
            fields.push({
              key: prop.replaceAll(".", "_"),
              sortable: this.sorting
            });
          }
        });
      }
      return fields;
    },
    emailPropertyName() {
      return this.propertiesToShow.find(
        prop =>
          prop.toLowerCase().includes("email") ||
          prop.toLowerCase().includes("e-mail") ||
          prop.toLowerCase().includes("@")
      );
    },
    propertiesToShow() {
      return this.properties.split(",").map(prop => prop.trim());
    },
    items() {
      const items = [...this.data];
      if (this.data) {
        return items.map(item => {
          const row = {};
          this.propertiesToShow.forEach(prop => {
            if (prop.includes(".")) {
              row[prop.replaceAll(".", "_")] = eval("item." + prop);
            } else {
              row[prop] = item[prop];
            }
          });
          return row;
        });
      }
      return items;
    },
    searchOptions() {
      const options = [{ value: "all", text: "All columns" }];

      this.propertiesToShow.forEach(prop => {
        options.push({ value: prop, text: this.formatToLabel(prop) });
      });
      return options;
    }
  },
  watch: {
    items() {
      this.totalItems = this.items.length;
    },
    visibleItems() {
      this.$emit("changed", this.visibleItems);
    }
  },
  async mounted() {
    try {
      this.data = (await this.axios.get(this.endpoint)).data;
    } catch (e) {
      this.errorMessage = e.message;
    }
  },
  methods: {
    formatToLabel(string) {
      return string
        .replaceAll(".", " ")
        .replaceAll("_", " ")
        .toLowerCase()
        .split(" ")
        .map(word => word.charAt(0).toUpperCase() + word.slice(1))
        .join(" ");
    },

    filterTable(row, filter) {
      if (filter === "all") {
        return Object.values(row).find(val =>
          val.toUpperCase().includes(this.searchValue.toUpperCase())
        );
      } else {
        return row[filter]
          .toUpperCase()
          .includes(this.searchValue.toUpperCase());
      }
    },
    onFiltered(filteredItems) {
      this.currentPage = 1;
      this.totalItems = filteredItems.length;
    }
  }
};
</script>

<style lang="scss" scoped>
.table-content {
  display: flex;
  justify-content: space-between;
  flex-direction: column;
  min-height: 220px;
}
</style>
