<template>
  <v-app>
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
            sm="6"
            md="9"
          >
            <v-card 
            elevation="5"
            class="pa-5"
            >
              <v-card-title>Octomize</v-card-title>
              
              <!-- Container for accordians -->
              <v-container>
                <v-expansion-panels>
                  <v-expansion-panel class="mb-5 p-3">
                    <v-expansion-panel-header>
                      <v-checkbox label="Benchmark"></v-checkbox>
                    </v-expansion-panel-header>
                    <v-expansion-panel-content>Fusce et placerat augue, tristique sagittis metus. In auctor metus in elit lobortis, vulputate ullamcorper quam lobortis. Aliquam malesuada, urna fermentum rutrum feugiat, neque est blandit urna, fringilla elementum erat libero et velit. Nam sagittis sollicitudin pulvinar.</v-expansion-panel-content>
                  </v-expansion-panel>

                  <v-expansion-panel class="m-5 p-3"> 
                    <v-expansion-panel-header>
                      <v-checkbox 
                        input-value="true"
                        value
                        label="Accelerate"></v-checkbox>
                    </v-expansion-panel-header>
                    <v-expansion-panel-content>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec vestibulum justo in risus vehicula posuere. Proin id laoreet lacus, at tempus leo. Pellentesque suscipit rutrum purus ut suscipit. </v-expansion-panel-content>
                  </v-expansion-panel>
                </v-expansion-panels>
              </v-container>

              <v-card 
                tile
                elevation="0"
              >
                <v-row>
                  <v-col>
                    <v-card-subtitle>Hardware targets</v-card-subtitle>
                  </v-col>
                  <v-col md="2">
                    <v-btn 
                      color="blue white--text"
                      @click="addTarget"  
                    >Add</v-btn>
                  </v-col>
                </v-row>
              </v-card>

              <!-- Table for options -->
              <v-simple-table>
                <template v-slot:default>
                  <thead>
                    <tr>
                      <th class="text-left blue--text">
                        Provider
                      </th>
                      <th class="text-left">
                        Instance
                      </th>
                      <th class="text-left">
                        VCPU
                      </th>
                      <th class="text-left">
                        Memory (GIB)
                      </th>
                      <th></th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr
                      v-for="(item, index) in targets"
                      :key="item"
                    >
                      <td>
                        <v-select
                          v-model="targets[index].provider"
                          :items="providers"
                          item-text="name"
                          item-value="value"
                          label="Select Provider"
                          solo
                        ></v-select>
                      </td>
                      <td>
                        <v-select
                          v-model="targets[index].instance"
                          :items="hardware.filter(hw => hw.provider === targets[index].provider)"
                          item-text="instance"
                          item-value="instance"
                          label="Select Provider"
                          solo
                        ></v-select>
                      </td>
                      <td>{{ getCPU(targets[index].instance, index) }}</td>
                      <td>{{ getMemory(targets[index].instance, index) }}</td>
                      <td>
                        <v-btn
                          v-if="targets[index].instance === ` `"
                          @click="removeTarget(index)" 
                          icon
                        >X</v-btn>
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
              <v-card-text class="text-right text-h5 green--text">{{targets.length}}</v-card-text>
              <v-row
                v-for="item in targets"
                :key="item.instance"
              >
                <v-col>
                  <v-card elevation="0">
                    <v-card-title class="text-subtitle-1 font-weight-bold">{{item.instance}}</v-card-title>
                    <v-card-subtitle>{{item.cpu}} cores</v-card-subtitle>
                  </v-card>
                </v-col>
                <v-col md="4"><v-card-text class="text-right text-h6 green--text">1</v-card-text></v-col>
              </v-row>
              <v-card-actions>
                <v-btn 
                  block
                  :disabled="targets.length === 0"
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
//import HelloWorld from './components/HelloWorld';

export default {
  name: 'App',

  components: {
    //HelloWorld,
  },

  data: () => ({
        hardware:[],
        providers: [{value:"AWS", name:"Amazon Web Services"}, {value:"GCP", name:"Google Cloud"}, {value:"Azure", name:"Azure"}],
        targets: [],
  }),
  
  methods: {
    async getData() {
      try {
        // Get data
        const response = await fetch("http://netheria.takehome.octoml.ai/hardware").then(response => response.json());
        this.hardware = response;
        console.log(response);
      } catch (error) {
        console.log(error);
      }
    },
    addTarget() {
      this.targets.push({"provider": "", "instance": "","cpu": 0,"memory": 0})
    },
    getCPU(selectedInstance, index) {
      const filtered = this.hardware.filter(hw => hw.instance === selectedInstance);
      if (filtered.length > 0){
        this.targets[index].cpu = filtered[0].cpu;
        return filtered[0].cpu;
      }
      else
        return 0;
    },
    getMemory(selectedInstance,index) {
      const filtered = this.hardware.filter(hw => hw.instance === selectedInstance);
      if (filtered.length > 0){
        this.targets[index].memory = filtered[0].memory;
        return filtered[0].memory;
      }
      else
        return 0;
    },
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

<style scoped>
.background {
  background-color: gray;
}
</style>