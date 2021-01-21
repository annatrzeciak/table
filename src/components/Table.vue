<template>
  <div class="table-content">
    <b-alert :show="errorMessage.length" variant="danger">
      {{ errorMessage }}
    </b-alert>

    <b-table
      v-if="items"
      striped
      hover
      :items="items"
      :fields="fields"
      :per-page="pagination ? perPage : 0"
      :current-page="currentPage"
      small
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
      :total-rows="rows"
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
    data: [],
    errorMessage: "",
    perPage: 3,
    currentPage: 1
  }),
  computed: {
    fields() {
      const fields = [];
      if (this.propertiesToShow) {
        this.propertiesToShow.forEach(prop => {
          if (prop === this.emailPropertyName) {
            fields.push({
              key: "email",
              label: prop
                .replaceAll(".", " ")
                .replaceAll("_", " ")
                .toLowerCase()
                .split(" ")
                .map(word => word.charAt(0).toUpperCase() + word.slice(1))
                .join(" "),

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
    rows() {
      return this.items.length;
    }
  },
  async mounted() {
    try {
      this.data = (await this.axios.get(this.endpoint)).data;
    } catch (e) {
      this.errorMessage = e.message;
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
