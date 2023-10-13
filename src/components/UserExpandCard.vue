<template>
  <div class="d-flex justify-center align-center w-100">
    <v-card style="width: 75%; max-width: 828px" elevation="2">
      <v-card-title
        :class="[
          this.$props.eventObj.event_type.includes('Deleted') ||
          this.$props.eventObj.event_type.includes('Demoted')
            ? 'red-background'
            : 'blue-background',
        ]"
      >
        <div class="d-flex justify-space-between align-center">
          <span>
            {{ formatEventName($props.eventObj.event_type) }}
          </span>
          <span class="text-caption">{{
            formattedDate(this.$props.eventObj.created_at)
          }}</span>
        </div></v-card-title
      >
      <v-card-subtitle class="pt-2">
        <div class="d-flex justify-space-between">
          <div>
            <span
              ><v-icon
                icon="mdi-alert"
                class="mr-2"
                color="orange-darken-3"
              ></v-icon
            ></span>
            <span class="text-subtitle-2">{{ getSubtitleText }}</span>
          </div>
        </div>
      </v-card-subtitle>
      <v-card-text>
        <div class="text-h6 wb-link pb-2">
          {{ $props.eventObj.resource_name }}
        </div>
      </v-card-text>
      <v-card-actions>
        <div class="d-flex w-100 justify-end">
          <div class="px-2">
            <a
              href="https://10ax.online.tableau.com/#/site/kjmdev797388"
              target="_blank"
            >
              <v-btn
                variant="flat"
                color="blue-grey-darken-1"
                density="comfortable"
                >View Site
                <v-icon icon="mdi-open-in-new" class="ml-2"></v-icon></v-btn
            ></a>
          </div>
          <div class="px-2">
            <a
              href="https://10ax.online.tableau.com/#/site/kjmdev797388/users"
              target="_blank"
            >
              <v-btn
                variant="flat"
                color="blue-grey-darken-1"
                density="comfortable"
                >View Users
                <v-icon icon="mdi-open-in-new" class="ml-2"></v-icon></v-btn
            ></a>
          </div></div
      ></v-card-actions>
    </v-card>
  </div>
</template>

<script>
export default {
  data: () => ({}),
  props: ["eventObj"],
  computed: {
    getSubtitleText() {
      switch (this.$props.eventObj.event_type) {
        case "UserDeleted":
          return "The following user has been deleted:";

        case "AdminPromoted":
          return "The following user has been promoted to an Admin:";

        case "AdminDemoted":
          return "The following Admin user has been demoted:";

        default:
          return "I don't know!!";
      }
    },
  },
  methods: {
    formatEventName(eventName) {
      let newStr = "";

      newStr = eventName.replace(/([a-z])([A-Z])/g, "$1 $2");
      newStr = newStr.replace(/([A-Z])([A-Z][a-z])/g, "$1 $2");

      return newStr;
    },

    formattedDate(eventDate) {
      let formattedDate = new Date(eventDate);

      let dateStr = formattedDate.toLocaleDateString("en-us", {
        weekday: "long",
        year: "numeric",
        month: "short",
        day: "numeric",
      });

      let timeStr = formattedDate.toLocaleTimeString("en-us", {
        hour: "2-digit",
        minute: "2-digit",
      });

      return `${dateStr} - ${timeStr}`;
    },
  },
};
</script>

<style>
.red-background {
  background-color: orangered;
}

.blue-background {
  background-color: #03a9f4;
}
</style>
