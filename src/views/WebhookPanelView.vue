<template>
  <v-container>
    <WebhookNav />
    <EventTable />
  </v-container>
</template>

<script>
import WebhookNav from "@/components/WebhookNav.vue";
import EventTable from "@/components/EventTable.vue";

export default {
  components: {
    WebhookNav,
    EventTable,
  },
};
</script>

<style></style>
