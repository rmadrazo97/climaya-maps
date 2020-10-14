<template>

  <vue100vh :css="{height: '100rvh'}">

    <l-map
      id="map"
      v-if="showMap"
      :zoom="zoom"
      :center="center"
      :options="mapOptions"
      style="height: 100vh; width: 100vw"
      @update:center="centerUpdate"
      @update:zoom="zoomUpdate"
    >
      <l-control 
        v-if="mapconfig.showSS"
        class="custom-control">
        <p @click="takeScreenshot">
          <camera-icon></camera-icon>
        </p>
      </l-control>

      <l-control>
        <v-menu
          open-on-hover
          top
          :offset-y="true"
          style="z-index: 999; padding-y: 10px;"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              color="primary"
              dark
              v-bind="attrs"
              v-on="on"
            >
              <v-icon dark>
                mdi-layers-triple
              </v-icon>
            </v-btn>
          </template>

          <v-list class="d-flex flex-wrap">
            <v-list-item>
              <v-radio-group >
                <v-radio-group v-model="selectedLayer">
                  <v-radio
                    v-for="(tile, index) in tileProviders"
                    :key="index"
                    :label="tile.name"
                    :value="index"                    
                  >
                  <template v-slot:label>
                    <div>
                      <p> <img class="" :src="tile.iconUrl"/> {{tile.name}}</p>
                      
                    </div>
                  </template>
                  </v-radio>
                  
                </v-radio-group>
              </v-radio-group>  
            </v-list-item>
          </v-list>
        </v-menu>
      </l-control>

      <l-tile-layer

        v-for="tileProvider in tileProviders"
        @update="console.log(e)"
        :key="tileProvider.name"
        :name="tileProvider.name"
        :visible="tileProvider.visible"
        :url="tileProvider.url"
        :attribution="tileProvider.attribution"
        layer-type="base"/>

      <l-tile-layer
        name="tileProvider.name"
        :visible="tileProviders[3].visible"
        url="https://services.arcgisonline.com/ArcGIS/rest/services/Reference/World_Boundaries_and_Places/MapServer/tile/{z}/{y}/{x}"
        attribution="Atribution"
        layer-type="base"
      />

      <!-- logo -->
      <l-control position="bottomleft">
        <img src="../../assets/logoCRRH.png" style="width: 130px;" alt="CRRH">
      </l-control>
      
      <!-- zoom control -->
      <l-control-zoom position="topright"  ></l-control-zoom>
      
      <!-- full screen -->
      <l-control-fullscreen
          position="topright"
          :options="fullscreenOptions"
      />

      <!-- left pane menu -->
      <!-- <l-control position="topleft">
        <div style="background-color: #fff; border: 2px solid rgba(0,0,0,0.2); border-radius: 4px; background-clip: padding-box;">
          <v-icon light style="font-size: 30px">
            mdi-chevron-right
          </v-icon>
        </div>
      </l-control> -->

      <!-- menu -->
      <l-control position="topleft">
        <template>
          <v-card>
            <v-navigation-drawer
              permanent
              expand-on-hover
            >
                <v-list>
                  <v-list-item>
                    <v-list-item-icon>
                      <v-icon>mdi-home</v-icon>
                    </v-list-item-icon>

                    <v-list-item-title>Home</v-list-item-title>
                  </v-list-item>

                  <v-list-group
                    :value="true"
                    prepend-icon="mdi-account-circle"
                  >
                    <template v-slot:activator>
                      <v-list-item-title>Users</v-list-item-title>
                    </template>

                    <v-list-group
                      :value="true"
                      no-action
                      sub-group
                    >
                      <template v-slot:activator>
                        <v-list-item-content>
                          <v-list-item-title>Admin</v-list-item-title>
                        </v-list-item-content>
                      </template>

                      <v-list-item
                        v-for="([title, icon], i) in admins"
                        :key="i"
                        link
                      >
                        <v-list-item-title v-text="title"></v-list-item-title>

                        <v-list-item-icon>
                          <v-icon v-text="icon"></v-icon>
                        </v-list-item-icon>
                      </v-list-item>
                    </v-list-group>

                    <v-list-group
                      no-action
                      sub-group
                    >
                      <template v-slot:activator>
                        <v-list-item-content>
                          <v-list-item-title>Actions</v-list-item-title>
                        </v-list-item-content>
                      </template>

                      <v-list-item
                        v-for="([title, icon], i) in cruds"
                        :key="i"
                        link
                      >
                        <v-list-item-title v-text="title"></v-list-item-title>

                        <v-list-item-icon>
                          <v-icon v-text="icon"></v-icon>
                        </v-list-item-icon>
                      </v-list-item>
                    </v-list-group>
                  </v-list-group>
                </v-list>
              <v-divider></v-divider>

              <v-list
                nav
                dense
              >
                <v-list-item link>
                  <v-list-item-icon>
                    <v-icon>mdi-folder</v-icon>
                  </v-list-item-icon>
                  <v-list-item-title>My Files</v-list-item-title>
                </v-list-item>
                <v-list-item link>
                  <v-list-item-icon>
                    <v-icon>mdi-account-multiple</v-icon>
                  </v-list-item-icon>
                  <v-list-item-title>Shared with me</v-list-item-title>
                </v-list-item>
                <v-list-item link>
                  <v-list-item-icon>
                    <v-icon>mdi-star</v-icon>
                  </v-list-item-icon>
                  <v-list-item-title>Starred</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-navigation-drawer>
          </v-card>
        </template>
      </l-control>

      <!-- CIRCLE -->
      <div v-if="mapconfig.showCircle">
        <l-circle
          :lat-lng="mapconfig.CiLatLon"
          :radius="mapconfig.CiRad"
          :color="mapconfig.CiCol"
          :visible="tileProvider.visible"
        />
      </div>

    </l-map>
  </vue100vh>
</template>

<script>
// leaflet imports
import { latLng } from "leaflet";
import { LMap, LTileLayer, LCircle, LControl, LControlZoom } from "vue2-leaflet";
import { Icon } from 'leaflet';
// 100vh div import
import vue100vh from 'vue-100vh'
// Icons
import CameraIcon from 'mdi-vue/Camera.vue';


// html2canvas
import html2canvas from 'html2canvas'
import Canvas2Image from 'canvas2image'
// full screen
import LControlFullscreen from 'vue2-leaflet-fullscreen';

delete Icon.Default.prototype._getIconUrl;
Icon.Default.mergeOptions({
  iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
  iconUrl: require('leaflet/dist/images/marker-icon.png'),
  shadowUrl: require('leaflet/dist/images/marker-shadow.png'),
});

export default {
  name: "Map",
  components: {
    // Leaflet components
    LMap,
    LTileLayer,
    LCircle,
    LControl,
    LControlZoom,
    // 
    vue100vh,
    // icons
    CameraIcon,
    // full screen
    LControlFullscreen
  },
  data() {
    return {
      mapconfig: {
        showCircle: false,
        CiLat: null,
        CiLon: null,
        CiRad: null,
        CiCol: null,
        CiLatLon: [null,null],
        showSS: false,
      },
      zoom: 4,
      position: 'topright',
      center: [12.346246, -83.147781],
      selectedLayer: 0,
      admins: [
        ['Management', 'mdi-account-multiple-outline'],
        ['Settings', 'mdi-cog-outline'],
      ],
      cruds: [
        ['Create', 'mdi-plus-outline'],
        ['Read', 'mdi-file-outline'],
        ['Update', 'mdi-update'],
        ['Delete', 'mdi-delete'],
      ],
      tileProviders: [
        {
          id: 1,
          name: 'Claro',
          visible: true,
          iconUrl: "https://img.icons8.com/windows/25/000000/map.png",
          attribution: '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
          url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
        },
        {
          id: 2,
          name: 'Oscuro',
          visible: false,
          iconUrl: "https://img.icons8.com/material-rounded/24/000000/map.png",
          attribution: '&copy; <a href="https://stadiamaps.com/">Stadia Maps</a>, &copy; <a href="https://openmaptiles.org/">OpenMapTiles</a> &copy; <a href="http://openstreetmap.org">OpenStreetMap</a> contributors',
          url: 'https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png',
        },
        {
          id: 3,
          name: 'Topográfico',
          visible: false,
          url: 'https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png',
          iconUrl: "https://img.icons8.com/material-two-tone/24/000000/information-pyramid.png",
          attribution: 'Map data: &copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, <a href="http://viewfinderpanoramas.org">SRTM</a> | Map style: &copy; <a href="https://opentopomap.org">OpenTopoMap</a> (<a href="https://creativecommons.org/licenses/by-sa/3.0/">CC-BY-SA</a>)',
        },
        {
          id: 4,
          name: 'Satelital',
          visible: false,
          iconUrl: "https://img.icons8.com/windows/32/000000/satellite.png",
          url: 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
          attribution: 'ArcGIS Online',
        },
        {
          id: 5,
          name: 'Terreno',
          visible: false,
          iconUrl: "https://img.icons8.com/ios-glyphs/25/000000/mountain.png",
          url: 'https://stamen-tiles-{s}.a.ssl.fastly.net/terrain/{z}/{x}/{y}{r}.png',
          attribution: 'Map tiles by <a href="http://stamen.com">Stamen Design</a>, <a href="http://creativecommons.org/licenses/by/3.0">CC BY 3.0</a> &mdash; Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
        },
      ],
      fullscreenOptions: {
        title: {
          'false': 'Switch to full-screen view',
          'true': 'Exit full-screen mode',
        }
      },
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      withPopup: latLng(47.41322, -1.219482),
      withTooltip: latLng(47.41422, -1.250482),
      currentZoom: 8,
      currentCenter: latLng(47.41322, -1.219482),
      showParagraph: false,
      mapOptions: {
        zoomSnap: 0.5,
        zoomControl: false
      },
      showMap: true
    };
  },
  methods: {
    zoomUpdate(zoom) {
      this.currentZoom = zoom;
    },
    centerUpdate(center) {
      this.currentCenter = center;
    },
    showLongText() {
      this.showParagraph = !this.showParagraph;
    },
    innerClick() {
      alert("Click!");
    },
    takeScreenshot() {
      console.log("ss");
      html2canvas(document.querySelector('#map'), {
            onrendered: function(canvas) {
                // document.body.appendChild(canvas);
              return Canvas2Image.saveAsPNG(canvas);
            }
        });
    }
  },
  beforeMount(){
    
    console.log(this.$route.query);
    // --- Map Configurations ---
    if( this.$route.query.zoom && this.$route.query.lat && this.$route.query.lon ) {
      this.zoom = parseInt(this.$route.query.zoom);
      this.center[0] = this.$route.query.lat;
      this.center[1] = this.$route.query.lon;
    }
    // --- END Map Configurations ---
    
    // --- Circle ---.
    if( this.$route.query.showCircle == 'true' && this.$route.query.CiLat && this.$route.query.CiLon ) {
      console.log("show circle");
      this.mapconfig.showCircle = this.$route.query.showCircle;
      this.mapconfig.CiLat = this.$route.query.CiLat;
      this.mapconfig.CiLon = this.$route.query.CiLon;
      this.mapconfig.CiLatLon[0] = this.$route.query.CiLat;
      this.mapconfig.CiLatLon[1] = this.$route.query.CiLon;

      if(this.$route.query.CiRad) {
        this.mapconfig.CiRad = this.$route.query.CiRad;
      } else {
        this.mapconfig.CiRad = 4000;
      }

      if(this.$route.query.CiCol) {
        this.mapconfig.CiCol = this.$route.query.CiCol;
      } else {
        this.mapconfig.CiCol = 'red';
      }

    }
    // --- Circle end ---
  },
  watch: {
    selectedLayer: function (val) {
      console.log(val);
      if(val == 0){
        this.tileProviders[val].visible = true;
        this.tileProviders[1].visible = false;
        this.tileProviders[2].visible = false;
        this.tileProviders[3].visible = false;
        this.tileProviders[4].visible = false;
      } else if (val == 1){
        this.tileProviders[val].visible = true;
        this.tileProviders[0].visible = false;
        this.tileProviders[2].visible = false;
        this.tileProviders[3].visible = false;
        this.tileProviders[4].visible = false;

      } else if (val == 2){
        this.tileProviders[val].visible = true;
        this.tileProviders[0].visible = false;
        this.tileProviders[1].visible = false;
        this.tileProviders[3].visible = false;
        this.tileProviders[4].visible = false;

      } else if (val == 3){
        this.tileProviders[val].visible = true;
        this.tileProviders[0].visible = false;
        this.tileProviders[1].visible = false;
        this.tileProviders[2].visible = false;
        this.tileProviders[4].visible = false;

      } else if (val == 4){
        this.tileProviders[val].visible = true;
        this.tileProviders[0].visible = false;
        this.tileProviders[1].visible = false;
        this.tileProviders[2].visible = false;
        this.tileProviders[3].visible = false;

      }
    }
  }
};
</script>

<style>
.custom-control {
  background: #fff;
  padding: 0 0.5em;
  border: 1.5px solid #aaa;
  border-radius: 10px;
}
.custom-control-watermark {
  font-size: 200%;
  font-weight: bolder;
  color: #aaa;
  text-shadow: #555;
}

html, body {margin: 0; height: 100%; overflow: hidden}
</style>
