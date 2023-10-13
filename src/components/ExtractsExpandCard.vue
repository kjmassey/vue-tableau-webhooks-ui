<template>
  <div class="d-flex justify-center align-center w-100">
    <v-card style="width: 75%; max-width: 828px" elevation="2">
      <v-card-title :class="computedClass">
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
          <!-- <span>{{ formattedDate($props.eventObj.created_at) }}</span> -->
        </div>
      </v-card-subtitle>
      <v-card-text>
        <div class="text-h6 wb-link pb-2">
          <a :href="computedItemUrl" target="_blank">
            {{ $props.eventObj.resource_name }}
            <v-icon icon="mdi-open-in-new" color="blue" size="18"></v-icon>
          </a>
        </div>

        <div class="py-1">
          Owner:
          <span
            ><a
              :href="`mailto:${$props.eventObj.owner_email}`"
              class="card-link"
              >{{ $props.eventObj.owner_name }}</a
            >
          </span>
        </div>
        <div class="py-1">
          Project Name:
          <span
            ><strong>{{ $props.eventObj.project_name }}</strong></span
          >
        </div>
      </v-card-text>
      <v-card-actions>
        <div class="d-flex w-100 justify-end">
          <div
            class="justify-self-start w-100"
            v-if="$props.eventObj.event_type.includes('RefreshFailed')"
          >
            <v-btn
              variant="flat"
              density="comfortable"
              color="green"
              class="px-2"
              @click="showRefreshDialog = true"
              ><v-icon icon="mdi-refresh" color="white" class="mx-1"></v-icon
              >Retry</v-btn
            >
            <v-btn
              variant="flat"
              density="comfortable"
              color="yellow-darken-3"
              class="px-2"
              @click="showSchedsDialog = true"
              ><v-icon icon="mdi-calendar" color="black" class="mx-1"></v-icon
              >Remove Schedules</v-btn
            >
            <v-btn
              variant="flat"
              density="comfortable"
              color="blue"
              class="px-2"
              @click="showTagDialog = true"
              ><v-icon icon="mdi-label" color="white" class="mx-1"></v-icon>Add
              Tag</v-btn
            >
            <v-btn
              variant="flat"
              density="comfortable"
              color="red-darken-2"
              class="px-2"
              @click="showDeleteDialog = true"
              ><v-icon
                icon="mdi-delete-outline"
                color="white"
                class="mx-1"
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
                variant="flat"
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
        <v-card-title class="delete-title text-white">
          <div class="d-flex w-100 justify-space-between">
            <span>Confirm: Delete Item</span>
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
              color="red-darken-2"
              variant="flat"
              @click="
                isLoading = true;
                deleteItem($props.eventObj.resource_luid);
              "
              :loading="isLoading"
              >Delete {{ $props.eventObj.resource_name }}</v-btn
            >
          </div>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog
      persistent
      v-model="showRefreshDialog"
      style="width: 50%; max-width: 550px"
    >
      <v-card>
        <v-card-title class="refresh-title text-white">
          <div class="d-flex w-100 justify-space-between">
            <span>Confirm: Retry Extract Refresh</span>
            <span
              ><v-icon
                icon="mdi-close close-icon"
                @click="showRefreshDialog = false"
              ></v-icon
            ></span>
          </div>
        </v-card-title>
        <v-card-subtitle class="py-6">
          <div class="d-flex justify-center">
            <span class="text-body-2">
              <v-icon
                icon="mdi-information"
                class="mr-2"
                color="orange-darken-3"
              ></v-icon
              >This will queue another refresh. <br />
              <br />
              Watch these logs to confirm success/failure.</span
            >
          </div></v-card-subtitle
        >
        <v-card-actions>
          <div class="d-flex justify-center w-100">
            <v-btn
              color="green"
              variant="flat"
              @click="
                isLoading = true;
                retryExtractRefresh($props.eventObj.resource_luid);
              "
              :loading="isLoading"
              >Refresh {{ $props.eventObj.resource_name }}</v-btn
            >
          </div>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog
      persistent
      v-model="showSchedsDialog"
      style="width: 50%; max-width: 550px"
    >
      <v-card>
        <v-card-title class="sched-title text-black">
          <div class="d-flex w-100 justify-space-between">
            <span>Confirm: Remove Extract Schedules</span>
            <span
              ><v-icon
                icon="mdi-close close-icon"
                @click="showSchedsDialog = false"
              ></v-icon
            ></span>
          </div>
        </v-card-title>
        <v-card-subtitle class="py-6">
          <div class="d-flex justify-center">
            <span class="text-body-2">
              <v-icon
                icon="mdi-information"
                class="mr-2"
                color="yellow-darken-3"
              ></v-icon
              >This will remove all existing refresh schedules.</span
            >
          </div></v-card-subtitle
        >
        <v-card-actions>
          <div class="d-flex justify-center w-100">
            <v-btn
              color="yellow-darken-3"
              variant="flat"
              @click="
                isLoading = true;
                removeSchedules($props.eventObj.resource_luid);
              "
              :loading="isLoading"
              >Unschedule {{ $props.eventObj.resource_name }}</v-btn
            >
          </div>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog
      persistent
      v-model="showTagDialog"
      style="width: 50%; max-width: 550px"
    >
      <v-card>
        <v-card-title class="tag-title text-white">
          <div class="d-flex w-100 justify-space-between">
            <span>Confirm: Remove Extract Schedules</span>
            <span
              ><v-icon
                icon="mdi-close close-icon"
                @click="showSchedsDialog = false"
              ></v-icon
            ></span>
          </div>
        </v-card-title>
        <v-card-subtitle class="py-6">
          <div class="d-flex justify-center">
            <span class="text-body-2">
              <v-icon
                icon="mdi-information"
                class="mr-2"
                color="yellow-darken-3"
              ></v-icon
              >This will add a 'Fails Often' tag to this item.</span
            >
          </div></v-card-subtitle
        >
        <v-card-actions>
          <div class="d-flex justify-center w-100">
            <v-btn
              color="blue"
              variant="flat"
              @click="
                isLoading = true;
                addTag($props.eventObj.resource_luid);
              "
              :loading="isLoading"
              >Tag {{ $props.eventObj.resource_name }}</v-btn
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
    showRefreshDialog: false,
    showSchedsDialog: false,
    showTagDialog: false,
  }),
  props: ["eventObj"],
  computed: {
    getSubtitleText() {
      switch (this.$props.eventObj.event_type) {
        case "DatasourceRefreshFailed":
          return "A scheduled or ad-hoc refresh for the datasource below has failed:";

        case "DatasourceRefreshSucceeded":
          return "A scheduled or ad-hoc refresh for the datasource below has succeeded:";

        case "DatasourceRefreshStarted":
          return "A scheduled or ad-hoc refresh for the datasource below has started:";

        case "WorkbookRefreshFailed":
          return "A scheduled or ad-hoc refresh for the workbook below has failed:";

        case "WorkbookRefreshSucceeded":
          return "A scheduled or ad-hoc refresh for the workbook below has succeeded:";

        case "WorkbookRefreshStarted":
          return "A scheduled or ad-hoc refresh for the workbook below has started:";

        default:
          return "I don't know!!";
      }
    },
    computedItemUrl() {
      let resourceName = this.$props.eventObj.resource_name;

      if (this.$props.eventObj.event_type.includes("Datasource")) {
        return `https://10ax.online.tableau.com/#/site/kjmdev797388/datasources?order=relevancy%3Adesc&search=${resourceName}`;
      } else {
        return this.$props.eventObj.webpage_url;
      }
    },
    computedClass() {
      if (this.$props.eventObj.event_type.includes("RefreshSucceeded")) {
        return "blue-background";
      } else if (this.$props.eventObj.event_type.includes("RefreshFailed")) {
        return "red-background";
      } else {
        return "gray-background";
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
    retryExtractRefresh(luid) {
      let reqBody = { luid: luid, event_type: this.$props.eventObj.event_type };

      axios
        .post("http://192.168.1.118:8000/webhookActions/refresh_item/", reqBody)
        .then(() => {
          this.showRefreshDialog = false;
          this.isLoading = false;
        })
        .catch((e) => {
          console.log("ERROR: ", e);
          this.showRefreshDialog = false;
          this.isLoading = false;
        });
    },
    removeSchedules(luid) {
      let reqBody = { luid: luid };

      axios
        .post(
          "http://192.168.1.118:8000/webhookActions/remove_schedules/",
          reqBody
        )
        .then(() => {
          this.showSchedsDialog = false;
          this.isLoading = false;
        })
        .catch((e) => {
          console.log("ERROR: ", e);
          this.showSchedsDialog = false;
          this.isLoading = false;
        });
    },
    addTag(luid) {
      let reqBody = { luid: luid, event_type: this.$props.eventObj.event_type };

      axios
        .post("http://192.168.1.118:8000/webhookActions/add_tag/", reqBody)
        .then(() => {
          this.showTagDialog = false;
          this.isLoading = false;
        })
        .catch((e) => {
          console.log("ERROR: ", e);
          this.showTagDialog = false;
          this.isLoading = false;
        });
    },

    deleteItem(luid) {
      let reqBody = {};
      let endpoint = "";

      switch (this.$props.eventObj.event_type) {
        case "WorkbookRefreshFailed":
          reqBody.wb_luid = luid;
          endpoint = "delete_workbook";

          break;

        default:
          reqBody.ds_luid = luid;
          endpoint = "delete_datasource";
      }

      axios
        .post(`http://192.168.1.118:8000/webhookActions/${endpoint}/`, reqBody)
        .then(() => {
          this.showDeleteDialog = false;
          this.isLoading = false;
        })
        .catch((e) => {
          console.log("ERROR: ", e);
          this.showDeleteDialog = false;
          this.isLoading = false;
        });
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

.gray-background {
  background-color: #bdbdbd;
}

.card-link {
  color: blue;
  text-decoration: none;
}

.card-link:hover {
  text-decoration: underline;
}

.delete-title {
  background-color: #d32f2f;
}

.refresh-title {
  background-color: green;
}

.sched-title {
  background-color: #f9a825;
}

.tag-title {
  background-color: #03a9f4;
}

.close-icon {
  cursor: pointer;
}
</style>
