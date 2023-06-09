<template>
  <v-container>
    <navbar />
    <v-divider></v-divider>
    <h1>Weekly Calendar</h1>
    <v-spacer></v-spacer>
    <div>
      <v-sheet tile height="54" class="d-flex">
        <v-btn icon class="ma-2" @click="$refs.calendar.prev()">
          <v-icon>mdi-chevron-left</v-icon>
        </v-btn>
        <v-select
          v-model="type"
          :items="types"
          dense
          outlined
          hide-details
          class="ma-2"
          label="type"
        ></v-select>
        <v-select
          :items="this.$store.state.doctors"
          v-model="doctorObj"
          item-text="firstName"
          item-value="doctorId"
          return-object
          dense
          outlined
          hide-details
          class="ma-2"
          label="doctors"
          @change="chosenDoctor"
        ></v-select>
        <v-select
          v-model="weekday"
          :items="weekdays"
          dense
          outlined
          hide-details
          label="weekdays"
          class="ma-2"
        ></v-select>
        <v-spacer></v-spacer>
        <v-btn icon class="ma-2" @click="$refs.calendar.next()">
          <v-icon>mdi-chevron-right</v-icon>
        </v-btn>
      </v-sheet>
      <v-sheet height="600">
        <v-calendar
          ref="calendar"
          v-model="value"
          :weekdays="weekday"
          :type="type"
          :events="filteredAppointments"
          :now="now"
        ></v-calendar>
      </v-sheet>
    </div>
  </v-container>
</template>
<script>
import Navbar from "../components/Navbar.vue";
import AppointmentService from "../services/AppointmentService";

export default {
  name: "calendar",
  components: {
    Navbar,
  },
  data() {
    return {
      type: "month",
      types: ["month", "week", "day"],
      mode: "stack",
      weekday: [0, 1, 2, 3, 4, 5, 6],
      weekdays: [
        { text: "Sun - Sat", value: [0, 1, 2, 3, 4, 5, 6] },
        { text: "Mon - Sun", value: [1, 2, 3, 4, 5, 6, 0] },
        { text: "Mon - Fri", value: [1, 2, 3, 4, 5] },
      ],
      value: "",
      events: [],
      colors: ["blue"],
      names: ["Appointment", "Holiday", "PTO"],
      appointments: [],
      now: "2023-04-20",
      selectedDoctorId: null, // id of doctor,
      doctors: [],
      doctorObj: {},
    };
  },
  methods: {
    chosenDoctor() {
      this.selectedDoctorId = this.doctorObj.doctorId;
    },
    getAppointments() {
      AppointmentService.getAppointments().then((response) => {
        this.$store.commit("SET_APPOINTMENTS", response.data);
        this.appointments = this.$store.state.appointments;
        this.getEvents();
      });
    },
    getEvents() {
      for (let i = 0; i < this.appointments.length; i++) {
        let temp = this.appointments[i];
        let time = this.appointments[i].appointmentTime;
        let endTime;
        if (time.slice(3, 5) === "30") {
          endTime = time.slice(0, 2);
          parseInt(endTime);
          endTime++;
          endTime = endTime.toString() + ":00:00";
        } else {
          endTime = time.slice(0, 2) + ":30:00";
        }
        let event = {
          docId: this.appointments[i].doctorId,
          name: "Appointment",
          start: temp.appointmentDate + "T" + temp.appointmentTime,
          end: (temp.appointmentDate += "T" + endTime),
          color: "blue",
          timed: false,
        };
        this.events.push(event);
      }
    },
  },
  computed: {
    // This should filter doctor apponintments by id - attached to :events for calendar
    filteredAppointments() {
      return this.events.filter((appt) => appt.docId === this.selectedDoctorId);
    },
  },
  created() {
    this.getAppointments();
    this.doctors = this.$store.state.doctors;
  },
};
</script>
