<template>
  <div class="ui grid">
    <div class="six wide column red">
      <form class="ui segment large form">
                <div class="ui message red" v-show="error">{{error}}</div>
                <div class="ui segment">
                    <div class="field"> 
                        <div class="ui right icon input large" :class="{loading:spinner}">
                            <input
                              type="text"
                              placeholder="Enter your address"
                              v-model="address"
                              ref="autocomplete"
                            />
                            <i class="dot circle link icon" @click="locatorButtonPressed"></i>
                        </div>
                    </div>
                    <div class="field">
                      <div class="two fields">
                        <div class="field">
                          <select v-model="type">
                            <option value="restaurant">Restaurant</option>
                          </select>
                        </div>
                      <div class="field">
                        <select v-model="radius">
                          <option value="5">5KM</option>
                          <option value="10">10KM</option>
                          <option value="15">15KM</option>
                          <option value="20">20KM</option>
                        </select>
                      </div>
                      </div>
                      </div>
                    <button class="ui button" @click="findCloseBuyButtonPressed">Find your assets</button>
                  </div>
        </form>
        <div class="ui segment" style="max-height:780px;overflow:auto;">
          <div class="ui divided items">
            <div class="item" v-for="(place, index) in places" :key="place.id" @click="showInfoWindow(index)" :class="{'active' : activeIndex === index}" style="padding:10px" >
              <div class="content">
                <div class="header">{{place.name}}</div>
                <div class="meta">{{place.vicinity}}</div>
              </div>
            </div>
          </div>
        </div>
    </div>
    <div class="ten wide column" ref="map"></div>
    </div>
</template>


<script>
import axios from "axios";
export default {
  data() {
    return {
      address: "",
      error: "",
      spinner: false,
      apiKey: "AIzaSyC7sfWlhcNToe3jPPtNKdY6D_ZbWiPDsLM",
      lat: 0,
      lng: 0,
      type: '',
      radius: '',
      places: [],
      markers: [],
      activeIndex: -1
    };
  },
//  mounted() {
//     const autocomplete = new google.maps.places.Autocomplete(
//       this.$refs["autocomplete"],
//       {
//         bounds: new google.maps.LatLngBounds(
//           new google.maps.LatLng(45.4215296, -75.6971931)
//         )
//       }
//     );

//     autocomplete.addListener("place_changed", () => {
//       const place = autocomplete.getPlace();
//       this.address = place.formatted_address;
//       this.lat = place.geometry.location.lat();
//       this.lng = place.geometry.location.lng();
//     });
//   },

  mounted() {
    new google.maps.places.Autocomplete(this.$refs["autocomplete"], {
      bounds: new google.maps.LatLngBounds(
        new google.maps.LatLng(25.276987, 55.296249)
      )
    });
  },
  methods: {
    locatorButtonPressed() {
      this.spinner = true;

      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            this.lat = position.coords.latitude;
            this.lng = position.coords.longitude;

            this.getAddressFrom(
              position.coords.latitude,
              position.coords.longitude
            );
            // this.showUserLocationOnTheMap(
            //   position.coords.latitude,
            //   position.coords.longitude
            // );
          },
          (error) => {
            this.error =
              "Locator is unable to find address. Please type your address.";
            this.spinner = false;
            console.log(error.message);
          }
        );
      } else {
        this.error = error.message;
        this.spinner = false;
        console.log("Your browser does not support Geo Location Api");
      }
    },
    getAddressFrom(lat, long) {
      axios
        .get(
          "https://maps.googleapis.com/maps/api/geocode/json?latlng=" +
            lat +
            "," +
            long +
            "&key=" +
            this.apiKey
        )
        .then((response) => {
          if (response.data.error_message) {
            this.error = response.data.error_message;
            this.spinner = false;
            console.log(response.data.error_message);
          } else {
            this.address = response.data.results[0].formatted_address;
            this.spinner = false;
            console.log(response.data.results[0].formatted_address);
          }
        })
        .catch((error) => {
          this.error = error.message;
          console.log(error.message);
        });
    },
    // showUserLocationOnTheMap(latitude, longitude) {
    //   //Create a map object
    //   let map = new google.maps.Map(document.getElementById("map"), {
    //     zoom: 15,
    //     center: new google.maps.LatLng(latitude, longitude),
    //     mapTypeId: google.maps.MapTypeId.ROADMAP,
    //   });
    //   //Add marker
    //   new google.maps.Marker({
    //     position: new google.maps.LatLng(latitude, longitude),
    //     map: map,
    //   });
    // },
    findCloseBuyButtonPressed() {
      const URL = `https://cors-anywhere.herokuapp.com/https://maps.googleapis.com/maps/api/place/nearbysearch/json?location=${
        this.lat
      },${this.lng}&type=${this.type}&radius=${this.radius * 1000}&key=${
        this.apiKey
      }`;
      
      axios.get(URL).then(response =>{
        this.places = response.data.results;
        this.showPlacesOnMap();
        console.log(response);
      }).catch (error => {
        this.error= error.message
      })
    },
    showPlacesOnMap(){

      const map = new google.maps.Map(
        this.$refs["map"],
        {
          zoom: 15,
          center: new google.maps.LatLng(this.lat, this.lng),
          mapTypeId: google.maps.MapTypeId.ROADMAP
        }
      );

      const infoWindow =  new google.maps.InfoWindow();

        for (let i = 0; i < this.places.length; i++) {
                const lat = this.places[i].geometry.location.lat;
                const lng = this.places[i].geometry.location.lng;
                const placeID = this.places[i].place_id;

                const marker = new google.maps.Marker({
                  position: new google.maps.LatLng(lat, lng),
                  map: map
                });

                this.markers.push(marker);

                google.maps.event.addListener(marker, "click", () => {

                  const URL = `https://cors-anywhere.herokuapp.com/https://maps.googleapis.com/maps/api/place/details/json?key=${this.apiKey}&place_id=${placeID}`;

                  axios
                  .get(URL)
                  .then(response =>{
                    if(response.data.error_message){
                      this.error = response.data.error_message;
                    } else {
                      const place = response.data.result;
                      infoWindow.setContent(
                        `<div class="ui header"> ${place.name}</div>
                        ${place.formatted_address} <br>
                        ${place.formatted_phone_number} <br>
                        <a href="${place.website}" target= "_blank">${place.website}</a>
                        `
                      );
                      infoWindow.open(map,marker);
                    }
                  }).catch(error => {
                    this.error = error.message;
                  })

                });
          }                                    
      },
    showInfoWindow(index){
      this.activeIndex = index;
      new google.maps.event.trigger(this.markers[index], 'click');
    }
  },
};
</script>

<style>
.ui.button,
.dot.circle.icon {
  background-color: #ff5a5f;
  color: white;
}

.pac-icon {
  display: none;
}

.pac-item {
  padding: 10px;
  font-size: 16px;
  cursor: pointer;
}

.pac-item:hover {
  background-color: #ececec;
}

.pac-item-query {
  font-size: 16px;
}

#map {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0%;
  left: 0;
  background-color: tomato;
}

.active{
  background-color: #ff5a5f !important;
}
</style>