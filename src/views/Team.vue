<template>
  <v-container class="" style="max-width: 1600px">
    <Snakebar
      :message="snakeBarMessage"
      :isShow.sync="isSnakeBarVisible"
      :color="snakeBarColor"
      :timeout="snakeBarTimeOut"
    />
    <v-row class="">
      <v-col>
        <v-toolbar
          class="elevation-0"
          style="border: 1px solid #e0e0e0; border-radius: 5px"
        >
          <v-toolbar-title class="google-font mr-3"
            >Team: {{ teamData.length }}</v-toolbar-title
          >
          <v-spacer></v-spacer>

          <!-- Desktop -->
          <v-text-field
            flat
            dense
            v-model="search"
            solo-inverted
            hide-details
            prepend-inner-icon="mdi-search-web"
            label="Search"
            single-line
            class="mr-2 hidden-sm-and-down"
          ></v-text-field>
          <!-- Desktop -->

          <!-- Mobile -->
          <v-slide-x-reverse-transition>
            <v-text-field
              flat
              dense
              v-if="isSearch"
              v-model="search"
              solo-inverted
              hide-details
              prepend-inner-icon="mdi-search-web"
              label="Search"
              single-line
              class="mr-2 hidden-md-and-up"
            ></v-text-field>
          </v-slide-x-reverse-transition>

          <v-btn
            fab
            x-small
            color="indigo"
            @click="openCloseSearch"
            class="mr-2 hidden-md-and-up"
            outlined
            dark
          >
            <v-icon dark v-if="!isSearch">mdi-account-search</v-icon>
            <v-icon dark v-else>mdi-close</v-icon>
          </v-btn>
          <!-- Mobile -->
          &nbsp;

          <!-- Toggle Menu for View -->
          <v-btn-toggle
            v-if="teamData.length"
            borderless
            background-color="white"
            color="indigo"
            dense
            v-model="dataView"
            class="hidden-sm-and-down"
          >
            <v-tooltip bottom>
              <template v-slot:activator="{ on }">
                <v-btn v-on="on">
                  <v-icon>mdi-grid</v-icon>
                </v-btn>
              </template>
              <span>Grid View</span>
            </v-tooltip>

            <v-tooltip bottom>
              <template v-slot:activator="{ on }">
                <v-btn v-on="on">
                  <v-icon>mdi-format-list-bulleted</v-icon>
                </v-btn>
              </template>
              <span>Table View</span>
            </v-tooltip>
          </v-btn-toggle>
          <!-- Toggle Menu for View -->
          <AddTeam
            v-if="role == 'Super Admin'"
            class="ml-2"
            @showSuccess="showSnakeBar"
            @message="showMessageSnakeBar"
          />
        </v-toolbar>
      </v-col>
    </v-row>

    <!-- <v-row class="px-1">
      <v-col md="3" lg="2">
         <v-select
            :items="items"
            class="mr-2"
            label="Member Status"
            outlined
            dense
          ></v-select>
      </v-col>
      <v-col md="3" lg="2">
         <v-select
            :items="items"
            class="mr-2"
            label="Member Role"
            outlined
            dense
          ></v-select>
      </v-col>
      <v-col md="3" lg="2">
         <v-select
            :items="items"
            class="mr-2"
            label="Visibility"
            outlined
            dense
          ></v-select>
      </v-col>
    </v-row> -->

    <v-row class="px-2">
      <v-col cols="12 ">
        <v-container fluid class="pa-0">
          <v-row v-if="isLoading">
            <v-col col="12" sm="3" md="2" cols="6" v-for="n in 6" :key="n">
              <v-sheet
                :color="`grey ${theme.isDark ? 'darken-2' : 'lighten-4'}`"
                class=""
              >
                <v-skeleton-loader
                  class="mx-auto"
                  max-width="300"
                  type="card"
                ></v-skeleton-loader>
              </v-sheet>
            </v-col>
          </v-row>
          <div v-else>
            <!-- Check the Condition Where we have a Data or not -->
            <div v-if="teamData.length">
              <!-- Grid View -->
              <v-data-iterator
                :items="teamData"
                v-if="dataView == 0"
                :loading="isLoading"
                loading-text="Loading Team from Dir"
                :search="search"
                disable-pagination
                hide-default-footer
              >
                <template v-slot:default="props">
                  <v-row>
                    <v-col
                      col="12"
                      cols="6"
                      md="2"
                      lg="2"
                      sm="3"
                      v-for="item in props.items"
                      :key="item.id"
                      class="pa-1"
                    >
                      <v-card
                        style="
                          cursor: pointer;
                          user-select: none;
                          border: 1px solid #e0e0e0;
                          border-radius: 5px;
                        "
                        height="100%"
                        v-ripple
                        @click="gotoTeamDetails(item.id)"
                        class="text-center elevation-0"
                      >
                        <v-card-text style="height: 100%">
                          <v-badge
                            :color="item.active ? 'green' : 'red'"
                            dot
                            overlap
                            offset-y="16"
                            offset-x="25"
                          >
                            <v-avatar size="100">
                              <img
                                :src="
                                  item.image.length > 0
                                    ? item.image
                                    : require('@/assets/img/default_avatar.jpg')
                                "
                                alt
                              />
                            </v-avatar>
                          </v-badge>
                          <p
                            class="mt-3 mb-0 google-font black--text"
                            style="font-size: 120%"
                          >
                            {{ item.name }}
                          </p>
                          <p
                            class="mt-0 mb-0 google-font caption"
                            style="font-size: 60%"
                          >
                            {{ item.designation }}
                          </p>
                          <v-chip
                            class="ma-1"
                            dark
                            :color="item.visible ? 'green' : 'red'"
                            x-small
                            >{{ item.role }}</v-chip
                          >
                        </v-card-text>
                      </v-card>
                    </v-col>
                  </v-row>
                </template>
              </v-data-iterator>
              <!-- Grid View -->
              <!-- Table View -->
              <div v-else class="pa-0 ma-0">
                <v-row>
                  <v-col class="pa-1">
                    <v-data-table
                      :mobile-breakpoint="0"
                      style="
                        border: 1px solid #e0e0e0;
                        border-radius: 5px;
                        background: white;
                      "
                      :search="search"
                      :loading="isLoading"
                      :headers="headers"
                      :items="teamData"
                      sort-desc
                      :items-per-page="10"
                      class="elevation-0 ma-0 pa-0"
                    >
                      <template v-slot:item.name="{ item }">
                        <v-list-item>
                          <v-list-item-avatar>
                            <v-img
                              :src="
                                item.image.length > 0
                                  ? item.image
                                  : require('@/assets/img/default_avatar.jpg')
                              "
                            ></v-img>
                          </v-list-item-avatar>

                          <v-list-item-content>
                            <v-list-item-title
                              class="google-font"
                              v-html="item.name"
                            ></v-list-item-title>
                            <v-list-item-subtitle
                              class="google-font"
                              v-html="item.email"
                            ></v-list-item-subtitle>
                          </v-list-item-content>
                        </v-list-item>
                      </template>
                      <template v-slot:item.active="{ item }">
                        <v-chip
                          x-small
                          v-if="item.active == true"
                          color="success"
                          >Active</v-chip
                        >
                        <v-chip v-else x-small dark color="red">Inctive</v-chip>
                      </template>
                      <template v-slot:item.visible="{ item }">
                        <v-chip
                          x-small
                          v-if="item.visible == true"
                          color="success"
                          >Visible</v-chip
                        >
                        <v-chip v-else x-small dark color="red"
                          >Not Visible</v-chip
                        >
                      </template>
                      <template v-slot:item.actions="{ item }">
                        <v-tooltip bottom>
                          <template v-slot:activator="{ on }">
                            <v-btn
                              icon
                              v-on="on"
                              @click="gotoTeamDetails(item.id)"
                            >
                              <v-icon>mdi-eye</v-icon>
                            </v-btn>
                          </template>
                          <span>{{ item.name }} Details</span>
                        </v-tooltip>
                      </template>
                    </v-data-table>
                  </v-col>
                </v-row>
              </div>
              <!-- Table View -->
            </div>
            <!-- Check the Condition Where we have a Data or not -->

            <!-- Not Data Found -->
            <div v-else>
              <v-row justify="center" align="center">
                <v-col cols="12" md="12" class="pa-1">
                  <v-container
                    fluid
                    class=""
                    style="
                      border: 1px solid #e0e0e0;
                      border-radius: 5px;
                      background: white;
                    "
                  >
                    <v-row justify="center" align="center" class="pa-3">
                      <v-col md="4" class="text-center">
                        <img
                          style="width: 50%; text-align: center"
                          :src="require('@/assets/img/svg/DataNotFound.svg')"
                        />
                        <h1 class="google-font">Team Members Data Not Found</h1>
                        <p class="google-font">Kindly add Team member</p>
                        <br />
                        <AddTeam
                          class="ml-2"
                          v-if="role == 'Super Admin'"
                          @showSuccess="showSnakeBar"
                          @message="showMessageSnakeBar"
                        />
                      </v-col>
                    </v-row>
                  </v-container>
                </v-col>
              </v-row>
            </div>
            <!-- Not Data Found -->
          </div>
        </v-container>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import TeamServices from '@/services/TeamServices';
import { mapState } from 'vuex';
export default {
  name: 'TeamView',
  inject: ['theme'],
  components: {
    Snakebar: () => import('@/components/Common/Snakebar'),
    AddTeam: () => import('@/components/Team/AddTeam'),
  },
  computed: {
    ...mapState(['role']),
  },
  data: () => ({
    items: ['All', 'Active', 'Inactive'],
    dataView: 0,
    isSearch: false,
    search: '',
    snakeBarMessage: '',
    isSnakeBarVisible: false,
    snakeBarColor: 'green',
    snakeBarTimeOut: 5000,
    isLoading: false,
    showDialog: false,
    teamData: [],
    headers: [
      {
        text: 'Name',
        align: 'start',
        value: 'name',
        width: '25%',
      },
      { text: 'Role', value: 'role' },
      { text: 'Designation', value: 'designation' },
      { text: 'Status', value: 'active' },
      { text: 'Visible', value: 'visible' },
      { text: 'Actions', value: 'actions', sortable: false },
    ],
  }),
  mounted() {
    if (this.$route.query.msg) {
      this.showSnakeBar('Team Removed Sucessfully');
    } else this.showData();
  },
  methods: {
    changeSnakebar(vale) {
      this.isSnakeBarVisible = vale;
    },
    openCloseSearch() {
      this.isSearch = !this.isSearch;
      this.search = '';
    },
    showMessageSnakeBar(text) {
      this.snakeBarMessage = text;
      this.isSnakeBarVisible = true;
    },
    showSnakeBar(text) {
      this.snakeBarMessage = text;
      this.isSnakeBarVisible = true;
      this.showData();
    },
    gotoTeamDetails(id) {
      this.$router.push('/team/' + id);
    },
    showData() {
      this.teamData = [];
      this.isLoading = true;
      TeamServices.getAllTeam()
        .then((res) => {
          this.teamData = res.data;
          this.isLoading = false;
        })
        .catch((e) => {
          this.isLoading = false;
          console.log('Error getting documents', e);
        });
    },
  },
};
</script>
<style>
.v-badge--dot .v-badge__badge {
  border-radius: 6px;
  height: 12px;
  min-width: 0;
  padding: 0;
  width: 12px;
}
</style>
