<template>
  <v-dialog v-model="dialog" width="700">
    <template v-slot:activator="{ on }">
      <div
        v-on="on"
        style="cursor: pointer; border-left: 8px solid #4e5fbb"
        class="pa-3 ma-0 lightModeCard"
      >
        <p class="google-font mt-1 mb-0" style="font-size: 90%">
          {{ data.data.local_date | dateFilter }}
        </p>
        <p class="google-font ma-0 mt-0" style="font-size: 120%">
          {{ data.data.name | summary(20) }}
        </p>
        <p class="google-font mt-0 mb-0" style="font-size: 90%">
          {{ data.data.local_time }}
        </p>
        <p class="google-font mt-0 mb-0" style="font-size: 90%">
          {{ data.data.venue.name | summary(20) }}
        </p>
      </div>
    </template>
    <v-card color="" v-if="dialog">
      <v-card-title
        class="px-5 py-5 google-font"
        style="background-position: right bottom"
      >
        {{ data.data.name }}
      </v-card-title>

      <v-card-text class="pa-5">
        <p class="google-font mt-3 mb-0" style="font-size: 110%">
          <b>Venue: </b> {{ data.data.venue.name }}
        </p>
        <p class="google-font mt-1 mb-0" style="font-size: 110%">
          <b>Date: </b>{{ data.data.local_date | dateFilter }}
        </p>
        <p class="google-font mt-0 mb-0" style="font-size: 110%">
          <b>Time: </b>{{ data.data.local_time }}
        </p>
        <p class="google-font mt-3 mb-0" style="font-size: 110%">
          <b>Description: </b>
          <span
            v-html="$options.filters.summary(data.data.description, 100)"
          ></span>
        </p>

        <v-btn
          color="#1a73e8"
          v-if="data.data.link.length > 0"
          :href="data.data.link"
          target="_blank"
          class="ma-0 elevation-0 my-2 mr-3"
          dark
          style="text-transform: capitalize"
        >
          See More at Meetup
        </v-btn>
      </v-card-text>

      <v-divider></v-divider>

      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="primary" text @click="dialog = false"> Close </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  props: {
    data: {
      data: {},
    },
  },
  data() {
    return {
      dialog: false,
    };
  },
  methods: {
    getCharString(data) {
      var splitArr = data.split(' ');
      if (splitArr.length > 1) {
        return (
          splitArr[0].substring(0, 1) +
          '' +
          splitArr[1].substring(0, 1)
        ).toUpperCase();
      } else {
        return splitArr[0].substring(0, 1).toUpperCase();
      }
    },
  },
  filters: {
    summary: (val, num) => {
      if (val.length > num) {
        return val.substring(0, num) + '..';
      } else {
        return val;
      }
    },
    dateFilter: (value) => {
      const date = new Date(value);
      return date.toLocaleString(['en-US'], {
        month: 'short',
        day: '2-digit',
        year: 'numeric',
      });
    },
  },
};
</script>
