<template>
  <div>
    <div>
      <h2>Search and add a pin</h2>
      <label>
        <gmap-autocomplete @place_changed="setPlace"> </gmap-autocomplete>

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
      <gmap-marker
        :key="index"
        v-for="(m, index) in markers"
        :position="m.position"
        @click="center = m.position"
      ></gmap-marker>
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
      // default to Montreal to keep it simple
      // change this to whatever makes sense
      center: { lat: 45.508, lng: -73.587 },
      markers: [],
      places: [],
      currentPlace: null,
      restaurant: ["default shop"],
    };
  },

  mounted() {
    this.geolocate();
  },

  methods: {
    async fetchLocation(place_name) {
      this.restaurant = [];
      console.log(place_name);
      const proxyurl = "https://cors-anywhere.herokuapp.com/";
      const url = `https://maps.googleapis.com/maps/api/place/textsearch/json?query=restaurants+in+${place_name}&radius=10000&key=${api.Key}`;
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
              console.log(Item.name);
              let data = {
                name: Item.name,
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