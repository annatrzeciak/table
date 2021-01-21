<template>
  <div id="app">
    <b-container>
      <h1>Users</h1>
      <Table
        endpoint="https://jsonplaceholder.typicode.com/users"
        search
        sorting
        pagination
        :properties="'name, email, company.name, address.city, website'"
        @changed="tableChanged"
      />
    </b-container>
  </div>
</template>

<script>
import Table from "@/components/Table.vue";

export default {
  name: "App",
  components: {
    Table
  },
  methods: {
    tableChanged(tableItems) {
      console.log(tableItems);
    }
  }
};
</script>

<style lang="scss">
  html{
    max-width: 100vw;
    overflow-x: hidden;
  }
#app {
  margin-top: 60px;
  overflow: hidden;
}
</style>
