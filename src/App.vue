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
  }
};
</script>

<style lang="scss">
#app {
  margin-top: 60px;
}
</style>
