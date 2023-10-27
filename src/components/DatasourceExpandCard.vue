<template>
  <div class="d-flex justify-center align-center w-100">
    <v-card style="width: 75%; max-width: 828px" elevation="2">
      <v-card-title
        :class="[
          this.$props.eventObj.event_type.includes('Deleted')
            ? 'red-background'
            : 'blue-background',
        ]"
      >
        <div class="d-flex justify-space-between align-center">
          <span> {{ formatEventName($props.eventObj.event_type) }} </span>
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
          <template v-if="$props.eventObj.event_type == 'DatasourceCreated'">
            <a
              :href="`https://10ax.online.tableau.com/#/site/kjmdev797388/datasources?order=relevancy%3Adesc&search=${$props.eventObj.resource_name}`"
              target="_blank"
            >
              {{ $props.eventObj.resource_name }}
              <v-icon icon="mdi-open-in-new" color="blue" size="18"></v-icon>
            </a>
          </template>
          <template v-else>
            {{ $props.eventObj.resource_name }}
            <div class="text-subtitle-2">
              LUID: {{ $props.eventObj.resource_luid }}
            </div>
          </template>
        </div>
        <template v-if="$props.eventObj.event_type == 'DatasourceCreated'">
          <div class="py-1">
            Created By:
            <span
              ><a
                :href="`mailto:${$props.eventObj.owner_email}`"
                class="card-link"
                >{{ $props.eventObj.owner_name }}</a
              >
              ({{ $props.eventObj.owner_site_role }})</span
            >
          </div>
          <div class="py-1">
            Project Name:
            <span
              ><strong>{{ $props.eventObj.project_name }}</strong></span
            >
          </div>
        </template>
      </v-card-text>
      <v-card-actions>
        <div class="d-flex w-100 justify-end">
          <div
            class="justify-self-start w-100"
            v-if="$props.eventObj.event_type == 'DatasourceCreated'"
          >
            <v-btn
              variant="outlined"
              density="comfortable"
              color="red-lighten-2"
              @click="showDeleteDialog = true"
              ><v-icon
                icon="mdi-delete-outline"
                color="red-lighten-2"
                class="mr-2"
              ></v-icon
              >Delete</v-btn
            >
          </div>
          <div class="px-2">
            <a
              href="https://10ax.online.tableau.com/#/site/kjmdev797388"
              target="_blank"
            >
              <v-btn
                variant="tonal"
                color="blue-grey-darken-1"
                density="comfortable"
                >View Site
                <v-icon icon="mdi-open-in-new" class="ml-2"></v-icon></v-btn
            ></a>
          </div></div
      ></v-card-actions>
    </v-card>

    <v-dialog
      persistent
      v-model="showDeleteDialog"
      style="width: 50%; max-width: 550px"
    >
      <v-card>
        <v-card-title class="delete-title">
          <div class="d-flex w-100 justify-space-between">
            <span>Confirm: Delete Datasource</span>
            <span
              ><v-icon
                icon="mdi-close close-icon"
                @click="showDeleteDialog = false"
              ></v-icon
            ></span>
          </div>
        </v-card-title>
        <v-card-subtitle class="py-6">
          <div class="d-flex justify-center">
            <span class="text-h6">
              <v-icon
                icon="mdi-alert"
                class="mr-2"
                color="orange-darken-3"
              ></v-icon
              >This cannot be undone!</span
            >
          </div></v-card-subtitle
        >
        <v-card-actions>
          <div class="d-flex justify-center w-100">
            <v-btn
              color="red-lighten-2"
              variant="flat"
              @click="
                isLoading = true;
                deleteDatasource($props.eventObj.resource_luid);
              "
              :loading="isLoading"
              >Delete {{ $props.eventObj.resource_name }}</v-btn
            >
          </div>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data: () => ({
    isLoading: false,
    showDeleteDialog: false,
  }),
  props: ["eventObj"],
  computed: {
    getSubtitleText() {
      switch (this.$props.eventObj.event_type) {
        case "DatasourceCreated":
          return "A new datasource has been created:";

        case "DatasourceDeleted":
          return "The following datasource has been deleted:";

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
    deleteDatasource(dsLuid) {
      let reqBody = { ds_luid: dsLuid };

      axios
        .post(
          "http://192.168.1.118:8000/webhookActions/delete_datasource/",
          reqBody
        )
        .then(() => {
          this.showDeleteDialog = false;
        })
        .catch((e) => {
          console.log("ERROR: ", e);
          this.showDeleteDialog = false;
        });
    },
  },
};
</script>

<style>
.red-background {
  background-color: #e57373 !important;
}

.blue-background {
  background-color: #81d4fa !important;
}

.card-link {
  color: blue;
  text-decoration: none;
}

.card-link:hover {
  text-decoration: underline;
}

.delete-title {
  background-color: #e57373 !important;
}

.close-icon {
  cursor: pointer;
}
</style>
