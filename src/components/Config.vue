<template>
  <v-container>
    <v-row class="pa-0" align-item="center">
      <v-col cols="3" style="padding-bottom: 0px !important">
        <v-autocomplete
          v-model="valuesDevice"
          :items="itemsDevice"
          outlined
          dense
          chips
          small-chips
          label="Device Type"
          multiple
          @change="(event) => getProjectByKey(this.values)"
        ></v-autocomplete>
      </v-col>
      <v-col cols="7" style="padding-bottom: 0px !important">
        <v-autocomplete
          v-model="valuesProject"
          :items="itemsProject"
          outlined
          dense
          chips
          small-chips
          label="Select Project"
          multiple
          @change="(event) => getProjectByKey(this.valuesProject)"
        ></v-autocomplete>
      </v-col>
      <v-btn
        class="ma-4"
        outlined
        color="indigo"
        style="align-self: flex-start"
        @click="reloadPage"
      >
        {{ timerCount }}
        <v-icon>mdi-cached</v-icon>
      </v-btn>
    </v-row>

    <v-row class="text-center">
      <v-col cols="12" style="padding-top: 0px !important">
        <v-card-title>
          Indoor Air Quality
          <v-spacer></v-spacer>
          <v-text-field
            v-model="search"
            append-icon="mdi-magnify"
            label="Search"
            single-line
            hide-details
          ></v-text-field>
        </v-card-title>
        <v-data-table
          dense
          :headers="headers"
          :items="data"
          item-key="name"
          class="elevation-1"
          :search="search"
        >
          <template v-slot:[`item.edit`]="{ item }">
            <v-btn
              class="ma-2"
              outlined
              small
              color="indigo"
              @click="editItem(item)"
              target="_blank"
            >
              <v-icon>mdi-pencil</v-icon>
            </v-btn>
          </template>
           <template v-slot:[`item.report`]="{ item }">
            <v-btn
              class="ma-2"
              outlined
              small
              color="indigo"
              @click="reportItem(item)"
            >
              <v-icon>mdi-file-document</v-icon>
            </v-btn>
          </template>
        </v-data-table>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
import moment from "moment";
export default {
  data: () => ({
    itemsProject: [],
    valuesProject: [],
    value: null,
    itemsDevice:[],
    valuesDevice:["IAQ"],
    itemsDate: ["Day", "Month", "Year"],
    valuesDate: "Day",
    valueDate: null,
    data: [],
    projectKey: [],
    itemData: {},
    search: "",
    timerEnabled: true,
    timerCount: 90,
    timer: {},
    headers: [
      {
        text: "MAC",
        align: "start",
        sortable: false,
        value: "iaq_MAC",
      },
      // { text: "Project", value: "name", align: "start"},
      // { text: "Date", value: "iaq_datetime", align: "start"},
      { text: "Config", value: "edit", align: "center" },
      { text: "Report",value:"report",align:"center"}
    ],
  }),
  created() {
    this.getIAQ();
  },
  methods: {
    getIAQ() {
      this.timerCount = 90;
      this.play;
      axios
        .get("http://192.168.0.100/resultIAQ.php", {
          headers: {
            "Project-Building": "AllProject",
          },
        })
        .then((response) => {
          if (response.status == 200) {
            var dataItem = [];
            for (var i = 0; i < response.data.project.length; i++) {
              this.itemsProject.push(response.data.project[i].name); // set Project Name List
              this.projectKey.push({
                name: response.data.project[i].name,
                value: response.data.project[i].data,
              });
              for (var j = 0; j < response.data.project[i].data.length; j++) {
                dataItem.push(response.data.project[i].data[j]); 
              }
              this.data = dataItem;
            }
            console.log(this.data)
            if (this.valuesProject.length != 0) {
              this.getProjectByKey();
            }
          }
        });
    },
    getProjectByKey() {
      var dataIAQ = [];
      if (this.valuesProject.length == 0) {
        this.getIAQ();
      } else if (this.valuesProject.length == 1) {
        const result = this.projectKey.filter((pj) => {
          return pj.name == this.valuesProject;
        });
        dataIAQ = result[0].value;
      } else if (this.valuesProject.length > 1) {
        for (var i = 0; i < this.valuesProject.length; i++) {
          const result = this.projectKey.filter((pj) => {
            return pj.name == this.valuesProject[i];
          });
          for (var j = 0; j < result[0].value.length; j++) {
            dataIAQ.push(result[0].value[j]);
          }
        }
      }
      this.data = dataIAQ;
      console.log(this.data);
    },
    reloadPage() {
      this.getIAQ();
    },
    editItem(item) {
      this.itemData = Object.assign({}, item);
      window.open(`http://${this.itemData.iaq_IP}:7878`,'_blank');
    },
    formatTime(time) {
      return moment(time).format("HH:mm:ss");
    },
  },
  mounted() {
    setInterval(() => {
      this.timerCount--;
      if (this.timerCount == 0) {
        this.getIAQ();
        this.timerCount = 90;
      }
    }, 1000);
  },
};
</script>