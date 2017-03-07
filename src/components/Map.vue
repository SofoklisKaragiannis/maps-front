/* vim: set softtabstop=2 shiftwidth=2 expandtab : */
<template>
  <div class="app-panel">
    <div class="settings-panel">

      <br>
      <h1>Bundle list:</h1>
      <!--Display bundle list -->
      <div class="list-group" style="width: 80%; margin: 0 auto;">
        <a href="javascript:void(0)" v-for="bundle in getAvailableBundles"
              v-on:click="displayBundle(bundle)">
          <div id="bundleList"><h3>{{ bundle.id }} - {{ bundle.name }} </h3></div>

        </a>
      </div>

      <!--Display selected bundle -->
      <div v-if="selectedBundle != undefined">
        <div id="bundleData">
          <div style="width: 50%; margin: 0 auto;">
            <img v-bind:src="selectedBundle.image" style="width: 200px;"/>
          </div>
          <h4>Id : {{ selectedBundle.id }}<br>
          Name : {{ selectedBundle.name }}<br>
          Info : {{ selectedBundle.info }}<br>
          Paths:</h4>
          <div v-for="path in selectedBundle.paths" style="padding-left: 5px;">
            <div style="width: 50%; margin: 0 auto;">
              <img v-bind:src="path.image" style="width: 200px;"/>
            </div>
            <h5>Id : {{ path.id }}<br>
            Name : {{ path.name }}<br>
            Info : {{ path.info }}<br>
            Length: {{ path.length }}<br>
            Places:</h5>
            <div v-for="place in path.places" style="padding-left: 10px;">
              <div style="width: 50%; margin: 0 auto;">
                <img v-bind:src="place.image" style="width: 200px;"/>
              </div>
              <h6>Name : {{ place.name }}<br>
              Info : {{ place.info }}<br>
              Radius: {{ place.radius }}<br>
              Position: lat {{ place.position.lat }} , lng {{ place.position.lng }} </h6>
            </div>
          </div>
        </div>
      </div>
    </div>

    <gmap-map
      :center="center"
      :zoom="zoom"
      :map-type-id="mapType"
      :options="{scrollwheel: scrollwheel}"
      class="map-panel"

    >

      <div v-for="polyline in availablePolylines">
        <gmap-polyline v-if="true" :path="polyline" :editable="false" :draggable="false"
                       :options="{geodesic:true, strokeColor:'#FF0000'}"
                       @path_changed="updatePolylinePath($event)">
        </gmap-polyline>
      </div>


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


  #bundleList {
    background: url(../assets/arrow1.png) no-repeat;
    background-size: 400px 52px;
    background-repeat: no-repeat;
    width: 400px;
    height: 52px;
    padding-top: auto;
    padding-bottom: auto;
    color: burlywood;
    text-align: left;
    vertical-align: middle;
    display: table;
    padding-left: 5px;
  }
  #bundleData {
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

  gmap-map {
    width:100%;
    height: 600px;
    display: block;
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
        center: { lat: 0, lng: 0 },
        zoom: 6,
        mapType: 'satellite',
        scrollwheel: true,
        availableBundles: undefined,
        availablePolylines: [],
        selectedBundle: undefined
      };
    },

    methods: {
      displayBundle(bundle) {
        // set selected bundle
        this.selectedBundle = bundle;
      },

      setMapData() {
        if (this.availableBundles !== undefined) {
          // extreme lat and lng values
          let latMax = -90;
          let latMin = 90;
          let lngMax = -180;
          let lngMin = 180;

          Object.entries(this.availableBundles).forEach(
            (bundle) => {
              Object.entries(bundle[1].paths).forEach(
                (path) => {
                  // array to store current polyline
                  let currentPolyline = [];
                  Object.entries(path[1].polyline).forEach(
                    (polylin) => {

                      let current = {lat: polylin[1].lat, lng: polylin[1].lng};
                      currentPolyline.push(current);

                      // set max lat value
                      if (latMax < polylin[1].lat) {
                        latMax = polylin[1].lat;
                      }

                      // set min lat value
                      if (latMin > polylin[1].lat) {
                        latMin = polylin[1].lat;
                      }

                      // set max lng value
                      if (lngMax < polylin[1].lng) {
                        lngMax = polylin[1].lng;
                      }
                      // set min lng value
                      if (lngMin > polylin[1].lng) {
                        lngMin = polylin[1].lng;
                      }
                    });
                  // add all polylines
                  this.availablePolylines.push(currentPolyline);
                });
            });

          //calculate and set map center coordinates
          let latMid = (latMax + latMin) / 2;
          let lngMid = (lngMax + lngMin) / 2;
          this.center = {lat: latMid, lng: lngMid};
        }
      }

    },
//    watch: {
//      '$route'(to, from) {
//        // Call resizePreserveCenter() on all maps
//        Vue.$gmapDefaultResizeBus.$emit('resize');
//      }
//    },

    computed: {
      getAvailableBundles() {
        // request all bundels if not set
        if (this.availableBundles === undefined) {
          Vue.use(vueResource);
          return Vue.http.get('http://127.0.0.1:8080/rest/v1/retrieveAll')
              .then((response) => {

                this.availableBundles = response.body;
                // initialise map data
                this.setMapData();
          })
          .catch((errorResponse) => {
            return errorResponse;
          })
        }
        return this.availableBundles;
      },
    },
  };
  /* eslint-enable */
</script>
