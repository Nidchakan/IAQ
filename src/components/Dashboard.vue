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
      <v-col>
        <v-data-table
          dense
          :headers="headers"
          :items="master"
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
            >
              <v-icon>mdi-pencil</v-icon>
            </v-btn>
          </template>
          <template v-slot:[`item.checkData`]="{ item }">
            <v-row justify="center">
              <v-dialog v-model="dialogmaster" max-width="290">
                <template v-slot:activator="{ on, attrs }">
                  <v-btn
                    class="ma-2"
                    outlined
                    small
                    color="indigo"
                    v-on="on"
                    v-bind="attrs"
                    @click="checkDataItem(item)"
                  >
                    <v-icon>mdi-playlist-check</v-icon>
                  </v-btn>
                </template>
                <v-card>
                  <v-card-title class="text-h5">
                    Use Google's location service?
                  </v-card-title>
                </v-card>
              </v-dialog></v-row
            >
          </template>
          <template v-slot:[`item.iaq_datetime`]="{ item }">
            <span>{{ formatTime(item.iaq_datetime) }}</span>
          </template>
          <template v-slot:[`item.iaq_CO2`]="{ item }">
            <v-chip :color="setColorIAQ(item.iaq_CO2, 'iaq_CO2')" dark>
              <span style="color: black"> {{ item.iaq_CO2 }}</span>
            </v-chip>
          </template>
          <template v-slot:[`item.iaq_TVOC`]="{ item }">
            <v-chip :color="setColorIAQ(item.iaq_TVOC, 'iaq_TVOC')" dark>
              <span style="color: black">{{ item.iaq_TVOC }}</span>
            </v-chip>
          </template>
          <template v-slot:[`item.iaq_PM10`]="{ item }">
            <v-chip :color="setColorIAQ(item.iaq_PM10, 'iaq_PM10')" dark>
              <span style="color: black">{{ item.iaq_PM10 }}</span>
            </v-chip>
          </template>
          <template v-slot:[`item.iaq_PM25`]="{ item }">
            <v-chip :color="setColorIAQ(item.iaq_PM25, 'iaq_PM25')" dark>
              <span style="color: black">{{ item.iaq_PM25 }}</span>
            </v-chip>
          </template>
        </v-data-table>
      </v-col>
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
          <template v-slot:[`item.iaq_datetime`]="{ item }">
            <span>{{ formatTime(item.iaq_datetime) }}</span>
          </template>
          <template v-slot:[`item.iaq_CO2`]="{ item }">
            <v-chip :color="setColorIAQ(item.iaq_CO2, 'iaq_CO2')" dark>
              <span style="color: black"> {{ item.iaq_CO2 }}</span>
            </v-chip>
          </template>
          <template v-slot:[`item.iaq_TVOC`]="{ item }">
            <v-chip :color="setColorIAQ(item.iaq_TVOC, 'iaq_TVOC')" dark>
              <span style="color: black">{{ item.iaq_TVOC }}</span>
            </v-chip>
          </template>
          <template v-slot:[`item.iaq_PM10`]="{ item }">
            <v-chip :color="setColorIAQ(item.iaq_PM10, 'iaq_PM10')" dark>
              <span style="color: black">{{ item.iaq_PM10 }}</span>
            </v-chip>
          </template>
          <template v-slot:[`item.iaq_PM25`]="{ item }">
            <v-chip :color="setColorIAQ(item.iaq_PM25, 'iaq_PM25')" dark>
              <span style="color: black">{{ item.iaq_PM25 }}</span>
            </v-chip>
          </template>
          <template v-slot:[`item.calibate`]="{ item }">
            <v-simple-checkbox
              v-model="item.calibate"
              disabled
            ></v-simple-checkbox>
          </template>
          <template v-slot:[`item.actions`]="{ item }">
            <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil</v-icon>
            <v-icon small class="mr-2" @click="deleteItem(item)"> mdi-file-document </v-icon>
            <v-icon small @click="deleteItem(item)"> mdi-playlist-check </v-icon>
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
    itemsDevice: ["IAQ"],
    valuesDevice: "IAQ",
    valueDate: null,
    data: [],
    master: [],
    projectKey: [],
    itemData: {},
    search: "",
    timerEnabled: true,
    timerCount: 90,
    timer: {},
    dialogl: false,
    dialogmaster: false,
    headers: [
      {
        text: "MAC",
        align: "center",
        sortable: false,
        value: "iaq_MAC",
      },
      // { text: "Project", value: "name", align: "start"},
      // { text: "Date", value: "iaq_datetime", align: "start"},
      // { text: "MAC", value: "iaq_MAC", align: "center" },
      { text: "Humidity (%)", value: "iaq_RH", align: "center" },
      { text: "Temperature (°C)", value: "iaq_TEMP", align: "center" },
      { text: "CO2 (ppm)", value: "iaq_CO2", align: "center" },
      { text: "TVOC (ppb)", value: "iaq_TVOC", align: "center" },
      { text: "PM10 (ug/m3)", value: "iaq_PM10", align: "center" },
      { text: "PM2.5 (ug/m3)", value: "iaq_PM25", align: "center" },
      { text: "Actions", value: "actions", align: "center" },
      { text: "calibate", value: "calibate", align: "center" }
    ],
  }),
  created() {
    this.getIAQ();
    this.getIAQmaster();
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
    },
    getIAQmaster() {
      this.timerCount = 90;
      this.play;
      axios
        .get("http://192.168.0.100/resultIAQ.php?mac_id=84F3EB868924", {
          headers: {
            "Project-Building": "OBT-TEST",
          },
        })
        .then((response) => {
          console.log(response.data);
          if (response.status == 200) {
            // this.master = response.data;
            this.master.push(response.data);
            console.log(this.master);
          }
        });
    },
    reloadPage() {
      this.getIAQ();
    },
    editItem(item) {
      this.itemData = Object.assign({}, item);
      window.open(`http://${this.itemData.iaq_IP}:7878`,'_blank');
    },
    checkDataItem() {
      
    },
    setColorIAQ(value, type) {
      switch (type) {
        case "iaq_CO2": {
          if (value <= 800) return "#4CAF50";
          //green
          else if (value >= 801 && value <= 1000) return "#e0f50b";
          //green yellow
          else if (value >= 1001 && value <= 1500) return "#FFC000";
          //orange
          else if (value >= 1501 && value <= 2000) return "#ff565f";
          // red
          else return "#e24bfc"; // purple
        }
        case "iaq_TVOC": {
          if (value <= 65) return "#4CAF50";
          //green
          else if (value >= 66 && value <= 220) return "#e0f50b";
          //green yellow
          else if (value >= 221 && value <= 660) return "#FFC000";
          //orange
          else if (value >= 661 && value <= 2200) return "#ff565f";
          // red
          else return "#e24bfc"; // purple
        }
        case "iaq_PM10": {
          if (value <= 50) return "#4CAF50";
          //green
          else if (value >= 51 && value <= 100) return "#e0f50b";
          //green yellow
          else if (value >= 101 && value <= 250) return "#FFC000";
          //orange
          else if (value >= 251 && value <= 350) return "#ff565f";
          // red
          else return "#e24bfc"; // purple
        }
        case "iaq_PM25": {
          if (value <= 15) return "#4CAF50";
          //green
          else if (value >= 16 && value <= 25) return "#e0f50b";
          //green yellow
          else if (value >= 26 && value <= 65) return "#FFC000";
          //orange
          else if (value >= 66 && value <= 150) return "#ff565f";
          // red
          else return "#e24bfc"; // purple
        }
      }
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