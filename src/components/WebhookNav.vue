<template>
  <v-app-bar elevation="2" color="blue-grey-darken-1" class="px-4"
    ><div></div>
    <v-icon icon="mdi-chart-line" size="36"></v-icon>
    <div class="text-h6 px-4">COE Admin Portal</div>
    <v-spacer />
    <div>
      <v-icon
        icon="mdi-information"
        size="36"
        class="cursor-hover"
        @click="showInfoDialog = true"
      ></v-icon></div
  ></v-app-bar>
  <v-scale-transition>
    <v-dialog
      v-model="showInfoDialog"
      style="max-width: 675px; min-width: 575px"
    >
      <v-card>
        <v-card-title class="d-flex">
          About This Portal <v-spacer /><v-btn
            icon="mdi-close"
            class="cursor-hover"
            density="compact"
            flat
            :ripple="false"
            @click="showInfoDialog = false"
          >
          </v-btn>
        </v-card-title>
        <v-card-subtitle
          ><span class="text-caption"
            >Developed by Kyle Massey for Tableau DataDev Day November
            2023</span
          >
        </v-card-subtitle>
        <v-card-text class="text-body-2">
          <div>
            This mock administrative portal was created to show how you can
            leverage the power of Tableau and its APIs - alongside other web
            technologies - to gain actionable insights into activities and
            events that are occurring with your Tableau content/site(s).
          </div>
          <div class="pt-4">
            Please feel free to borrow, steal, adapt and modify any of the
            source code, which is available here.
          </div>
        </v-card-text>
        <v-card-actions>
          <div class="w-100">
            <div class="text-caption pb-2 px-4"><em>Tools used:</em></div>
            <div
              class="d-flex justify-space-between align-center cursor-hover w-100 px-4 pb-4"
            >
              <v-tooltip
                text="Tableau/Tableau REST API/Tableau Webhooks"
                location="bottom"
              >
                <template v-slot:activator="{ props }">
                  <v-img
                    src="../../public/tableau.png"
                    :max-width="36"
                    :max-height="36"
                    cover
                    v-bind="props"
                    @click="openURL('http://tableau.com/developer')"
                  >
                  </v-img>
                </template>
              </v-tooltip>

              <v-tooltip text="AWS Lambda" location="bottom">
                <template v-slot:activator="{ props }">
                  <v-img
                    src="../../public/aws_lambda.png"
                    :max-width="36"
                    :max-height="36"
                    cover
                    v-bind="props"
                    @click="openURL('https://aws.amazon.com/pm/lambda')"
                  >
                  </v-img>
                </template>
              </v-tooltip>

              <v-tooltip text="AWS RDS (MySQL)" location="bottom">
                <template v-slot:activator="{ props }">
                  <v-img
                    src="../../public/aws_rds.png"
                    :max-width="36"
                    :max-height="36"
                    cover
                    v-bind="props"
                    @click="openURL('https://aws.amazon.com/rds/')"
                  >
                  </v-img>
                </template>
              </v-tooltip>

              <v-tooltip text="Django" location="bottom">
                <template v-slot:activator="{ props }">
                  <v-img
                    src="../../public/django-icon.png"
                    :max-width="36"
                    :max-height="36"
                    cover
                    v-bind="props"
                    @click="openURL('https://www.djangoproject.com/')"
                  >
                  </v-img>
                </template>
              </v-tooltip>

              <v-tooltip text="Vue.js" location="bottom">
                <template v-slot:activator="{ props }">
                  <v-img
                    src="../../public/vue.png"
                    :max-width="36"
                    :max-height="36"
                    cover
                    v-bind="props"
                    @click="openURL('https://vuejs.org')"
                  >
                  </v-img>
                </template>
              </v-tooltip>

              <v-tooltip text="Vuetify" location="bottom">
                <template v-slot:activator="{ props }">
                  <v-img
                    src="../../public/vuetify.png"
                    :max-width="36"
                    :max-height="36"
                    cover
                    v-bind="props"
                    @click="openURL('https://vuetifyjs.com/en/')"
                  >
                  </v-img>
                </template>
              </v-tooltip>
            </div>
          </div>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-scale-transition>
</template>

<script>
export default {
  data: () => ({
    showInfoDialog: true,
  }),
  methods: {
    openURL(url) {
      window.open(url, "_blank");
    },
  },
};
</script>

<style>
.cursor-hover {
  cursor: pointer;
}
</style>
