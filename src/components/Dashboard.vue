<template>
  <v-app id="inspire">
    <v-app-bar app color="white" flat>
      <v-container class="py-0 fill-height">
        <div id="app">
          <img height="50px" src="../assets/logo_obotrons.png" />
        </div>
        <v-spacer></v-spacer>
        <v-responsive max-width="260" class="mr-2">
          <v-autocomplete
            v-model="valuesDevice"
            :items="itemsDevice"
            flat
            hide-details
            rounded
            solo-inverted
            label="Device Type"
            @change="(event) => getProjectByKey(this.values)"
          ></v-autocomplete>
        </v-responsive>
        <v-responsive max-width="260">
          <v-autocomplete
            v-model="valuesProject"
            :items="itemsProject"
            flat
            hide-details
            rounded
            solo-inverted
            label="Select Project"
            multiple
            @change="(event) => getProjectByKey(this.valuesProject)"
          ></v-autocomplete>
        </v-responsive>
        <v-btn
          class="ma-2"
          outlined
          color="indigo"
          style="align-self: flex-start"
          @click="reloadPage()"
        >
          {{ timerCount }}
          <v-icon>mdi-cached</v-icon>
        </v-btn>
        <v-col class="text-right">
          <span class="mr-2" style="color: red">{{ date }}</span>
        </v-col>
      </v-container>
    </v-app-bar>
    <v-main class="grey lighten-3" style="padding: 10px 0px 0px">
      <div class="mx-8">
        <v-row>
          <v-col cols="3">
            <v-sheet rounded="lg">
              <v-list color="transparent">
                <v-subheader>Main IAQ Report</v-subheader>
                <v-list-item>
                  <v-list-item-title
                    class="text-left"
                    style="padding: 0px; font-size: medium"
                    >MAC</v-list-item-title
                  >
                  <v-autocomplete
                    style="padding: 0px; font-size: 15px"
                    v-model="valuesMacIAQ"
                    :items="itemsMacIAQ"
                    hide-details
                    label="Master mac"
                    @change="(event) => getProjectByMaster(this.valuesMacIAQ)"
                  ></v-autocomplete>
                </v-list-item>
                <v-list-item
                  v-for="([title, value], item) in itemsIAQ"
                  :key="item"
                  text
                >
                  <v-list-item-title class="text-left" style="padding: 0 0px">
                    {{ title }}
                  </v-list-item-title>
                  <v-list-item-title class="text-right" style="padding: 0 0px">
                    {{ value }}
                  </v-list-item-title>
                </v-list-item>
                <v-divider class="my-2"></v-divider>
                <v-list-item link color="grey lighten-4">
                  <v-list-item-content>
                    <v-list-item-title @click="getProjectByMaster()">
                      Refresh
                      <v-icon>mdi-cached</v-icon>
                    </v-list-item-title>
                  </v-list-item-content>
                </v-list-item>
              </v-list>
            </v-sheet>
          </v-col>
          <v-col>
            <v-sheet min-height="70vh" rounded="lg">
              <v-col cols="12" style="padding-top: 0px !important">
                <v-card-title>
                  Indoor Air Quality
                  <v-spacer></v-spacer>
                  <v-dialog v-model="dialog" max-width="500px">
                    <template v-slot:activator="{ on, attrs }">
                      <v-btn
                        color="primary"
                        dark
                        class="mb-2"
                        v-bind="attrs"
                        v-on="on"
                        style="margin-left: 10px"
                      >
                        Calibate all
                      </v-btn>
                    </template>
                    <v-card>
                      <v-card-title>
                        <span class="text-h5">{{ formTitle }}</span>
                      </v-card-title>

                      <v-card-text>
                        Please select project which you want to calibate all iaq
                        value.
                        <v-container>
                          <v-row>
                            <v-col cols="12">
                              <v-autocomplete
                                v-model="valuesCalibateProject"
                                :items="itemsCalibateProject"
                                outlined
                                dense
                                chips
                                small-chips
                                label="Select Project"
                                multiple
                              ></v-autocomplete>
                            </v-col>
                          </v-row>
                        </v-container>
                      </v-card-text>
                      <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn
                          color="red darken-1"
                          text
                          @click="dialog = false"
                        >
                          Cancel
                        </v-btn>
                        <v-btn
                          color="green darken-1"
                          text
                          @click="setAutoCalibate()"
                        >
                          OK
                        </v-btn>
                      </v-card-actions>
                    </v-card>
                  </v-dialog>
                </v-card-title>
                <v-card-title style="padding: 0px 16px 16px">
                  DateTime : {{ dateTime }}
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
                  :headers="headers"
                  :items="data"
                  item-key="name"
                  class="elevation-1"
                  :search="search"
                  @page-count="pageCount = $event"
                  hide-default-footer
                  :page.sync="page"
                  :items-per-page="itemsPerPage"
                >
                  <template v-slot:[`item.actions`]="{ item }">
                    <v-tooltip top>
                      <template v-slot:activator="{ on, attrs }">
                        <v-icon
                          small
                          class="mr-2"
                          v-bind="attrs"
                          v-on="on"
                          @click="editItem(item)"
                          >mdi-pencil</v-icon
                        >
                      </template>
                      <span>Setting</span>
                    </v-tooltip>
                    <v-tooltip top>
                      <template v-slot:activator="{ on, attrs }">
                        <v-icon
                          small
                          class="mr-2"
                          v-bind="attrs"
                          v-on="on"
                          @click="reportDataItem(item)"
                        >
                          mdi-file-document
                        </v-icon>
                      </template>
                      <span>Report</span>
                    </v-tooltip>
                    <v-tooltip top>
                      <template v-slot:activator="{ on, attrs }">
                        <v-icon
                          small
                          class="mr-2"
                          v-bind="attrs"
                          v-on="on"
                          @click="CalibateIAQ(item)"
                        >
                          mdi-tooltip-edit
                        </v-icon>
                      </template>
                      <span>Calibate</span>
                    </v-tooltip>
                  </template>
                  <template v-slot:[`item.iaq_MAC`]="{ item }">
                    <v-tooltip bottom>
                      <template v-slot:activator="{ on, attrs }">
                        <span v-bind="attrs" v-on="on">{{ item.iaq_MAC }}</span>
                      </template>
                      <span>{{ item.projectName }}</span>
                    </v-tooltip>
                  </template>
                  <template v-slot:[`item.iaq_RH`]="{ item }">
                    <span style="color: grey">{{ item.iaq_RH }}</span>
                    <v-spacer></v-spacer>
                    <span> ({{ item.compare_iaq_RH }}) </span>
                  </template>
                  <template v-slot:[`item.iaq_TEMP`]="{ item }">
                    <span style="color: grey">{{ item.iaq_TEMP }}</span>
                    <v-spacer></v-spacer>
                    <span> ({{ item.compare_iaq_TEMP }}) </span>
                  </template>
                  <template v-slot:[`item.iaq_CO2`]="{ item }">
                    <v-chip
                      :color="setColorIAQ(item.iaq_CO2, 'iaq_CO2')"
                      dark
                      style="margin: 4px"
                    >
                      <span style="color: black"> {{ item.iaq_CO2 }}</span>
                    </v-chip>
                    <v-spacer></v-spacer>
                    <span>({{ item.compare_iaq_CO2 }})</span>
                  </template>
                  <template v-slot:[`item.iaq_TVOC`]="{ item }">
                    <v-chip
                      :color="setColorIAQ(item.iaq_TVOC, 'iaq_TVOC')"
                      dark
                      style="margin: 4px"
                    >
                      <span style="color: black">{{ item.iaq_TVOC }}</span>
                    </v-chip>
                    <v-spacer></v-spacer>
                    <span> ({{ item.compare_iaq_TVOC }}) </span>
                  </template>
                  <template v-slot:[`item.iaq_PM10`]="{ item }">
                    <v-chip
                      :color="setColorIAQ(item.iaq_PM10, 'iaq_PM10')"
                      dark
                      style="margin: 4px"
                    >
                      <span style="color: black">{{ item.iaq_PM10 }}</span>
                    </v-chip>
                    <v-spacer></v-spacer>
                    <span> ({{ item.compare_iaq_PM10 }}) </span>
                  </template>
                  <template v-slot:[`item.iaq_PM25`]="{ item }">
                    <v-chip
                      :color="setColorIAQ(item.iaq_PM25, 'iaq_PM25')"
                      dark
                      style="margin: 4px"
                    >
                      <span style="color: black">{{ item.iaq_PM25 }}</span>
                    </v-chip>
                    <v-spacer></v-spacer>
                    <span> ({{ item.compare_iaq_PM25 }}) </span>
                  </template>
                </v-data-table>
                <div class="text-center pt-2">
                  <v-pagination
                    v-model="page"
                    :length="pageCount"
                  ></v-pagination>
                </div>
              </v-col>
            </v-sheet>
          </v-col>
        </v-row>
      </div>
    </v-main>
  </v-app>
</template>

<script>
import axios from "axios";
import moment from "moment";
export default {
  data: () => ({
    formTitle: "Do you want to calibate All devices?",
    dialog: false,
    itemsProject: [],
    itemsCalibateProject: [],
    valuesCalibateProject: [],
    valuesProject: [],
    projectKey: [],
    masterdata: [],
    itemsMacIAQ: [],
    valuesMacIAQ: "",
    data: [],
    compareIAQ: [],
    itemData: {},
    timerCount: 90,
    page: 1,
    pageCount: 0,
    itemsPerPage: 10,
    search: "",
    date: "",
    dateTime: "",
    itemsDevice: ["IAQ"],
    valuesDevice: "IAQ",
    itemsIAQ: [
      ["MAC", "null"],
      ["Humidity(%)", "null"],
      ["Temperature(°C)", "null"],
      ["CO2(ppm)", "null"],
      ["TVOC(ppb)", "null"],
      ["PM10(ug/m3)", "null"],
      ["PM2.5(ug/m3)", "null"],
      ["O2", "null"],
    ],
    headers: [
      {
        text: "MAC",
        align: "center",
        sortable: false,
        value: "iaq_MAC",
      },
      { text: "Humidity(%)", value: "iaq_RH", align: "center" },
      { text: "Temperature(°C)", value: "iaq_TEMP", align: "center" },
      { text: "CO2(ppm)", value: "iaq_CO2", align: "center" },
      { text: "TVOC(ppb)", value: "iaq_TVOC", align: "center" },
      { text: "PM10(ug/m3)", value: "iaq_PM10", align: "center" },
      { text: "PM2.5(ug/m3)", value: "iaq_PM25", align: "center" },
      { text: "Actions", value: "actions", align: "center" },
    ],
  }),
  created() {
    this.readMasterMac();
  },
  methods: {
    readMasterMac() {
      axios.get("http://192.168.0.100/readmac_master.php").then((response) => {
        if (response.status == 200) {
          this.valuesMacIAQ = response.data.macID
          this.getIAQ()
        }
      });
    },
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
            var itemProjectData = [];
            var valueProjectData = [];
            var itemProject = [];
            for (var i = 0; i < response.data.project.length; i++) {
              itemProjectData.push(response.data.project[i].name); // set Project Name List
              if (response.data.project[i].name != "MAIN") {
                valueProjectData.push(response.data.project[i].name);
                itemProject.push(response.data.project[i].name);
              }
              this.projectKey.push({
                name: response.data.project[i].name,
                value: response.data.project[i].data,
              });

              for (var j = 0; j < response.data.project[i].data.length; j++) {
                dataItem.push(response.data.project[i].data[j]);
                if (response.data.project[i].name == "MAIN") {
                  this.itemsMacIAQ.push(
                    response.data.project[i].data[j].iaq_MAC
                  );
                }
              }
              this.data = dataItem;

              this.itemsProject = itemProjectData;
              this.itemsCalibateProject = itemProject;
              this.valuesCalibateProject = valueProjectData;

              if (this.valuesProject.length == 0) {
                this.valuesProject = this.itemsProject;
              }

              var dataLastest = Object.assign({}, dataItem[dataItem.length]);
              this.dateTime = this.formatTime(dataLastest.iaq_datetime);
            }

            if (this.valuesProject.length != 0) {
              this.getProjectByKey();
            } else {
              this.cal();
            }

            if(this.valuesMacIAQ != "") {
              this.getProjectByMaster()
            }
          }
        });
    },
    getProjectByKey() {
      var dataIAQ = [];
      // this.valuesCalibateProject = this.valuesProject;
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
      this.cal();
    },
    getProjectByMaster() {
      const result = this.data.filter((pj) => {
        return pj.iaq_MAC == this.valuesMacIAQ;
      });

      this.masterdata = result[0];
      this.itemsIAQ = [
        ["Humidity(%)", result[0].iaq_RH],
        ["Temperature(°C)", result[0].iaq_TEMP],
        ["CO2(ppm)", result[0].iaq_CO2],
        ["TVOC(ppb)", result[0].iaq_TVOC],
        ["PM10(ug/m3)", result[0].iaq_PM10],
        ["PM2.5(ug/m3)", result[0].iaq_PM25],
        ["O2", result[0].iaq_O2],
      ];
      this.cal();
      this.sendIAQmaster();
    },
    sendIAQmaster() {
      this.saveFile();
      this.timerCount = 90;
      this.play;
      axios
        .post(
          `http://192.168.0.100/mastercalib.php?master_mac=${this.valuesMacIAQ}`
        )
        .then();
      this.cal();
    },
    reloadPage() {
      this.getIAQ();
    },
    editItem(item) {
      this.itemData = Object.assign({}, item);
      window.open(`http://${this.itemData.iaq_IP}:7878`, "_blank");
    },
    CalibateIAQ(item) {
      this.itemData = Object.assign({}, item);
      window.open(
        `http://${this.itemData.iaq_IP}:7878/calibrate.html`,
        "_blank"
      );
    },
    reportDataItem() {
      alert("This feature will be coming up");
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
      return moment(time).format("DD MMM YYYY \t HH:mm");
    },
    setAutoCalibate() {
      for (var index in this.valuesCalibateProject) {
        axios
          .get(
            `http://192.168.0.100/calIAQ.php?project=${this.valuesCalibateProject[index]}`,
            {
              headers: {
                "Master-Project": "MAIN-MASTER",
              },
            }
          )
          .then((response) => {
            if (response.status == 200) {
              this.dialog = false;
            }
          });
      }
    },

    cal() {
      var itemData = [];
      for (var idnex in this.projectKey) {
        for (var entry in this.projectKey[idnex].value) {
          var temp =
            this.projectKey[idnex].value[entry].iaq_TEMP -
            this.masterdata.iaq_TEMP;
          var humid =
            this.projectKey[idnex].value[entry].iaq_RH - this.masterdata.iaq_RH;
          var co2 =
            this.projectKey[idnex].value[entry].iaq_CO2 -
            this.masterdata.iaq_CO2;
          var tvoc =
            this.projectKey[idnex].value[entry].iaq_TVOC -
            this.masterdata.iaq_TVOC;
          var pm10 =
            this.projectKey[idnex].value[entry].iaq_PM10 -
            this.masterdata.iaq_PM10;
          var pm25 =
            this.projectKey[idnex].value[entry].iaq_PM25 -
            this.masterdata.iaq_PM25;
          itemData.push({
            projectName: this.projectKey[idnex].name,
            iaq_MAC: this.projectKey[idnex].value[entry].iaq_MAC,
            iaq_TEMP: this.projectKey[idnex].value[entry].iaq_TEMP,
            iaq_IP: this.projectKey[idnex].value[entry].iaq_IP,
            iaq_RH: this.projectKey[idnex].value[entry].iaq_RH,
            iaq_CO2: this.projectKey[idnex].value[entry].iaq_CO2,
            iaq_TVOC: this.projectKey[idnex].value[entry].iaq_TVOC,
            iaq_PM10: this.projectKey[idnex].value[entry].iaq_PM10,
            iaq_PM25: this.projectKey[idnex].value[entry].iaq_PM25,
            compare_iaq_TEMP: temp.toFixed(2),
            compare_iaq_RH: humid.toFixed(2),
            compare_iaq_CO2: co2,
            compare_iaq_TVOC: tvoc,
            compare_iaq_PM10: pm10,
            compare_iaq_PM25: pm25,
          });
        }
      }

      this.data = itemData;
    },
    saveFile: function () {
      var currentMac = { mac: this.valuesMacIAQ };
      let data = JSON.stringify(currentMac);
      window.localStorage.setItem("mac.json", data);
      // console.log(JSON.parse(window.localStorage.getItem("mac.json")));
    },
  },
  mounted() {
    setInterval(() => {
      this.timerCount--;
      if (this.timerCount == 0) {
        this.getIAQ();
        this.getProjectByMaster();
        this.timerCount = 90;
      }
    }, 1000);
    setInterval(() => {
      this.date = moment(Date()).format("DD MMMM  YYYY | HH:mm:ss");
    }, 1000);
  },
};
</script>