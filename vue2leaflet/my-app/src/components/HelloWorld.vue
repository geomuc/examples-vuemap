<template>
  <v-app id="inspire">
    <v-navigation-drawer
      v-model="drawer"
      app
    >
      <v-list dense>
        <v-list-item >
          <v-list-item-action>
            <v-icon>mdi-home</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>Home</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item >
          <v-list-item-action>
            <v-icon>mdi-contact-mail</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>Contact</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar
      app
      color="indigo"
      dark
    >
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title>Application</v-toolbar-title>
    </v-app-bar>

    <v-content>
      <v-container
        class="fill-height"
        fluid
      >
        <v-row no-gutters>
          <v-col
            cols="12"
            sm="6"
          >
            <v-card
              class="pa-2"
              outlined
              tile
            >
            <v-select
              v-model= "auswahl"
              :items="items"
              label="Select Address"
              outlined
              v-on:change="setMarker()"
            ></v-select>
            <v-text-field
              label="type label"
              v-model="labeltext"
              v-on:input="textChanged()"
            ></v-text-field>
            <p>Long {{longout}} :: Lat {{latout}}</p>
            <v-text-field
              label="type angle"
              v-model="labelangle"
              v-on:input="angleChanged()"
            ></v-text-field>
            </v-card>
          </v-col>
            <v-col
            cols="12"
            sm="6"
          >
            <v-card
              class="pa-2"
              outlined
              tile
              height="500px"
              width="100%"
            >
              <v-img
                height="100%"
                width="100%"
                id="mapimg"
              >
                <l-map id="karte" :zoom="zoom" :center="center" @update:center="centerUpdated">
                  <!--<l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>-->
                  <!--<l-wms-tile-layer
                    base-url="https://geoservices.bayern.de/wms/v2/ogc_dop80_oa.cgi?"
                    layers="by_dop80c"                         
                    name="by_dop80c"
                  />-->
                  <l-wms-tile-layer
                    base-url="http://localhost:9999/geoserver/nyc_roads/wms?"
                    layers="roads,buildings"                         
                    name="roads"
                  />
                  http://localhost:9999/geoserver/nyc_roads/wms?
                  <l-marker :lat-lng="marker" draggable=true></l-marker>
                  <l-marker :lat-lng="labelmarker" draggable=true>
                    <l-icon
                      :icon-anchor="staticAnchor"
                      class-name="textlabel">                      
                      <div class="labelclass" id="lc" ref="lcid">{{labeltext}}</div>
                    </l-icon>
                  </l-marker>
                </l-map>
              </v-img>
            </v-card>
          </v-col>
        </v-row>

      </v-container>

    </v-content>
    <v-footer
      color="indigo"
      app
    >
      <span class="white--text">&copy; 2019</span>
    </v-footer>
  </v-app>
</template>

<script>
/* eslint-disable */
import L from 'leaflet';
import { LMap, LTileLayer, LMarker, LIcon, LPolyline, LWMSTileLayer } from 'vue2-leaflet';
import { Icon } from 'leaflet'
import 'leaflet/dist/leaflet.css'

delete L.Icon.Default.prototype._getIconUrl;

L.Icon.Default.mergeOptions({
  iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
  iconUrl: require('leaflet/dist/images/marker-icon.png'),
  shadowUrl: require('leaflet/dist/images/marker-shadow.png')
});

  export default {
    props: {
      source: String,
    },
      components: { LMap, LTileLayer, LMarker, LIcon, LPolyline, "l-wms-tile-layer": LWMSTileLayer },
    data() {
      return {
        auswahl: "",
        longitude: 11.6,
        latitude: 48.13,
        longout: 11.6,
        latout: 48.13,
        items: [],
        addresses: [],
        zoom:16,
        drawer: null,
        center: L.latLng(48.13,11.6),
        url:'http://{s}.tile.osm.org/{z}/{x}/{y}.png',
        attribution:'&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
        marker: L.latLng(48.13,11.6),
        // text
        labeltext: "soso",
        labelmarker: L.latLng(48.13,11.6),
        staticAnchor: [100, 10],
        //icontext = '<div class="headline">Mein Text</div>',
        icontext: 'Mein Text',
        labelangle: 30,
        polyline: {
          latlngs: [[48.13,11.6], [48.13,11.6]],
          color: 'red'
        }
      }
    },
    created(){
      this.fetchData();
    },
    mounted(){
      var lengthrotate = 0.01;
      var difflat=Math.cos(30* Math.PI / 180);
      var difflong=Math.sin(30* Math.PI / 180);
      console.log("Mounted");
      console.log(difflat);
      //this.polyline.latlngs = [[48.00,11.00], [48.0+difflat,11.0+difflong]]
    },
    methods: {
      fetchData(){
        fetch('http://localhost:8080/address')
          .then(function(response) {
            return response.json();
          })
          .then(data => {
        this.addresses=data;
        console.log(data);
        this.getAdrString();
        })
        .catch(function(error) {
            console.error(error);
        });
      },
      getAdrString(){
        var adrString = [];
        this.addresses.forEach(function(element) {
          adrString.push(element.number+", "+element.street);
        });
        this.items=adrString;
      },
      setMarker(){  
        var res = this.auswahl.split(",")[0];
        var address = "";
        this.addresses.forEach(function(element) {
            if (element.number==res){
              address=element;
            };
        });
        var long = address.location.coordinates[0];
        var lat =  address.location.coordinates[1];
        console.log(long);
        console.log(lat);
        this.marker=L.latLng(lat,long);
        this.center=L.latLng(lat,long);
        this.zoom=17;
        this.labelmarker= L.latLng(lat,long)
      },
      textChanged(){
        console.log(this.labeltext);
      },
      centerUpdated (center) {
        this.center = center;
        this.longout=center.lng.toFixed(4);
        this.latout=center.lat.toFixed(4);

      },
      angleChanged(){
        console.log(this.labelangle);
        console.log(this.$refs.lcid.style);
        this.$refs.lcid.style.transform="rotate("+this.labelangle+"deg)";
      },
    }
  };
</script>

<style>

#lc {
  width: 200px;
  height: 20px;
  background-color: yellow;
  transform: rotate(20deg);
  text-align: center;
}
</style>