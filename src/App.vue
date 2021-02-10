<template>
  <div class="container">
    <div class="row justify-content-center">
      <add-appointment @add="addItem" />
      <search-appointments
        @searchRecords="searchAppointment"
        @requestKey="changeKey"
        @requestDir="changeDir"
        :myKey="filterKey"
        :myDir="filterDir"
      />
      <appointment-list
        :appointments="filteredApt"
        @remove="removeItem"
        @edit="editItem"
      />
    </div>
  </div>
</template>

<script>
// import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import axios from "axios";
import AppointmentList from "./components/AppointmentList.vue";
import AddAppointment from "./components/AddAppointment.vue";
import SearchAppointments from "./components/SearchAppointment";

import _ from "lodash";
export default {
  name: "App",
  components: {
    // FontAwesomeIcon,
    AppointmentList,
    AddAppointment,
    SearchAppointments,
  },
  computed: {
    searchedApt: function() {
      return this.appointments.filter((item) => {
        return (
          item.petName.toLowerCase().match(this.searchTerm.toLowerCase()) ||
          item.petOwner.toLowerCase().match(this.searchTerm.toLowerCase()) ||
          item.aptNotes.toLowerCase().match(this.searchTerm.toLowerCase())
        );
      });
    },
    filteredApt: function() {
      return this.searchedApt;
    },
  },
  watch: {
    filteredApt: function() {
      this.searchedApt.sort(this.compared);
    },
  },
  data: function() {
    return {
      title: "Appointment List",
      appointments: [],
      aptIndex: 0,
      searchTerm: "",
      filterKey: "petOwner",
      filterDir: "desc",
    };
  },
  mounted: function() {
    axios.get("./data/appointments.json").then(
      (response) =>
        (this.appointments = response.data.map((item) => {
          item.aptId = this.aptIndex;
          this.aptIndex++;
          return item;
        }))
    );
  },
  methods: {
    compared: function(a, b) {
      //console.log(a[this.filterDir])
      if (this.filterDir == "asc") {
        if (a[this.filterKey] < b[this.filterKey]) {
          return -1;
        }
        if (a[this.filterKey] > b[this.filterKey]) {
          return 1;
        }
      } else {
        if (a[this.filterKey] < b[this.filterKey]) {
          return 1;
        }
        if (a[this.filterKey] > b[this.filterKey]) {
          return -1;
        }
      }

      return 0;
    },
    changeKey: function(val) {
      this.filterKey = val;
      this.appointments.sort(this.compared);
    },
    changeDir: function(val) {
      this.filterDir = val;
      this.appointments.sort(this.compared);
    },
    removeItem: function(apt) {
      this.appointments = _.without(this.appointments, apt);
    },
    editItem: function(id, field, text) {
      const aptIndex = _.findIndex(this.appointments, { aptId: id });
      this.appointments[aptIndex][field] = text;
    },
    addItem: function(apt) {
      apt.aptId = this.aptIndex;
      this.aptIndex++;
      this.appointments.push(apt);
    },
    searchAppointment: function(term) {
      this.searchTerm = term;
    },
  },
};
</script>

<style></style>
