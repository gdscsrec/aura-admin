<template>
  <div class="text-center">
    <v-dialog
      v-model="dialog"
      fullscreen
      hide-overlay
      transition="dialog-bottom-transition"
      scrollable
    >
      <template v-slot:activator="{}">
        <v-tooltip bottom>
          <template v-slot:activator="{ on }">
            <v-btn
              fab
              x-small
              color="indigo"
              outlined
              dark
              v-on="on"
              @click.stop="dialog = true"
            >
              <v-icon dark>mdi-plus</v-icon>
            </v-btn>
          </template>
          <span>Add New Custom Event</span>
        </v-tooltip>
      </template>
      <v-card v-if="dialog" class style>
        <v-toolbar color="white">
          <v-btn icon @click="dialog = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
          <v-toolbar-title class="google-font"
            >New Custom Event</v-toolbar-title
          >
          <v-spacer></v-spacer>
          <v-btn
            color="indigo"
            :loading="loading"
            depressed
            dark
            @click="SaveEvent"
            >Save Event</v-btn
          >
        </v-toolbar>
        <v-card-text class="px-1">
          <v-container fluid class style>
            <v-form ref="form" v-model="valid" lazy-validation>
              <v-row justify="center" align="start">
                <v-col md="3" lg="2" cols="12" sm="3">
                  <img
                    style="width: 100%; text-align: center"
                    :src="require('@/assets/img/svg/dataentry.svg')"
                  />
                  <h3 class="google-font">New Custom Event</h3>
                  <p class="google-font mb-0" style="color: red">
                    *indicates required field
                  </p>
                  <p>Event ID should be Unique</p>
                  <p style="color: red">
                    Once you assigned an ID to event, it can't be changed
                  </p>
                </v-col>
                <v-col md="8" lg="9" cols="12" sm="8">
                  <v-row>
                    <v-col class="ma-0" md="12" cols="12">
                      <h4 class="google-font mb-0">Event Status</h4>
                    </v-col>
                    <v-col md="3" cols="6" class="ma-0">
                      <v-select
                        :items="items"
                        v-model="eventData.active"
                        label="Active Status*"
                        outlined
                      ></v-select>
                    </v-col>

                    <v-col md="3" cols="6" class="ma-0">
                      <v-select
                        :items="items"
                        v-model="eventData.visible"
                        label="Visiblity Status*"
                        outlined
                      ></v-select>
                    </v-col>

                    <v-col md="3" xs="12" cols="12" class="ma-0">
                      <v-text-field
                        v-model="eventData.id"
                        class="ma-0"
                        :rules="idRules"
                        label="Event ID*"
                        type="text"
                        outlined
                      ></v-text-field>
                    </v-col>
                  </v-row>

                  <v-row>
                    <v-col class="ma-0" md="12" cols="12">
                      <h4 class="google-font mb-0">Event Info</h4>
                    </v-col>

                    <v-col md="5" xs="3" cols="12" class="ma-0">
                      <v-text-field
                        v-model="eventData.name"
                        class="ma-0"
                        :rules="idRules"
                        label="Event Name*"
                        type="text"
                        outlined
                      ></v-text-field>
                    </v-col>

                    <v-col md="3" xs="3" cols="12" class="ma-0">
                      <v-menu
                        ref="menu"
                        v-model="menu"
                        :close-on-content-click="false"
                        :return-value.sync="eventData.date"
                        transition="scale-transition"
                        offset-y
                        min-width="290px"
                      >
                        <template v-slot:activator="{ on }">
                          <v-text-field
                            v-model="eventData.date"
                            label="Date *"
                            outlined
                            v-on="on"
                          ></v-text-field>
                        </template>
                        <v-date-picker
                          v-model="eventData.date"
                          no-title
                          scrollable
                        >
                          <v-spacer></v-spacer>
                          <v-btn text color="primary" @click="menu = false"
                            >Cancel</v-btn
                          >
                          <v-btn
                            text
                            color="primary"
                            @click="$refs.menu.save(eventData.date)"
                            >OK</v-btn
                          >
                        </v-date-picker>
                      </v-menu>
                    </v-col>

                    <v-col md="2" xs="3" cols="12" class="ma-0">
                      <v-dialog
                        ref="dialog"
                        v-model="modal2"
                        :return-value.sync="eventData.time.starttime"
                        persistent
                        width="290px"
                      >
                        <template v-slot:activator="{ on }">
                          <v-text-field
                            v-model="eventData.time.starttime"
                            label="Start Time"
                            readonly
                            outlined
                            v-on="on"
                          ></v-text-field>
                        </template>
                        <v-time-picker
                          v-if="modal2"
                          v-model="eventData.time.starttime"
                          full-width
                        >
                          <v-spacer></v-spacer>
                          <v-btn text color="primary" @click="modal2 = false"
                            >Cancel</v-btn
                          >
                          <v-btn
                            text
                            color="primary"
                            @click="$refs.dialog.save(eventData.time.starttime)"
                            >OK</v-btn
                          >
                        </v-time-picker>
                      </v-dialog>
                    </v-col>

                    <v-col md="2" xs="3" cols="12" class="ma-0">
                      <v-dialog
                        ref="dialog1"
                        v-model="modal1"
                        :return-value.sync="eventData.time.endtime"
                        persistent
                        width="290px"
                      >
                        <template v-slot:activator="{ on }">
                          <v-text-field
                            v-model="eventData.time.endtime"
                            label="End Time"
                            readonly
                            outlined
                            v-on="on"
                          ></v-text-field>
                        </template>
                        <v-time-picker
                          v-if="modal1"
                          v-model="eventData.time.endtime"
                          full-width
                        >
                          <v-spacer></v-spacer>
                          <v-btn text color="primary" @click="modal1 = false"
                            >Cancel</v-btn
                          >
                          <v-btn
                            text
                            color="primary"
                            @click="$refs.dialog1.save(eventData.time.endtime)"
                            >OK</v-btn
                          >
                        </v-time-picker>
                      </v-dialog>
                    </v-col>
                    <v-col md="4" xs="4" cols="12" class="ma-0">
                      <v-text-field
                        v-model="eventData.venue.name"
                        class="ma-0"
                        label="Venue"
                        outlined
                      ></v-text-field>
                    </v-col>
                    <v-col md="8" xs="4" cols="12" class="ma-0">
                      <v-text-field
                        v-model="eventData.venue.googlemapsurl"
                        class="ma-0"
                        label="Venue Google Maps Link"
                        outlined
                      ></v-text-field>
                    </v-col>
                    <v-col md="7" xs="7" cols="6" class="ma-0">
                      <v-text-field
                        v-model="eventData.image"
                        class="ma-0"
                        label="Image URL"
                        outlined
                      ></v-text-field>
                    </v-col>
                    <v-col md="4" xs="4" cols="6" class="ma-0">
                      <UploadImage
                        type="events"
                        :userId="eventData.id"
                        @message="showMessageSnakeBar"
                        @uploadedImage="imageUploadDone"
                      />
                    </v-col>
                    <v-col md="12" xs="12" cols="12" class="ma-0">
                      <v-combobox
                        chips
                        v-model="eventData.hashtags"
                        clearable
                        label="Event Hashtags"
                        multiple
                        outlined
                      >
                        <template
                          v-slot:selection="{ attrs, item, select, selected }"
                        >
                          <v-chip
                            v-bind="attrs"
                            :input-value="selected"
                            close
                            @click="select"
                            @click:close="remove(item)"
                          >
                            <strong class="google-font">{{ item }}</strong>
                          </v-chip>
                        </template>
                      </v-combobox>
                    </v-col>

                    <v-col md="12" xs="12" cols="12" class="ma-0">
                      <v-textarea
                        outlined
                        name="input-7-4"
                        v-model="eventData.des"
                        label="Event Description"
                      ></v-textarea>
                    </v-col>
                  </v-row>

                  <v-row>
                    <v-col class="ma-0" md="12" cols="12">
                      <h4 class="google-font mb-0">
                        Speaker, Partners & Team Info
                      </h4>
                    </v-col>
                    <v-col md="6" xs="6" lg="4" cols="12" class="ma-0">
                      <v-autocomplete
                        v-model="eventData.speakers"
                        :items="speakersData"
                        outlined
                        item-text="name"
                        item-value="id"
                        label="Select Speaker"
                        multiple
                      >
                        <template v-slot:selection="{ item, index }">
                          <v-chip small v-if="index === 0">
                            <span>{{ item.name }}</span>
                          </v-chip>
                          <span v-if="index === 1" class="grey--text caption"
                            >(+{{ speakersData.length - 1 }} others)</span
                          >
                        </template>
                      </v-autocomplete>
                    </v-col>
                    <v-col md="6" xs="6" lg="4" cols="12" class="ma-0">
                      <v-autocomplete
                        v-model="eventData.partners"
                        :items="partnersData"
                        outlined
                        item-text="name"
                        item-value="id"
                        label="Select Partners"
                        multiple
                      >
                        <template v-slot:selection="{ item, index }">
                          <v-chip small v-if="index === 0">
                            <span>{{ item.name }}</span>
                          </v-chip>
                          <span v-if="index === 1" class="grey--text caption"
                            >(+{{ partnersData.length - 1 }} others)</span
                          >
                        </template>
                      </v-autocomplete>
                    </v-col>
                    <v-col md="6" xs="6" lg="4" cols="12" class="ma-0">
                      <v-autocomplete
                        v-model="eventData.team"
                        :items="teamData"
                        outlined
                        chips
                        item-text="name"
                        item-value="id"
                        small-chips
                        label="Select Team"
                        multiple
                      >
                        <template v-slot:selection="{ item, index }">
                          <v-chip small v-if="index === 0">
                            <span>{{ item.name }}</span>
                          </v-chip>
                          <span v-if="index === 1" class="grey--text caption"
                            >(+{{ eventData.team.length - 1 }} others)</span
                          >
                        </template>
                      </v-autocomplete>
                    </v-col>
                  </v-row>

                  <v-row>
                    <v-col class="ma-0" md="12" cols="12">
                      <h4 class="google-font mb-0">Event Links Info</h4>
                    </v-col>
                    <v-col md="4" xs="4" lg="4" cols="12" class="ma-0">
                      <v-text-field
                        v-model="eventData.links.meetup"
                        class="ma-0"
                        label="Event Meetup URL"
                        outlined
                      ></v-text-field>
                    </v-col>
                    <v-col md="4" xs="4" lg="4" cols="12" class="ma-0">
                      <v-text-field
                        v-model="eventData.links.registration"
                        class="ma-0"
                        label="Event Registration Link"
                        outlined
                      ></v-text-field>
                    </v-col>
                    <v-col md="4" xs="4" lg="4" cols="12" class="ma-0">
                      <v-text-field
                        v-model="eventData.links.facebook"
                        class="ma-0"
                        label="Event Facebook Page Link"
                        outlined
                      ></v-text-field>
                    </v-col>
                    <v-col md="4" xs="4" lg="4" cols="12" class="ma-0">
                      <v-text-field
                        v-model="eventData.links.feedback"
                        class="ma-0"
                        label="Event Feedback Link"
                        outlined
                      ></v-text-field>
                    </v-col>
                    <v-col md="4" xs="4" lg="4" cols="12" class="ma-0">
                      <v-text-field
                        v-model="eventData.links.callforspeaker"
                        class="ma-0"
                        label="Call For Speaker Link"
                        outlined
                      ></v-text-field>
                    </v-col>
                    <v-col md="4" xs="4" lg="4" cols="12" class="ma-0">
                      <v-text-field
                        v-model="eventData.links.youtube"
                        class="ma-0"
                        label="Youtube Live URL"
                        outlined
                      ></v-text-field>
                    </v-col>
                  </v-row>

                  <v-row class>
                    <v-col class="ma-0" md="12" cols="12" style>
                      <h4 class="google-font mb-0">Event Agenda Info</h4>
                    </v-col>
                    <v-col class="ma-0" md="12" cols="12">
                      <v-toolbar
                        class="elevation-0"
                        style="border: 1px solid #e0e0e0; border-radius: 5px"
                      >
                        <v-toolbar-title class="google-font mr-3"
                          >Event Agenda</v-toolbar-title
                        >
                        <v-spacer></v-spacer>
                        <AddNewAgenda :data.sync="eventData.agenda" />
                      </v-toolbar>
                    </v-col>

                    <v-col cols="12" v-if="eventData.agenda.length <= 0" class>
                      <v-img
                        :src="require('@/assets/img/svg/DataNotFound.svg')"
                        :height="150"
                        contain
                      ></v-img>
                      <p class="google-font my-0 py-0 mb-2 text-center">
                        No Agenda found
                      </p>
                    </v-col>

                    <v-col cols="12" v-else>
                      <v-row>
                        <v-col md="12" class="my-1 py-0">
                          <v-data-table
                            :headers="headers"
                            mobile-breakpoint="0"
                            :items.sync="eventData.agenda"
                            :items-per-page="5"
                            class="elevation-0 lightModeCard"
                          >
                            <template v-slot:item.actions="{ item }">
                              <EditAgenda :data.sync="item" />
                              <v-btn
                                fab
                                x-small
                                color="indigo"
                                class="mx-1"
                                outlined
                                dark
                              >
                                <v-icon @click="deleteData(idx)"
                                  >mdi-delete</v-icon
                                >
                              </v-btn>
                              <!-- <RemoveAgenda :data.sync = "item"/> -->
                            </template>
                          </v-data-table>
                        </v-col>
                      </v-row>
                    </v-col>
                  </v-row>
                </v-col>
              </v-row>
            </v-form>
          </v-container>
        </v-card-text>

        <v-divider></v-divider>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import { mapState } from 'vuex';
import TeamServices from '@/services/TeamServices';
import PartnersServices from '@/services/PartnersServices';
import SpeakerServices from '@/services/SpeakersServices';
import CustomEventServices from '@/services/CustomEventServices';
export default {
  components: {
    AddNewAgenda: () => import('@/components/Events/CustomEvents/AddNewAgenda'),
    EditAgenda: () => import('@/components/Events/CustomEvents/EditAgenda'),
    UploadImage: () => import('@/components/Common/ImageUpload'),
  },
  props: [],
  computed: { ...mapState(['userDetails']) },
  data() {
    return {
      menu: false,
      modal2: false,
      modal1: false,
      headers: [
        { text: 'Start Time', value: 'starttime' },
        { text: 'End Time', value: 'endtime' },
        { text: 'Title', value: 'title' },
        { text: 'Description', value: 'des' },
        { text: 'Actions', sortable: false, value: 'actions' },
      ],
      imagePre: '',
      valid: true,
      idRules: [
        (v) => !!v || 'Field Value is required',
        (v) =>
          (v && v.length <= 100) || 'Name must be less than 100 characters',
      ],
      nameRules: [
        (v) => !!v || 'Name is required',
        (v) => (v && v.length <= 100) || 'Name must be less than 50 characters',
      ],
      emailRules: [
        (v) => !!v || 'E-mail is required',
        (v) => /.+@.+\..+/.test(v) || 'E-mail must be valid',
      ],
      teamRole: ['Organizing Team', 'Core Team', 'Volunteer'],
      dialog: false,
      loading: false,
      items: [true, false],
      eventData: {
        id: '',
        active: Boolean,
        visible: Boolean,
        name: '',
        image: '',
        date: '',
        des: '',
        venue: {
          name: '',
          googlemapsurl: '',
        },
        links: {
          meetup: '',
          facebook: '',
          registration: '',
          feedback: '',
          callforspeaker: '',
          youtube: '',
        },
        time: {
          starttime: '',
          endtime: '',
        },
        hashtags: [],
        speakers: [],
        partners: [],
        team: [],
        agenda: [],
      },
      speakersData: [],
      partnersData: [],
      teamData: [],
    };
  },
  mounted() {
    this.ShowSpeakers();
    this.ShowPartners();
    this.ShowTeam();
  },
  methods: {
    showMessageSnakeBar(text) {
      this.$emit('message', text);
    },
    imageUploadDone(text) {
      this.eventData.image = text;
    },
    deleteData(index) {
      this.eventData.agenda.splice(index, 1);
    },
    ShowSpeakers() {
      this.speakersData = [];
      SpeakerServices.getAllSpeakers()
        .then((res) => {
          if (res.success == true) {
            this.speakersData = res.data;
          }
        })
        .catch((e) => {
          console.log('Error getting documents', e);
        });
    },
    ShowPartners() {
      this.partnersData = [];
      PartnersServices.getAllPartners()
        .then((res) => {
          if (res.success == true) {
            this.partnersData = res.data;
          }
        })
        .catch((e) => {
          console.log('Error getting documents', e);
        });
    },
    ShowTeam() {
      this.teamData = [];
      TeamServices.getAllTeam()
        .then((res) => {
          this.teamData = res.data;
        })
        .catch((e) => {
          console.log('Error getting documents', e);
        });
    },
    remove(item) {
      this.eventData.hashtags.splice(this.eventData.hashtags.indexOf(item), 1);
      this.eventData.hashtags = [...this.eventData.hashtags];
    },
    SaveEvent() {
      this.loading = true;
      this.eventData.createdBy = {
        name: this.userDetails.name,
        id: this.userDetails.id,
      };
      this.eventData.createdOn = new Date();
      this.eventData.lastUpdatedOn = '';
      this.eventData.lastUpdatedBy = {
        name: '',
        id: '',
      };
      CustomEventServices.addCustomEvent(this.eventData.id, this.eventData)
        .then((res) => {
          if (res.success == true) {
            this.loading = false;
            this.dialog = false;
            this.$emit('showSuccess', res.msg);
          }
        })
        .catch((e) => {
          this.loading = false;
          console.log(e);
          this.$emit('message', e.msg);
        });
    },
  },
  // }
};
</script>
