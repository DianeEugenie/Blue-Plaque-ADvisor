<template lang="html">
<div id="add-form">
  <form class="addForm" v-on:submit.prevent="addLocation">
    <input type="text" v-model="title" placeholder="Plaque Title" required>
    <input type="text" v-model="subjects" placeholder="Name of Subject">
    <input type="text" v-model="colour_name" placeholder="Colour"required>
    <textarea rows=10 v-model="inscription" placeholder="Inscription" required></textarea>
    <input type="text" v-model="address" placeholder="Address">
    <input class="submit-button" type="submit" value="Add Location">
  </form>
</div>
</template>

<script>

import { eventBus } from '@/main.js';
import PlaqueService from '@/services/PlaqueService.js';

export default {
  name: 'user-add',
  data(){
    return {
      title: '',
      subjects: '',
      colour_name: '',
      inscription: '',
      address: ''
    }
  },
  props: ['userLocation'],
  methods: {
    addLocation(location){
      this.userLocation["title"]=this.title
      this.userLocation["subjects"]=this.subjects
      this.userLocation["colour_name"]=this.colour_name
      this.userLocation["inscription"]=this.inscription
      this.userLocation["address"]=this.address
      this.userLocation["people"]=[]
      PlaqueService.postLocations(this.userLocation)
      .then((res) => {
        this.title = this.subjects = this.colour_name = this.inscription = this.address = ""
        this.userLocation.userAdded = false
        eventBus.$emit('location-newmarker', res)
        eventBus.$emit('option-selected', 'details')
      })
    }
    }

}
</script>

<style lang="css" scoped>

#add-form{
  margin: auto 0;
  display: flex;
  flex-direction: column;
}

input,
textarea {
  border: 1px dotted grey;
  width: 14vw;
  margin: 5px 0;
}

input[type=submit]{
  background-color: #FFDB3F;
}
</style>
