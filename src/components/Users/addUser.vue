<template>
  <div class="text-center">
    <v-dialog v-model="dialog" persistent scrollable width="600">
      <template v-slot:activator="{ on }">
        <v-btn depressed color="indigo" dark v-on="on" @click="getData"
          >Add User</v-btn
        >
      </template>
      <v-card v-if="dialog" class>
        <v-card-title
          class="google-font"
          style="border-bottom: 1px solid #e0e0e0"
          primary-title
          dark
          >Add User</v-card-title
        >
        <v-card-text class="px-5">
          <v-container fluid>
            <v-form ref="form" v-model="valid" lazy-validation>
              <v-row class="pa-0">
                <v-col cols="12" class="pa-1 ma-0">
                  <v-autocomplete
                    v-model="selectedUser"
                    :items="teamData"
                    outlined
                    :loading="loading"
                    chips
                    item-text="name"
                    item-value="id"
                    small-chips
                    label="Select Users"
                  >
                    <template v-slot:selection="{ item, index }">
                      <v-chip small v-if="index === 0">
                        <span>{{ item.name }}</span>
                      </v-chip>
                      <span v-if="index === 1" class="grey--text caption"
                        >(+{{ selectedUser.length - 1 }} others)</span
                      >
                    </template>
                  </v-autocomplete>
                </v-col>
                <v-col cols="12" class="pa-1 ma-0">
                  <v-select
                    :items="items"
                    v-model="userRole"
                    label="Role"
                    outlined
                  ></v-select>
                </v-col>
              </v-row>
            </v-form>
            <!-- <successDialog :data.sync="output"/> -->

            <div class="text-center">
              <v-dialog v-model="emailDialog" persistent scrollable width="500">
                <v-card v-if="emailDialog" class>
                  <v-card-title
                    class="google-font"
                    style="border-bottom: 1px solid #e0e0e0"
                    primary-title
                    dark
                    >Status</v-card-title
                  >
                  <v-card-text class="px-5">
                    <v-container fluid>
                      <v-row class="pa-0">
                        <v-col cols="12" class="pa-1 ma-0">
                          <p
                            :class="[
                              !output.data.success
                                ? 'red--text'
                                : 'green--text',
                            ]"
                            class="google-font heading my-0"
                          >
                            {{ output.data.msg }}
                          </p>
                          <p
                            v-if="output.data.emailstatus"
                            :class="[
                              !output.data.emailstatus.success
                                ? 'red--text'
                                : 'green--text',
                            ]"
                            class="google-font heading my-0 mt-1"
                          >
                            {{ output.data.emailstatus.msg }}
                          </p>
                        </v-col>
                      </v-row>
                    </v-container>
                  </v-card-text>

                  <v-divider></v-divider>

                  <v-card-actions class="grey lighten-4">
                    <div class="flex-grow-1"></div>
                    <v-btn color="indigo" text @click="close()">Close</v-btn>
                  </v-card-actions>
                </v-card>
              </v-dialog>
            </div>
          </v-container>
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions class="grey lighten-4">
          <div class="flex-grow-1"></div>
          <v-btn color="indigo" text @click="dialog = false">Close</v-btn>
          <v-btn
            color="indigo"
            dark
            @click="addUser"
            depressed
            :loading="loading"
            >Add</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import firebase from '@/config/firebase';
import TeamServices from '@/services/TeamServices';
import { mapState } from 'vuex';
export default {
  components: {},
  data: () => ({
    dialog: false,
    loading: false,
    items: ['Super Admin', 'Admin', 'Viewer'],
    userRole: '',
    valid: false,
    selectedUser: [],
    isAdding: false,
    teamData: [],
    output: {},
    emailDialog: false,
  }),
  computed: {
    ...mapState(['generalConfig']),
  },
  mounted() {},
  methods: {
    getData() {
      this.ShowTeam();
    },
    ShowTeam() {
      this.teamData = [];
      this.loading = true;
      TeamServices.getAllTeam()
        .then((res) => {
          if (res.success) {
            this.teamData = res.data;
          }
          this.loading = false;
        })
        .catch((e) => {
          this.loading = false;
          console.log('Error getting documents', e);
        });
    },
    close() {
      this.dialog = false;
      this.emailDialog = false;
      this.output.data.success
        ? this.$emit('showSuccess', 'Added Success')
        : this.$emit('failed', this.output.data.msg);
    },
    addUser() {
      this.loading = true;
      let userData = this.teamData.filter(
        (team) => team.id == this.selectedUser
      );
      userData[0]['userType'] = this.userRole;
      userData[0]['communityEmail'] = this.generalConfig.email;
      userData[0]['communityName'] = this.generalConfig.name;
      let appp = firebase.functions.httpsCallable('team-createAuthUser');
      appp(userData[0])
        .then((res1) => {
          // console.log(res1)
          this.output = res1;
          this.emailDialog = true;
          this.loading = false;
        })
        .catch((e) => {
          console.log(e);
          this.output = e;
          this.emailDialog = true;
          this.loading = false;
        });
    },
  },
};
</script>
