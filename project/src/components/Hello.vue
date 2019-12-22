<template>
  <div class="hello">

    <b-form-group
      label="Select Filter"
      label-cols-sm="2"
      label-align-sm="right"
      label-size="sm"
      class="mb-0 pb-2 my-1">
      <b-form-checkbox-group v-model="filterOn" class="mt-1"  id="filter">
        <b-form-checkbox value="id">Drone</b-form-checkbox>
        <b-form-checkbox value="name">Name</b-form-checkbox>
        <b-form-checkbox value="battery">Battery</b-form-checkbox> <br>
        <b-form-checkbox value="max_speed">Max Speed</b-form-checkbox>
        <b-form-checkbox value="average_speed">Average Speed</b-form-checkbox>
        <b-form-checkbox value="status">Status</b-form-checkbox>
      </b-form-checkbox-group>
    </b-form-group>
    
    <b-form-group
      label="Filter"
      label-cols-sm="2"
      label-align-sm="right"
      label-size="sm"
      label-for="filterInput"
      class="mb-0 pb-4 my-1">
      <b-input-group size="sm" id="filter">
        <b-form-input
          v-model="filter"
          type="search"
          id="filterInput"
          placeholder="Type to Search"></b-form-input>
        <b-input-group-append>
          <b-button :disabled="!filter" @click="filter = ''">Clear</b-button>
        </b-input-group-append>
      </b-input-group>
    </b-form-group>
    
    <b-table
      show-empty
      stacked="md"
      :items="items"
      :fields="fields"
      :current-page="currentPage"
      :per-page="perPage"
      :filter="filter"
      :filterIncludedFields="filterOn"
      :sort-by.sync="sortBy"
      :sort-desc.sync="sortDesc"
      :sort-direction="sortDirection"
      @filtered="onFiltered"
      id="my-table"
      responsive
    >
    
    <template v-slot:head(id)="data"><b class="text-info strong">{{ data.label.toUpperCase() }}</b></template>
    <template v-slot:head(battery)="data"><b class="text-info strong">{{ data.label.toUpperCase() }}</b></template>
    <template v-slot:head(max_speed)="data"><b class="text-info strong">{{ data.label.toUpperCase() }}</b></template>
    <template v-slot:head(average_speed)="data"><b class="text-info strong">{{ data.label.toUpperCase() }}</b></template>
    <template v-slot:head(status)="data"><b class="text-info strong">{{ data.label.toUpperCase() }}</b></template>
    <template v-slot:head(nameaddressimage)="data"><b class="text-info strong text-justify">{{ data.label.toUpperCase() }}</b></template>

    <template v-slot:cell(id)="d"><b class="strong h3"> {{d.value}} </b></template>

    <template v-slot:cell(battery)="d">
      <b-progress :value="d.value" :max="max" v-b-tooltip.hover :title="d.value" striped :animated="animate"></b-progress>
    </template>

    <template v-slot:cell(max_speed)="d"><b class="h3 strong">{{d.value}} </b> <small>m/h</small></template>
    <template v-slot:cell(average_speed)="d"><b class="h3 strong">{{d.value}} </b><small>m/h</small></template>

    <template v-slot:cell(status)="d"> 
      <template v-if="d.value === 'delayed'">
        <div  class="h4 bg-warning text-white text-uppercase " id="status"> {{ d.value }} </div>
      </template>
      <template v-else-if="d.value === 'failed'">
        <div  class="h4 bg-danger text-white text-uppercase align-middle" id="status"> {{ d.value }} </div>
      </template>
      <template v-else-if="d.value === 'success'">
        <div  class="h4 bg-success text-white text-uppercase " id="status"> {{  d.value }} </div>
      </template>
      <template v-else>
        <div  class="h4 bg-primary text-white text-uppercase " id="status"> {{  d.value }} </div>
      </template>
    </template>

    <template v-slot:cell(nameaddressimage)="d">
      <b-media no-body>
        <n-media-aside >
          <b-img-lazy v-bind="mainProps" :src="d.item.image" rounded="circle" alt="Circle image"></b-img-lazy>
        </n-media-aside>
        
        <b-media-body class="ml-3 text-justify" id="customer">
          <h4 >{{ d.item.name }}</h4>
          <h6>{{ d.item.address}}</h6>
        </b-media-body>
      </b-media>
    </template>

    </b-table>

     <div id="pagination">
      <p class="mt-3 text-center">Página {{ currentPage }}</p>
      <b-pagination v-model="currentPage" :total-rows="rows"
          :per-page="perPage"
          align="center"
          aria-controls="my-table"></b-pagination>
    </div>
  </div> 
</template>

<script>
import axios from 'axios';

export default {
  name: 'hello',
  data () {
    return {
      fields: [
        {key: 'id', label: 'Drone', sortable: true, sortDirection: 'desc'},
        {key: 'nameaddressimage', label: 'Customers'},
        {key: 'battery', label: "battery", sortable: true, sortDirection: 'desc'},
        {key: 'max_speed', label: 'Max Speed', sortable: true, sortDirection: 'desc'},
        {key: 'average_speed', label: 'Average Speed', sortable: true, sortDirection: 'desc'},
        {key: 'status', sortable: true, sortDirection: 'desc'}
      ],
      items: [],
      errors: [],
      mainProps: {
        blank: true,
        blankColor: "#bbb",
        idth: 50,
        height: 50
      },
      currentPage: 1,
      perPage: 10,
      max: 100,
      animate: true,
      sortBy: '',
      sortDesc: false,
      sortDirection: 'asc',
      filter: null,
      filterOn: [],
    }
  },

  created() {
    axios.get('http://localhost:8004/drones').then(response => {
      this.items = response.data
    }).catch(e => {
      this.errors.push(e)
    })
  },
  computed: {
    rows() {
       return this.items.length
    },
  },
  methods: {
    info(item, index, button) {
      this.infoModal.title = `Row index: ${index}`
      this.infoModal.content = JSON.stringify(item, null, 2)
      this.$root.$emit('bv::show::modal', this.infoModal.id, button)
    },
    resetInfoModal() {
      this.infoModal.title = ''
      this.infoModal.content = ''
    },
    onFiltered(filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length
      this.currentPage = 1
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#filter{
  width: 400px;
}

#status{
  border-radius: 25px;
  width: auto;  
  margin-top: .5rem;
}

#customer{
  width: 120px;
}

</style>
