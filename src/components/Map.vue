/* vim: set softtabstop=2 shiftwidth=2 expandtab : */
<template>
  <div class="app-panel">
    <div class="settings-panel">
      <h3 v-if="title != undefined" class="fixSpaces">Select Place:</h3>
      <!--Display place list -->
      <div v-if="availablePlaces != undefined">
        <div class="list-group" style="width: 90%; margin: 0 auto;">
          <a href="javascript:void(0)" v-for="(place, index) in availablePlaces"
             v-on:click="displayPlace(place)">
              <h6 class="fixSpaces" style="text-align: left; color: black; text-decoration: none !important;">{{index}}. {{ place.name }} - {{ place.vicinity }} </h6>
          </a>
        </div>
      </div>

      <br>
      <!--Display selected place -->
      <div v-if="selectedPlace != undefined">
        <div id="placeData">
          <div style="width: 50%; margin: 0 auto;">
            <img v-bind:src="selectedPlace.icon" style="width: 200px;"/>
          </div>
          <h5 class="fixSpaces">Name : {{ selectedPlace.name }}<br>
            Vicinity : {{ selectedPlace.vicinity }}<br>
            Types:</h5>
          <div v-for="type in selectedPlace.types" style="padding-left: 5px;">
            <h6 class="fixSpaces"> - {{ type }}</h6>
          </div>
        </div>
      </div>
    </div>

    <!--Create map-->
    <gmap-map
      :center="center"
      :zoom="zoom"
      :map-type-id="mapType"
      :options="{scrollwheel: scrollwheel}"
      class="map-panel"
      @rightclick="mapRclicked"

    >
      <!--Create markers-->
      <gmap-marker
        v-if="m.enabled"
        :position="m.position"
        :opacity="m.opacity"
        :draggable="m.draggable"
        @click="m.clicked++"
        @rightclick="m.rightClicked++"
        @dragend="m.dragended++"

        @position_changed="updateChild(m, 'position', $event)"

        v-for="m in markers"
        :key="m.id"
      >
        <gmap-info-window
          :opened="m.ifw"
          :content="m.ifw2text"
        ></gmap-info-window>
      </gmap-marker>

    </gmap-map>
  </div>
</template>

<style>
  .app-panel {
    width: 100%;
    height: 100%;
    position: fixed;
    top: 0;
    left: 0;
    display: flex;
    flex-direction: row;
  }

  .map-panel {
    flex: 4 1 80%;
  }
  .settings-panel {
    overflow-y: scroll;
    flex: 1 0 500px;
  }

  gmap-map {
    width:100%;
    height: 600px;
    display: block;
  }

  .fixSpaces {
    margin-top: 5px;
    margin-bottom: 0px;
    padding-top: 0px;
    padding-bottom: 0px;
  }

  #placeData {
    width: 90%;
    margin: 0 auto;
    padding: 0px;
    vertical-align: middle;
    text-align: left;
    color: black;
    display: table;
    border: groove;
    background-color: burlywood;
  }
</style>

<script>
  /* eslint-disable */
  import Vue from 'vue';
  import vueResource from 'vue-resource';

  export default {
    data: function (){
      // initial data values
      return {
        lastId: 1,
        markers: [],
        markersEven: false,
        center: { lat: 59.332339, lng: 18.064479 },
        zoom: 12,
        mapType: 'satellite',
        scrollwheel: true,
        availablePlaces: undefined,
        title: undefined,
        selectedPlace: undefined
      };
    },

    methods: {
      // handles right click
      mapRclicked (mouseArgs) {
        //reset variables
        this.availablePlaces = undefined;
        this.selectedPlace = undefined;
        this.title = undefined;
        //create marker
        this.addMarker(mouseArgs);
        //request location
        this.getLocation(mouseArgs);
      },

      getLocation(mouseArgs) {
        // create request url string
        let requestString = process.env.apiUrl+'/rest/v1/getLocation?lat=' + mouseArgs.latLng.lat() +'&lng=' + mouseArgs.latLng.lng();
        if (this.availableBundles === undefined) {
          Vue.use(vueResource);
          // request places from running environment
          return Vue.http.get(requestString)
            .then((response) => {
              // initialise map data
            this.availablePlaces = response.body.results;
            if (this.availablePlaces != undefined && this.availablePlaces.length > 0) {
              this.title = 'Select Place:';
            }
          })
            //error handling
          .catch((errorResponse) => {
            return errorResponse;
          })
        }
      },

      addMarker(mouseArgs) {
        this.lastId++;
        // add marker data
        this.markers = [{
          id: this.lastId,
          position: { lat: mouseArgs.latLng.lat() , lng: mouseArgs.latLng.lng() },
          opacity: 1,
          draggable: false,
          enabled: true,
          clicked: 0,
          rightClicked: 0,
          dragended: 0,
          ifw: true,
          ifw2text: "Position: "+mouseArgs.latLng.lat()+","+mouseArgs.latLng.lng()
        }];
        return this.markers[this.markers.length - 1];
      },

      displayPlace(place) {
        this.selectedPlace = place;
      }
    },
  };
  /* eslint-enable */
</script>
