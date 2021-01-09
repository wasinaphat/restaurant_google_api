<template>
  <div>
    <div>
      <h2>Search your favorite restaurant</h2>
      <label>
        <gmap-autocomplete @place_changed="setPlace"> </gmap-autocomplete>
        <div class="slidecontainer">
          <input
            type="range"
            min="1"
            max="10"
            value="5"
            class="slider"
            id="myRange"
            v-model="distance"
          />
          <p>Search restaurant with disance : {{ distance }} km</p>
        </div>
        <button @click="addMarker">Add</button>
      </label>
      <br />
    </div>
    <div>
      <ul v-for="item of restaurant" :key="item.name">
        <li>{{ item.name }}</li>
      </ul>
    </div>
    <br />
    <gmap-map :center="center" :zoom="12" style="width: 100%; height: 1024px">
      <!-- <gmap-marker
        :key="index"
        v-for="(m, index) in markers"
        :position="m.position"
        @click="center = m.position"
      > -->
      <gmap-marker
        :key="index"
        v-for="(m, index) in markers"
        :position="m.position"
        @click="toggleInfoWindow(m, index)"
      >
      </gmap-marker>
      <!-- <gmap-info-window
        @closeclick="window_open = false"
        :opened="window_open"
        :position="infowindow"
        :options="{
          pixelOffset: {
            width: 0,
            height: -35,
          },
        }"
      >
        <h1>Choosen</h1>
      </gmap-info-window> -->
      <gmap-info-window
        :options="infoOptions"
        :position="infoWindowPos"
        :opened="infoWinOpen"
        @closeclick="infoWinOpen = false"
      >
        ร้าน : {{ DisplayRestaurant }}
      </gmap-info-window>
    </gmap-map>
  </div>
</template>

<script>
// import axios from 'axios'
import api from "../../api.json";
export default {
  name: "GoogleMap",
  data() {
    return {
      distance: 1,
      // default to Montreal to keep it simple
      // change this to whatever makes sense
      center: { lat: 45.508, lng: -73.587 },
      markers: [],
      places: [],
      currentPlace: null,
      restaurant: ["default shop"],
      DisplayRestaurant: "",
      infoWindowPos: null,
      infoWinOpen: false,
      infoOptions: {
        content: "",
        pixelOffset: {
          width: 0,
          height: -35,
        },
      },
    };
  },

  mounted() {
    this.geolocate();
  },
  computed: {},

  methods: {
    toggleInfoWindow(marker, idx) {
      let value = this.restaurant.filter((Item) => {
        return (
          Item.location.lat === marker.position.lat &&
          Item.location.lng === marker.position.lng
        );
      });

      this.DisplayRestaurant = JSON.stringify(value[0].name);
      console.log("place", JSON.stringify(value[0].name));
      this.infoWindowPos = marker.position;
      this.infoOptions.content = JSON.stringify(value[0].name);
      console.log(idx);
      this.infoWinOpen = true;
      
    },

    openWindow() {
      this.window_open = true;
    },
    async fetchLocation(place_name) {
      this.restaurant = [];
      this.markers = [];
      console.log(place_name);
      const proxyurl = "https://cors-anywhere.herokuapp.com/";
      const url = `https://maps.googleapis.com/maps/api/place/textsearch/json?query=restaurants+in+${place_name}&radius=${
        this.distance * 1000
      }&key=${api.Key}`;
      await fetch(proxyurl + url, {
        method: "GET", // or 'PUT'
        headers: {
          "Content-Type": "application/json",
          "Access-Control-Allow-Origin": "*",
        },
      })
        .then(async (response) => {
          let value = await response.json();
          //  console.log(value.results)
          if (value.results.length > 0) {
            value.results.forEach((Item) => {
              console.log(Item.geometry.location);
              this.markers.push({ position: Item.geometry.location });

              let data = {
                name: Item.name,
                location: {
                  lat: Item.geometry.location.lat,
                  lng: Item.geometry.location.lng,
                },
              };
              this.restaurant.push(data);
            });
            console.log(this.restaurant);
          }
        })
        .then((contents) => console.log(contents))
        .catch(() =>
          console.log("Can’t access " + url + " response. Blocked by browser?")
        );
    },

    setPlace(place) {
      this.fetchLocation(place.name);
      //   console.log(place.place_id);
      this.currentPlace = place;
    },
    addMarker() {
      if (this.currentPlace) {
        const marker = {
          lat: this.currentPlace.geometry.location.lat(),
          lng: this.currentPlace.geometry.location.lng(),
        };
        this.markers.push({ position: marker });
        this.places.push(this.currentPlace);
        this.center = marker;
        this.currentPlace = null;
      }
    },
    geolocate: function () {
      navigator.geolocation.getCurrentPosition((position) => {
        //   console.log(position)
        this.center = {
          lat: position.coords.latitude,
          lng: position.coords.longitude,
        };
      });
    },
  },
};
</script>