<template>
  <div class="main">
    <div class="flex">
      <!-- Map Display here -->
      <div class="map-holder">
        <div id="map"></div>
      </div>

      <!-- Coordinates Display here -->
      <div class="dislpay-arena">
        <div class="coordinates-header">
          <h3>Current Coordinates</h3>
          <p>Latitude: {{ center[0] }}</p>
          <p>Longitude: {{ center[1] }}</p>
        </div>

        <div class="coordinates-header">
          <h3>Current Location</h3>

          <div class="form-group">
            <input
                type="text"
                class="location-control"
                :value="location"
                readonly
            />
            <button type="button" class="copy-btn" @click="copyLocation">
              Copy
            </button>
          </div>

          <button
              type="button"
              :disabled="loading"
              :class="{ disabled: loading}"
              class="location-btn"
              @click="getLocation"
          >
            Get Location
          </button>
        </div>
        <div class="image_container">
          <div class="info_title">Info</div>
          <div v-if="roadTitle" class="road_content">{{roadTitle}}</div>
          <div v-if="roadImg" class="road_image" :style="{ backgroundImage: 'url(' + roadImg + ')' }"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import mapboxgl from "mapbox-gl";
import axios from "axios";
import MapboxGeocoder from "@mapbox/mapbox-gl-geocoder";

import "@mapbox/mapbox-gl-geocoder/dist/mapbox-gl-geocoder.css";
export default {
  name: "MainPage",
  data() {
    return {
      loading: false,
      location: "",
      access_token: process.env.VUE_APP_MAP_ACCESS_TOKEN,
      center: [9.85891, 46.76176],
      map: {},
      trackLines: [],
      roadImg: '',
      roadTitle: ''

    }
  },
  mounted() {

     axios.get('https://gist.githubusercontent.com/dutsik/3d23f1c562691381eeda4a93a564f521/raw/7287fc013df54704bb457fcd44228be82f8a1727/track_minimal.json')
    .then(response => {
      let tracks = response.data
      for (let tracksKey in tracks) {
        let data = {
          type: 'Feature',
          properties: {
            color: '#730c61',
          },
          geometry: {
            type: 'LineString',
            coordinates: []
          }
        }
        data.properties.photos = tracks[tracksKey].photos
        data.properties.title = tracks[tracksKey].title
        let longitude = tracks[tracksKey].track.longitude.split(',');
        let latitude = tracks[tracksKey].track.latitude.split(',');
        for (const trackKey in latitude) {
          let lat = latitude[trackKey]
          let lon = longitude[trackKey]
          let arr = []
          arr.push(lon)
          arr.push(lat)
          data.geometry.coordinates.push(arr)
        }
        this.trackLines.push(data)
      }
    })
    .catch(error => {
      console.log(error)
    })

    this.createMap();
  },
  methods: {
    async createMap() {
      try {
        mapboxgl.accessToken = this.access_token;
        this.map = new mapboxgl.Map({
          container: "map",
          style: 'mapbox://styles/mapbox/streets-v8',
          center: this.center,
          zoom: 9,
        });
        let geocoder =  new MapboxGeocoder({
          accessToken: this.access_token,
          mapboxgl: mapboxgl,
          marker: false,
        });
        // let mapboxInspect = new MaxboxInspect({
        //   showMapPopup: true,
        //   showMapPopupOnHover: false,
        //   showInspectMapPopupOnHover: false
        // })
        // this.map.addControl(mapboxInspect)
        this.map.addControl(geocoder);
        geocoder.on("result", (e) => {
          const marker = new mapboxgl.Marker({
            draggable: true,
            color: "#1a000b",
          })
              .setLngLat(e.result.center)
              .addTo(this.map);
          this.center = e.result.center;
          marker.on("dragend", (e) => {
            this.center = Object.values(e.target.getLngLat());
          });
        });
        this.map.addControl(new mapboxgl.NavigationControl());
        this.map.on('load', () => {
          this.map.addSource('lines', {
            type: 'geojson',
            data: {
              type: 'FeatureCollection',
              features: this.trackLines
            }
          })
          this.map.addLayer({
            'id': 'lines',
            'type': 'line',
            'source': 'lines',
            'paint': {
              'line-width': 5,
              'line-color': ['get', 'color']
            }
          });

        })
        this.map.on('mouseenter', 'lines', () => {
          this.map.getCanvas().style.cursor = 'pointer';
        });
        this.map.on('mouseleave', 'lines', () => {
          this.map.getCanvas().style.cursor = '';
        });
        this.map.on('click', 'lines', (e) => {
          let photo = JSON.parse(e.features[0].properties.photos);
          this.roadTitle = e.features[0].properties.title;
          if(photo.smallThumb !== undefined) {
            this.roadImg = photo.smallThumb
          } else {
            this.roadImg = ''
          }
        })

      } catch (err) {
        console.log("map error", err);
      }
    },
    async getLocation() {
      try {
        this.loading = true;
        const response = await axios.get(
            `https://api.mapbox.com/geocoding/v5/mapbox.places/${this.center[0]},${this.center[1]}.json?access_token=${this.access_token}`
        );
        this.loading = false;
        this.location = response.data.features[0].place_name;
      } catch (err) {
        this.loading = false;
        console.log(err);
      }
    },
    copyLocation() {
      if (this.location) {
        navigator.clipboard.writeText(this.location);
        alert("Location Copied")
      }
      return;
    },
  },
}
</script>

<style scoped lang="scss">
.main {
  padding: 45px 50px;
}
.flex {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}
.map-holder {
  width: 65%;
  overflow: hidden;
}
#map {
  height: 70vh;
}
.dislpay-arena {
  background: #ffffff;
  box-shadow: 0px -3px 10px rgba(0, 58, 78, 0.1);
  border-radius: 5px;
  padding: 20px 30px;
  width: 25%;
}
.coordinates-header {
  margin-bottom: 50px;
}
.coordinates-header h3 {
  color: #1f2a53;
  font-weight: 600;
}
.coordinates-header p {
  color: rgba(13, 16, 27, 0.75);
  font-weight: 600;
  font-size: 0.875rem;
}
.form-group {
  position: relative;
}
.location-control {
  height: 30px;
  background: #ffffff;
  border: 1px solid rgba(31, 42, 83, 0.25);
  box-shadow: 0px 0px 10px rgba(73, 165, 198, 0.1);
  border-radius: 4px;
  padding: 0px 10px;
  width: 90%;
}
.location-control:focus {
  outline: none;
}
.location-btn {
  margin-top: 15px;
  padding: 10px 15px;
  background: #d80739;
  box-shadow: 0px 4px 10px rgba(73, 165, 198, 0.1);
  border-radius: 5px;
  border: 0;
  cursor: pointer;
  color: #ffffff;
  font-size: 0.875rem;
  font-weight: 600;
}
.location-btn:focus {
  outline: none;
}
.disabled {
  background: #db7990;
  cursor: not-allowed;
}
.copy-btn {
  background: #f4f6f8 0% 0% no-repeat padding-box;
  border: 1px solid #f4f6f8;
  border-radius: 0px 3px 3px 0px;
  position: absolute;
  color: #5171ef;
  font-size: 0.875rem;
  font-weight: 500;
  height: 30px;
  padding: 0px 10px;
  cursor: pointer;
  right: 3.5%;
  top: 5%;
}
.copy-btn:focus {
  outline: none;
}
  .image_container {
    width: 100%;
    display: flex;
    flex-direction: column;
    & .info_title {
      font-size: 24px;
      color: #1f2a53;
      font-weight: 600;
      margin-bottom: 15px;
    }
    & .road_content {
      margin-bottom: 20px;
      color: rgba(13, 16, 27, 0.75);
      font-weight: 600;
      font-size: 18px;
    }
    & .road_image {
      width: 65%;
      height: 120px;
      border: 1px solid #ccc;
      background-repeat: no-repeat;
      background-position: center;
      background-size: cover;
    }
  }
</style>
