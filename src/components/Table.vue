<template>
  <div>
    <b-alert :show="errorMessage.length" variant="danger">{{errorMessage}}</b-alert>

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
    data: null,
    errorMessage: ''
  }),
  async mounted() {
    try {
      this.data = (await this.axios.get(this.endpoint)).data;
    } catch (e) {
      this.errorMessage = e.message
    }
  }
};
</script>

<style scoped></style>
