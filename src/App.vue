<template>
  <v-app>
    <!-- Navigation, just for looks -->
    <v-navigation-drawer
      app
      class="pt-4"
      color="grey lighten-3"
      mini-variant
      permanent
    >
      <v-avatar
        v-for="n in 6"
        :key="n"
        :color="`grey ${n === 1 ? 'darken' : 'lighten'}-1`"
        :size="n === 1 ? 36 : 20"
        class="d-block text-center mx-auto mb-9"
      ></v-avatar>
    </v-navigation-drawer>


    <v-main class="background ">

      <!-- Main card for user input -->
      <v-container class="justify-center ">
        <v-row></v-row>
        <v-row>
          <v-col
            cols="12"
            md="9"
          >
            <v-card 
            elevation="5"
            class="pa-5"
            >
              <v-card-title class="text-h4">Octomize</v-card-title>
              
              <!-- Container for accordians -->
              <v-row>
                <v-col>
                  <v-expansion-panels 
                    class="d-flex align-center"
                  >
                    <!-- Benchmark accordian -->
                    <v-expansion-panel 
                      class="mb-5 p-3 rounded-lg"
                      elevation="0"
                    >
                      <v-expansion-panel-header>
                        <v-checkbox label="Benchmark"></v-checkbox>
                      </v-expansion-panel-header>
                      <v-expansion-panel-content>Fusce et placerat augue, tristique sagittis metus. In auctor metus in elit lobortis, vulputate ullamcorper quam lobortis. Aliquam malesuada, urna fermentum rutrum feugiat, neque est blandit urna, fringilla elementum erat libero et velit. Nam sagittis sollicitudin pulvinar.</v-expansion-panel-content>
                    </v-expansion-panel>

                    <!-- Accelerate panel, default selected to match figma spec -->
                    <v-expansion-panel 
                      class="m-5 p-3 rounded-lg"
                      elevation="0"
                    > 
                      <v-expansion-panel-header>
                        <v-checkbox 
                          input-value="true"
                          value
                          label="Accelerate"></v-checkbox>
                      </v-expansion-panel-header>
                      <v-expansion-panel-content>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec vestibulum justo in risus vehicula posuere. Proin id laoreet lacus, at tempus leo. Pellentesque suscipit rutrum purus ut suscipit. </v-expansion-panel-content>
                    </v-expansion-panel>
                  </v-expansion-panels>
                </v-col>
              </v-row>
              <v-row class="mb-12"></v-row>
              <!-- new smaller card for the description of the table and the add button -->
              <v-row>
                <v-col>
                  <v-card 
                    tile
                    elevation="0"
                  >
                    <v-row>
                      <v-col col="10" md="10">
                        <v-card-subtitle>Hardware targets</v-card-subtitle>
                      </v-col>
                      <v-col class="d-flex justify-end"  >
                        <!-- Add button adds elements to target array, used
                            to add rows to table and total runs card -->
                        <v-btn 
                          color="blue white--text"
                          @click="addTarget"
                          elevation="0"
                        >Add</v-btn>
                      </v-col>
                    </v-row>
                  </v-card>
                </v-col>
              </v-row>
              <!-- Table for options -->
              <v-simple-table>
                <template v-slot:default>
                  <!-- Header row, includes an empty spot so the 
                       x button lines up more nicely -->
                  <thead>
                    <tr>
                      <th 
                        class="text-left blue--text" 
                        width="30%"
                      >
                        Provider
                      </th>
                      <th 
                        class="text-left"
                        width="30%"
                      >
                        Instance
                      </th>
                      <th 
                        class="text-left"
                        width="10%"
                      >
                        VCPU
                      </th>
                      <th 
                        class="text-left"
                        width="15%"
                      >
                        Memory (GIB)
                      </th>
                      <th width="5%"></th>
                    </tr>
                  </thead>

                  <!-- Table body -->
                  <tbody>
                    <!-- dynamically creates rows -->
                    <tr
                      v-for="(item, index) in targets"
                      :key="item"
                    >
                      <td >
                        <!-- Dropdown, contains hardcoded list of providers -->
                        <v-select
                          v-model="targets[index].provider"
                          :items="providers"
                          item-text="name"
                          item-value="value"
                          label="Select Provider"
                          solo
                          class="mt-5"
                        ></v-select>
                      </td>
                      <td>
                        <!-- Dropdown, contains list of instance options based on selected provider -->
                        <v-select
                          v-model="targets[index].instance"
                          :items="hardware.filter(hw => hw.provider === targets[index].provider)"
                          :disabled="targets[index].provider === ''"
                          item-text="instance"
                          item-value="instance"
                          label="Select Instance"
                          solo
                          class="mt-5"
                        ></v-select>
                      </td>
                      <!-- cpu/memory specs are updated based on selected instance -->
                      <td :class="targets[index].instance === '' ? 'disabled-text' : ''">
                        {{ getCPU(targets[index].instance, index) }}
                      </td>
                      <td :class="targets[index].instance === '' ? 'disabled-text' : ''">
                        {{ getMemory(targets[index].instance, index) }}</td>
                      <td class="ps-0 pe-0">
                        <!-- button to remove an item, only shows once selections have been
                             made in both dropdowns -->
                        <v-btn
                          v-if="targets[index].instance != ``"
                          @click="removeTarget(index)" 
                          icon
                        ><v-icon>mdi-close</v-icon></v-btn>
                      </td>
                    </tr>
                  </tbody>
                </template>
              </v-simple-table>
            </v-card>
          </v-col>

          <v-col >
            <!-- Container for run outputs -->
            <v-card class="justify-end m-5">
              <v-card-subtitle class="text-right">Total Runs</v-card-subtitle>
              <v-card-text 
                class="text-right text-h4 green--text"
              >{{runs ? targets.length : 0}}</v-card-text>
              
              <!-- Dynamically adds rows for run details, in the same
                   order they are displayed in the table -->
              <v-row
                v-for="item in targets"
                :key="item.instance"
              > 
                <v-col v-if="item.instance === ''"></v-col>
                <v-col v-if="item.instance !== ''">
                  <!-- Card displays instance and cpu details -->
                  <v-card elevation="0">
                    <v-card-title class="text-subtitle-1 font-weight-bold">{{item.instance}}</v-card-title>
                    <v-card-subtitle>{{item.cpu}} cores</v-card-subtitle>
                  </v-card>
                </v-col>
                <v-col 
                  md="4"
                  v-if="item.instance !== ''">
                  <v-card-text class="text-right text-h6 green--text">1</v-card-text>
                </v-col>
              </v-row>

              <!-- Octomize button is disabled when there are no hardware
                   targets specified, but is otherwise just for show -->
              <v-card-actions>
                <v-btn 
                  block
                  :disabled="!runs"
                  color="blue white--text"
                >Octomize</v-btn>
              </v-card-actions>
            </v-card>
          </v-col>
        </v-row>
      </v-container>

    </v-main>
  </v-app>
</template>

<script>

export default {
  name: 'App',

  data: () => ({
        //API response
        hardware:[],
        //hardcoded array of provider objects for dropdown
        providers: [{value:"AWS", name:"Amazon Web Services"}, {value:"GCP", name:"Google Cloud"}, {value:"Azure", name:"Azure"}],
        //array of hardware targets defined by the user
        targets: [],
        runs: false,
  }),
  
  methods: {
    /*
      Method: getData
       -Gets data from netheria API, will catch and log errors
    */
    async getData() {
      try {
        // Get data
        const response = await fetch("http://netheria.takehome.octoml.ai/hardware").then(response => response.json());
        this.hardware = response;
        //console.log(response);
      } catch (error) {
        console.log(error);
      }
    },
    /*
      Method: addTarget
       -Adds an empty object to the target array
     */
    addTarget() {
      this.targets.push({"provider": "", "instance": "","cpu": 0,"memory": 0})
    },
    /*
      Method: getCPU
       -Gets the value of the cpu element of the specified instance, and both returns it
        and uses it to update the cpu element at target[index]
      Parameters:
       -selectedInstance: the value of the instance property selected by the user
       -index: the index of the object in the target array to be modified
      
    */
    getCPU(selectedInstance, index) {
      const filtered = this.hardware.filter(hw => hw.instance === selectedInstance);
      if (filtered.length > 0){
        this.runs=true;
        this.targets[index].cpu = filtered[0].cpu;
        return filtered[0].cpu;
      }
      else
        return 0;
    },
    /*
      Method: getMemory
       -Gets the value of the memory element of the specified instance, and both returns it
        and uses it to update the cpu element at target[index]
      Parameters:
       -selectedInstance: the value of the instance property selected by the user
       -index: the index of the object in the target array to be modified
      
    */
    getMemory(selectedInstance, index) {
      const filtered = this.hardware.filter(hw => hw.instance === selectedInstance);
      if (filtered.length > 0){
        this.targets[index].memory = filtered[0].memory;
        return filtered[0].memory;
      }
      else
        return 0;
    },
    /*
      Method: removeTarget
       -removes an object from target[index]
      Parameters:
       -index: index of item to be removed
     */
    removeTarget(index) {
      this.targets.splice(index,1);
      this.selectedProvider.splice(index,1);
      this.selectedInstance.splice(index,1);
    }    
  },

  mounted() {
    this.$nextTick(function () {
      // Get data when page is rendered
      this.getData();
    });
  }
};
</script>
<style>
  .v-expansion-panel::before {
    box-shadow: none !important;
  }
  .v-expansion-panel{
    border: 2px solid rgba(0, 0, 0, 0.20) !important;
  }
  .theme--light.v-expansion-panels .v-expansion-panel:not(:first-child)::after {
    border: none !important;
  }
  .disabled-text{
    color: rgba(0, 0, 0, 0.50);
  }
  table {
  table-layout: fixed;
  width: 100%;
  }
  th,
  td {
    overflow: hidden;
  }
</style>
