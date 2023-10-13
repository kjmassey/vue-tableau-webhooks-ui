<template>
  <div class="d-flex w-100 justify-center">
    <v-card elevation="5" class="w-100" style="max-width: 1670px">
      <v-card-title class="py-4" style="background-color: #cfd8dc">
        <div class="d-flex justify-start align-center">
          <span class="text-h5"
            ><v-icon icon="mdi-view-list-outline" size="48"></v-icon>
            <strong>Tableau Webhooks Log</strong></span
          >
        </div></v-card-title
      >

      <v-card-text
        ><div class="d-flex py-4 justify-start">
          <v-select
            variant="outlined"
            density="compact"
            :items="filterEvents"
            label="Filter events..."
            placeholder="Nothing selected"
            hide-details
            clearable
            multiple
            class="w-50"
            style="max-width: 600px"
            v-model="selectedEvents"
            chips
            @update:model-value="getWebhookEvents()"
          >
          </v-select>
        </div>

        <v-data-table-server
          :items="hookEvents2"
          :headers="headers"
          :show-expand="true"
          :loading="isLoading"
          v-model:sort-by="sortBy"
          v-model:items-per-page="tableItemsPerPage"
          :items-length="tableTotalItems"
          v-model:page="tablePage"
          @update:items-per-page="getWebhookEvents()"
          @update:page="getWebhookEvents()"
          :hover="true"
        >
          <template #[`item.event_type`]="{ item }">
            <v-avatar size="42" color="blue-grey-lighten-5" class="mr-8 my-3">
              <v-icon
                :icon="getEventIcon(item)"
                :color="getIconColor(item.event_type)"
                size="24"
              />
            </v-avatar>
            {{ item.event_type }}</template
          >
          <template #[`item.site_name`]="{ item }">
            <a
              href="https://10ax.online.tableau.com/#/site/kjmdev797388"
              target="_blank"
              >{{ item.site_name }}</a
            ></template
          >
          <template #[`item.created_at`]="{ item }">
            {{ formattedTableDate(item.created_at) }}
          </template>
          <template v-slot:expanded-row="{ columns, item }">
            <td :colspan="columns.length" class="py-4">
              <!-- <div class="d-flex justify-center align-center w-100">
                <v-card style="width: 75%; max-width: 828px" elevation="15">
                  <v-card-title style="background-color: #03a9f4">{{
                    formatEventName(item.event_type)
                  }}</v-card-title>
                  <v-card-subtitle class="pt-2">
                    <div class="d-flex justify-space-between">
                      <span class="text-subtitle-2"
                        >A new workbook has been created.</span
                      >
                      <span>{{ formattedDate(item.created_at) }}</span>
                    </div>
                  </v-card-subtitle>
                  <v-card-text>
                    <div class="text-h6 wb-link pb-2">
                      <a :href="item.webpage_url" target="_blank">
                        {{ item.resource_name }}
                        <v-icon
                          icon="mdi-open-in-new"
                          size="16"
                          color="#03A9F4"
                        ></v-icon>
                      </a>
                    </div>
                    <div>
                      <span class="text-caption pr-2">Project Name:</span>
                      {{ item.project_name }}
                    </div>
                    <div>
                      <span class="text-caption pr-2">Owner:</span>
                      <a :href="`mailto:${item.owner_email}`" target="_blank">
                        {{ item.owner_name }}
                      </a>
                    </div>
                  </v-card-text>
                </v-card>
              </div> -->
              <UserExpandCard :eventObj="item" v-if="item.resource == 'USER'" />
              <DatasourceExpandCard
                :eventObj="item"
                v-if="item.resource == 'DATASOURCE'"
              />
              <WorkbookExpandCard
                :eventObj="item"
                v-if="item.resource == 'WORKBOOK'"
              />
              <ExtractsExpandCard
                :eventObj="item"
                v-if="item.resource == 'EXTRACTS'"
              />

              <ViewExpandCard v-if="item.resource == 'VIEW'" :eventObj="item" />
            </td>
          </template>
        </v-data-table-server>
      </v-card-text>
    </v-card>
  </div>
</template>

<script>
import { VDataTableServer } from "vuetify/labs/VDataTable";
import UserExpandCard from "@/components/UserExpandCard.vue";
import DatasourceExpandCard from "@/components/DatasourceExpandCard.vue";
import WorkbookExpandCard from "./WorkbookExpandCard.vue";
import ExtractsExpandCard from "./ExtractsExpandCard.vue";
import ViewExpandCard from "./ViewExpandCard.vue";
import axios from "axios";

export default {
  data: () => ({
    sortBy: [{ key: "created_at", order: "desc" }],
    isLoading: false,
    tableItemsPerPage: 10,
    tablePage: 1,
    tableTotalItems: 0,
    headers: [
      { title: "id", key: "id", align: " d-none" },
      { title: "Event Type", key: "event_type", sortable: false },
      { title: "Content Type", key: "resource", sortable: false },
      { title: "Site Name", key: "site_name", sortable: false },
      { title: "Project Name", key: "project_name", sortable: false },
      { title: "Event Date", key: "created_at", sortable: false },
      { title: "Details", key: "data-table-expand" },
    ],
    hookEvents: [
      { id: 1, event_name: "Something that happened...." },
      { id: 2, event_name: "Another something that happened...." },
    ],
    hookEvents2: [],
    filterEvents: [
      "AdminDemoted",
      "AdminPromoted",
      "DatasourceCreated",
      "DatasourceDeleted",
      "DatasourceRefreshFailed",
      "DatasourceRefreshStarted",
      "DatasourceRefreshSucceeded",
      "DatasourceUpdated",
      "UserDeleted",
      "ViewDeleted",
      "WorkbookCreated",
      "WorkbookDeleted",
      "WorkbookRefreshFailed",
      "WorkbookRefreshStarted",
      "WorkbookRefreshSucceeded",
      "WorkbookUpdated",
    ],
    selectedEvents: [],
    // hookEvents2: [
    //   {
    //     id: 1,
    //     site_luid: "f8dbec1d-2217-42b0-868a-229a2ff090bb",
    //     resource: "WORKBOOK",
    //     resource_luid: "d39a37f6-6d98-4692-a93b-92e01f5edbc5",
    //     resource_name: "trigger2",
    //     created_at: "2023-10-06T19:12:33Z",
    //     text: "Event WorkbookCreated for resource trigger2: d39a37f6-6d98-4692-a93b-92e01f5edbc5 on site f8dbec1d-2217-42b0-868a-229a2ff090bb posted at 2023-10-06T19:12:33.242Z",
    //     webpage_url:
    //       "https://10ax.online.tableau.com/#/site/kjmdev797388/workbooks/177417",
    //     project_luid: "934cd2d2-0b32-40a3-ae15-b3ea1c0e33c0",
    //     project_name: "23 Nov DataDev Day",
    //     owner_luid: "9ac19678-6f50-4e21-84d9-b77def14d0cd",
    //     site_name: "KJM-dev-797388",
    //     owner_name: "Kyle Massey",
    //     owner_email: "kyle.massey@gmail.com",
    //     owner_site_role: "SiteAdministratorCreator",
    //     event_type: "WorkbookCreated",
    //   },
    //   {
    //     id: 2,
    //     site_luid: "f8dbec1d-2217-42b0-868a-229a2ff090bb",
    //     resource: "WORKBOOK",
    //     resource_luid: "d39a37f6-6d98-4692-a93b-92e01f5edbc5",
    //     resource_name: "trigger2",
    //     created_at: "2023-10-06T19:12:33Z",
    //     text: "Event WorkbookCreated for resource trigger2: d39a37f6-6d98-4692-a93b-92e01f5edbc5 on site f8dbec1d-2217-42b0-868a-229a2ff090bb posted at 2023-10-06T19:12:33.242Z",
    //     webpage_url:
    //       "https://10ax.online.tableau.com/#/site/kjmdev797388/workbooks/177417",
    //     project_luid: "934cd2d2-0b32-40a3-ae15-b3ea1c0e33c0",
    //     project_name: "23 Nov DataDev Day",
    //     owner_luid: "9ac19678-6f50-4e21-84d9-b77def14d0cd",
    //     site_name: "KJM-dev-797388",
    //     owner_name: "Kyle Massey",
    //     owner_email: "kyle.massey@gmail.com",
    //     owner_site_role: "SiteAdministratorCreator",
    //     event_type: "WorkbookCreated",
    //   },
    //   {
    //     id: 3,
    //     site_luid: "f8dbec1d-2217-42b0-868a-229a2ff090bb",
    //     resource: "WORKBOOK",
    //     resource_luid: "d39a37f6-6d98-4692-a93b-92e01f5edbc5",
    //     resource_name: "trigger2",
    //     created_at: "2023-10-06T19:12:33Z",
    //     text: "Event WorkbookCreated for resource trigger2: d39a37f6-6d98-4692-a93b-92e01f5edbc5 on site f8dbec1d-2217-42b0-868a-229a2ff090bb posted at 2023-10-06T19:12:33.242Z",
    //     webpage_url:
    //       "https://10ax.online.tableau.com/#/site/kjmdev797388/workbooks/177417",
    //     project_luid: "934cd2d2-0b32-40a3-ae15-b3ea1c0e33c0",
    //     project_name: "23 Nov DataDev Day",
    //     owner_luid: "9ac19678-6f50-4e21-84d9-b77def14d0cd",
    //     site_name: "KJM-dev-797388",
    //     owner_name: "Kyle Massey",
    //     owner_email: "kyle.massey@gmail.com",
    //     owner_site_role: "SiteAdministratorCreator",
    //     event_type: "WorkbookCreated",
    //   },
    //   {
    //     id: 4,
    //     site_luid: "f8dbec1d-2217-42b0-868a-229a2ff090bb",
    //     resource: "WORKBOOK",
    //     resource_luid: "0c7ba112-7b5c-4fd3-a8b3-05521f019a17",
    //     resource_name: "trigger2",
    //     created_at: "2023-10-09T17:44:57Z",
    //     text: "Event WorkbookCreated for resource trigger2: 0c7ba112-7b5c-4fd3-a8b3-05521f019a17 on site f8dbec1d-2217-42b0-868a-229a2ff090bb posted at 2023-10-09T17:44:56.941Z",
    //     webpage_url:
    //       "https://10ax.online.tableau.com/#/site/kjmdev797388/workbooks/179262",
    //     project_luid: "934cd2d2-0b32-40a3-ae15-b3ea1c0e33c0",
    //     project_name: "23 Nov DataDev Day",
    //     owner_luid: "9ac19678-6f50-4e21-84d9-b77def14d0cd",
    //     site_name: "KJM-dev-797388",
    //     owner_name: "Kyle Massey",
    //     owner_email: "kyle.massey@gmail.com",
    //     owner_site_role: "SiteAdministratorCreator",
    //     event_type: "WorkbookCreated",
    //   },
    //   {
    //     id: 5,
    //     site_luid: "f8dbec1d-2217-42b0-868a-229a2ff090bb",
    //     resource: "WORKBOOK",
    //     resource_luid: "2235a537-1dbb-47ec-b1dd-74486049a159",
    //     resource_name: "trigger from somewhere else",
    //     created_at: "2023-10-09T17:48:26Z",
    //     text: "Event WorkbookCreated for resource trigger from somewhere else: 2235a537-1dbb-47ec-b1dd-74486049a159 on site f8dbec1d-2217-42b0-868a-229a2ff090bb posted at 2023-10-09T17:48:25.991Z",
    //     webpage_url:
    //       "https://10ax.online.tableau.com/#/site/kjmdev797388/workbooks/179263",
    //     project_luid: "10b57785-c989-4949-93ec-716d19553da2",
    //     project_name: "Archive",
    //     owner_luid: "9ac19678-6f50-4e21-84d9-b77def14d0cd",
    //     site_name: "KJM-dev-797388",
    //     owner_name: "Kyle Massey",
    //     owner_email: "kyle.massey@gmail.com",
    //     owner_site_role: "SiteAdministratorCreator",
    //     event_type: "WorkbookCreated",
    //   },
    //   {
    //     id: 6,
    //     site_luid: "f8dbec1d-2217-42b0-868a-229a2ff090bb",
    //     resource: "WORKBOOK",
    //     resource_luid: "6d901061-ba09-4118-aa3d-c134f614d294",
    //     resource_name: "Extract workbook 2",
    //     created_at: "2023-10-09T17:59:16Z",
    //     text: "Event WorkbookDeleted for resource Extract workbook 2: 6d901061-ba09-4118-aa3d-c134f614d294 on site f8dbec1d-2217-42b0-868a-229a2ff090bb posted at 2023-10-09T17:59:15.607Z",
    //     webpage_url: null,
    //     project_luid: null,
    //     project_name: null,
    //     owner_luid: null,
    //     site_name: "KJM-dev-797388",
    //     owner_name: null,
    //     owner_email: null,
    //     owner_site_role: null,
    //     event_type: "WorkbookDeleted",
    //   },
    //   {
    //     id: 7,
    //     site_luid: "f8dbec1d-2217-42b0-868a-229a2ff090bb",
    //     resource: "WORKBOOK",
    //     resource_luid: "96e054bc-a0d0-401a-a6ca-e9892e883beb",
    //     resource_name: "Regional Manager Performance",
    //     created_at: "2023-10-09T19:30:12Z",
    //     text: "Event WorkbookDeleted for resource Regional Manager Performance: 96e054bc-a0d0-401a-a6ca-e9892e883beb on site f8dbec1d-2217-42b0-868a-229a2ff090bb posted at 2023-10-09T19:30:12.287Z",
    //     webpage_url: null,
    //     project_luid: null,
    //     project_name: null,
    //     owner_luid: null,
    //     site_name: "KJM-dev-797388",
    //     owner_name: null,
    //     owner_email: null,
    //     owner_site_role: null,
    //     event_type: "WorkbookDeleted",
    //   },
    // ],
  }),
  computed: {
    computedHeaders() {
      let headers = [];

      if (this.hookEvents2.length > 0) {
        Object.keys(this.hookEvents2[0]).forEach((e) => {
          headers.push({ title: e, key: e });
        });
      }

      return headers;
    },
  },

  components: {
    VDataTableServer,
    UserExpandCard,
    DatasourceExpandCard,
    WorkbookExpandCard,
    ExtractsExpandCard,
    ViewExpandCard,
  },
  methods: {
    getIconColor(eventName) {
      if (
        eventName.includes("Created") ||
        eventName.includes("Promoted") ||
        eventName.includes("Succeeded")
      ) {
        return "blue";
      } else if (eventName.includes("RefreshStarted")) {
        return "grey";
      } else {
        return "red";
      }
    },
    getWebhookEvents() {
      this.isLoading = true;
      let pageSize =
        this.tableItemsPerPage == -1
          ? this.tableTotalItems
          : this.tableItemsPerPage;

      let pageNum = this.tableItemsPerPage == -1 ? 1 : this.tablePage;
      let eventParamStr = "";

      if (this.selectedEvents.length > 0) {
        eventParamStr = `&event_type=${this.selectedEvents.join(",")}`;
      }

      axios
        .get(
          `http://192.168.1.118:8000/webhookEvents/?page_size=${pageSize}&p=${pageNum}${eventParamStr}`
        )
        .then((resp) => {
          this.hookEvents2 = resp.data.results;
          this.tableTotalItems = resp.data.count;

          this.isLoading = false;
        });
    },
    getEventIcon(item) {
      let iconMap = {
        WORKBOOK: {
          WorkbookCreated: "mdi-file-plus",
          WorkbookDeleted: "mdi-file-remove",
          WorkbookUpdated: "mdi-file-edit",
          WorkbookRefreshStarted: "mdi-file-refresh",
        },
        DATASOURCE: {
          DatasourceCreated: "mdi-database-plus",
          DatasourceDeleted: "mdi-database-minus",
          DatasourceUpdated: "mdi-database-edit",
        },
        VIEW: {
          ViewDeleted: "mdi-eye-remove",
        },
        USER: {
          AdminDemoted: "mdi-account-supervisor",
          AdminPromoted: "mdi-account-supervisor",
          UserDeleted: "mdi-account-minus",
        },
        EXTRACTS: {
          WorkbookRefreshStarted: "mdi-file-refresh",
          WorkbookRefreshSucceeded: "mdi-check-outline",
          WorkbookRefreshFailed: "mdi-alert-circle",
          DatasourceRefreshStarted: "mdi-database-sync",
          DatasourceRefreshSucceeded: "mdi-check-outline",
          DatasourceRefreshFailed: "mdi-alert-circle",
        },
      };

      if (item.resource in iconMap) {
        if (item.event_type in iconMap[item.resource]) {
          return iconMap[item.resource][item.event_type];
        } else {
          return "mdi-help";
        }
      } else {
        return "mdi-help";
      }

      // if (eventName.includes("Created")) {
      //   return "mdi-note-plus";
      // } else {
      //   return "mdi-note-minus";
      // }
    },
    formatEventName(eventName) {
      let newStr = "";

      newStr = eventName.replace(/([a-z])([A-Z])/g, "$1 $2");
      newStr = newStr.replace(/([A-Z])([A-Z][a-z])/g, "$1 $2");

      return newStr;
    },
    formattedTableDate(eventDate) {
      let formattedDate = new Date(eventDate);

      let dateStr = formattedDate.toLocaleDateString("en-us", {
        year: "numeric",
        month: "numeric",
        day: "numeric",
      });

      let timeStr = formattedDate.toLocaleTimeString("en-us", {
        hour: "2-digit",
        minute: "2-digit",
      });

      return `${dateStr} - ${timeStr}`;
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
  mounted() {
    this.getWebhookEvents();
  },
};
</script>

<style>
.wb-link a {
  text-decoration: none;
  color: black;
}

.wb-link a:hover {
  text-decoration: underline;
}
</style>
