<template>

  <vue100vh :css="{height: '100rvh'}">
    <span v-if="loading">Loading...</span>
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
      <l-control position="topleft" v-if="menu_type"> 
        <template>
          <v-card >
            <v-navigation-drawer
              permanent
              expand-on-hover
            >
                  
              <v-list>
                <v-list-group
                  v-for="(item, idx1) in menus"
                  :key="idx1"
                  v-model="item.active"
                  :prepend-icon="item.icon"
                  no-action
                >
                  <template v-slot:activator>
                    <v-list-item-content>
                      <v-list-item-title v-text="item.title"></v-list-item-title>
                    </v-list-item-content>
                  </template>

                  <v-container :multiple="true" v-model="item.selected_option" :mandatory="false">
                    <v-list-item
                      v-for="(child, idx2) in item.items"
                      :key="idx2"
                    >
                      <v-list-item-content class="pl-3">                        
                        <v-checkbox
                          :label="child.title"
                          :value="child.value"
                          v-model="selectedOptions"
                        ></v-checkbox>
                      </v-list-item-content>
                    </v-list-item>
                  </v-container>

                </v-list-group>
              </v-list>
              <v-divider></v-divider>

              <v-list
                nav
                dense
              >
                <v-list-item link>
                  <v-list-item-icon>
                    <v-icon>mdi-share-variant</v-icon>
                  </v-list-item-icon>
                  <v-list-item-title>Compartir</v-list-item-title>
                </v-list-item>
                <v-list-item link>
                  <v-list-item-icon>
                    <v-icon>mdi-monitor-screenshot</v-icon>
                  </v-list-item-icon>
                  <v-list-item-title>Captura</v-list-item-title>
                </v-list-item>
                <v-list-item link>
                  <v-list-item-icon>
                    <v-icon>mdi-refresh-circle</v-icon>
                  </v-list-item-icon>
                  <v-list-item-title>Refrescar</v-list-item-title>
                </v-list-item>
                <v-list-item link>
                  <v-list-item-icon>
                    <v-icon>mdi-alert-circle</v-icon>
                  </v-list-item-icon>
                  <v-list-item-title>Reportar Errores</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-navigation-drawer>
          </v-card>
        </template>
      </l-control>

      <!-- GeoJson -->
      <!-- <l-geo-json
        :geojson="geojson"
        :optionsStyle="geoStyle"        
      /> -->

      <!-- GeoJson Markers -->


      <!-- Markers -->
      <div v-for="(feature, idx) in geojson.features" :key="idx" >
        <l-marker  :lat-lng="[feature.geometry.coordinates[1], feature.geometry.coordinates[0]]" >
          <l-icon
            :icon-size="dynamicSize"
            :icon-anchor="dynamicAnchor"
          >
            
            <v-icon :color="getFlightCat(feature.properties.fltcat)">
              mdi-circle
            </v-icon>
            <div v-if="currentZoom >= 6.5">
              <p class="marker-label">{{feature.properties.id}}</p>
              <div class="marker-temp-div" >
                <p class="marker-temp" :style="`background: ${ tempScale(feature.properties.temp)} !important;`">
                  {{feature.properties.temp}}
                  <img class="marker-temp-icon" src="https://img.icons8.com/android/15/000000/thermometer.png"/>
                </p>
              </div>
              <div class="marker-wind-div d-flex" :style="`background: ${ windScale(feature.properties.wspd)} !important;`">
                <div v-if="!feature.properties.wspd == 0">
                  <v-icon :color="getFlightCat(feature.properties.fltcat)" 
                    :style="`transform: rotate(${getWindDirRotation(feature.properties.wdir)}deg);`">
                    mdi-arrow-right-thick
                  </v-icon>
                </div>
                <div v-else>
                  <v-icon>
                    mdi-circle-off-outline
                  </v-icon>
                </div>
                <p class="marker-wind" >
                  {{feature.properties.wspd}}KT
                  <img class="marker-wind-icon" src="https://img.icons8.com/fluent-systems-filled/15/000000/wind.png"/>
                </p>
              </div>
            </div>
            
          </l-icon>
          </l-marker>
      </div>
      

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
import { LMap, LTileLayer, LCircle, LControl, LControlZoom, LIcon, LMarker } from "vue2-leaflet";
import { Icon } from 'leaflet';
// 100vh div import
import vue100vh from 'vue-100vh';
// Icons
import CameraIcon from 'mdi-vue/Camera.vue';

// html2canvas
import html2canvas from 'html2canvas';
import Canvas2Image from 'canvas2image';
// full screen
import LControlFullscreen from 'vue2-leaflet-fullscreen';

// GeoJson
import importedJson from "./geoJson.json";

// chroma color scales
import chroma from "chroma-js";

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
    LIcon,
    LMarker,
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
      // general config
      selectedOptions: [],
      zoom: 4,
      position: 'topright',
      center: [12.346246, -83.147781],
      selectedLayer: 0,
      menu_type: null,
      loading: false,
      staticAnchor: [16, 37],
      iconSize: 32,
      // geoJSON
      geojson: importedJson,
      geoStyle: {
        "color": "#ff7800",
        "weight": 10,
        "opacity": 0.25
      },
      // menus
      menus: [
        {
          icon: 'mdi-weather-partly-snowy-rainy',
          title: 'Metar',
          active: false,
          selected_option: 1,
          items: [
              {
                title: 'Temperatura',
                value: 1 
              },
              {
                title: 'Barbas',
                value: 6 
              },
              {
                title: 'WindSp',
                value: 2 
              },
              {
                title: 'Wind',
                value: 3 
              },
              {
                title: 'Tiempo Presente WX',
                value: 4 
              },
              {
                title: 'Symbol',
                value: 5 
              }
            ],
        },
        {
          icon: 'mdi-space-station',
          title: 'Satélite',
          active: false,
          selected_option: 1,
          items: [
              {
                title: 'IR',
                value: 1 
              },
              {
                title: 'Vis',
                value: 2 
              },
              {
                title: 'VW',
                value: 3 
              },
              {
                title: 'Otros',
                value: 4 
              }
            ],
        },
        {
          icon: 'mdi-radar',
          title: 'Radar',
          active: false,
          selected_option: 1,
          items: [
              {
                title: 'Radar',
                value: 1 
              }
            ],
        },
        {
          icon: 'mdi-weather-lightning',
          title: 'Rayos',
          active: false,
          selected_option: 1,
          items: [
              {
                title: 'Rayos',
                value: 1 
              }
            ],
        },
        {
          icon: 'mdi-radio-tower',
          title: 'Estaciones',
          active: false,
          selected_option: 1,
          items: [
              {
                title: 'Temperatura',
                value: 1 
              },
              {
                title: 'WindSp',
                value: 2 
              },
              {
                title: 'Wind',
                value: 3 
              },
              {
                title: 'Tiempo Presente WX',
                value: 4 
              },
              {
                title: 'Symbol',
                value: 5 
              }
            ],
        }, 
      ],
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
      showMap: true,
      // color scales
      windScale: null,
      tempScale: null
    };
  },
  methods: {
    getFlightCat(cat){
      switch (cat) {
        case "LIFR":
          return "magenta";
        case "IFR":
          return "red";
        case "MVFR":
          return "blue";
        case "VFR":
          return "green";
        default:
          //Declaraciones ejecutadas cuando ninguno de los valores coincide con el valor de la expresión
          return "black";
      }
    },
    getWindDirRotation(p_dir){
      if (p_dir>=  0 && p_dir< 13) return "90";
      if (p_dir>= 13 && p_dir< 58) return "135";
      if (p_dir>= 58 && p_dir<103) return "180";
      if (p_dir>=103 && p_dir<148) return "225";
      if (p_dir>=148 && p_dir<193) return "270";
      if (p_dir>=193 && p_dir<238) return "315"; 
      if (p_dir>=238 && p_dir<283) return "0";
      if (p_dir>=283 && p_dir<328) return "45";
      if (p_dir>=328)              return "90";
    },
    zoomUpdate(zoom) {
      this.currentZoom = zoom;
      console.log(this.currentZoom);
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
    let windColorArr = [
      '#FFFFFF','#CDFFFE','#99FFCC','#99FF98','#9AFF67',
      '#9AFE01','#CCFF00','#FEFF00','#FE9900','#FF6602',
      '#FE3403','#FE0302'
    ]
    let tempColorArr = [
      '#000000','#303030','#312F9B','#080282','#1B00FC','#2F67F0',
      '#99CDFB','#4FC1BE','#048004','#9DCF00','#FED100',
      '#FE6602','#F50304'
    ]
    this.windScale = chroma.scale(windColorArr).domain([0,60], 12, 'quantiles');
    this.tempScale = chroma.scale(tempColorArr).domain([0,40], 13, 'quantiles');
    console.log(this.$route.query);
    // --- Map Configurations ---
    if( this.$route.query.zoom && this.$route.query.lat && this.$route.query.lon ) {
      this.zoom = parseInt(this.$route.query.zoom);
      this.center[0] = this.$route.query.lat;
      this.center[1] = this.$route.query.lon;
    }
    // --- END Map Configurations ---

    // --- Menu type to load ---
    if( this.$route.query.menu ) {
      this.menu_type = this.$route.query.menu;
    }
    // --- END - Menu type to load ---
    
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
  mounted(){

  },
  computed: {
    dynamicSize () {
      return [this.iconSize, this.iconSize * 1.15];
    },
    dynamicAnchor () {
      return [this.iconSize / 2, this.iconSize * 1.15];
    }
  },
  async created() {
      console.log(this.geojson);
      this.loading = true;
      // const response = await fetch("https://aviationweather.gov/cgi-bin/json/MetarJSON.php?zoom=8&filter=prior&density=0&taf=false&bbox=-104.94,1.79,-60.99,28.15/");
      // const data = await response.json();
      // this.geojson = data;
      this.loading = false;
  },
  watch: {
    menus: function(val){
      console.log(val);
    },
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
.marker-label {
    font-size: 11px;
    border-radius: 5px;
    border-style: solid;
    border-color: #9e9e9e;
    border-width: thin;
    background-color: #ffffffb8;
    width: 60px;
    padding-left: 10px;
    padding-right: 10px;
    position: relative;
    right: 15px;
    color: #5b5b5b;
}

.marker-temp {
    border-radius: 50%;
    width: 25px;
    height: 25px;
    padding: 0px;
    background: #ffffff8c;
    border: 1px solid #9e9e9e;
    color: #242424;
    text-align: center;
    font-size: 16px;
    position: relative;
    bottom: 65px;
    left: 40px;
}

.marker-temp-icon {
  position: relative;
  bottom: 10px;
  left: 12px;
}

.marker-wind-div {
    background: #ffffff8c;
    border-radius: 5px;
    border: 1px solid #9e9e9e;
    bottom: 145px;
    left: -10px;
    color: #5b5b5b;
    position: relative;
    height: 25px;
    width: 200%;
} 

.marker-wind {
    text-align: center;
    font-size: 15px;  
}

.marker-wind-icon {
  position: relative;
  bottom: 5px;
  left: 20px;
}

.marker-wind-dir {
  position: relative;
  bottom: 65px;
  left: 40px;
}

html, body {margin: 0; height: 100%; overflow: hidden}
</style>
