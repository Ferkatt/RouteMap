<template>
  <div class="map-container">
    <div id="google-map"></div>
  </div>
</template>

<script>
import {Loader} from '@googlemaps/js-api-loader';
const mapOptions = {
  center: {
    lat: 22.372081,
    lng: 114.107877
  },
  zoom: 4
};
export default {
  props: {
    locations: {
      type: Array,
      default: []
    },
  },
  data() {
    return {
      locations: this.locations,
      place: {},
    };
  },
  async mounted() {
    let googleApiloader = new Loader({
      apiKey: this.$config.GOOGLEMAPSAPIKEY,
      libraries: ["places"],
    });
    googleApiloader.load()
      .then((google) => {
        let map = new google.maps.Map(this.$el.querySelector("#google-map"), mapOptions);
      });
  },
  watch: {
    locations(){
      let markers = []
      const labels = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
      let labelIndex = 0;
      let googleApiloader = new Loader({
        apiKey: this.$config.GOOGLEMAPSAPIKEY,
        libraries: ["places"],
      });
      const places = this.locations
      const center = Math.floor(places.length/2)
      const centerPosition = [places[center].geometry.location.lat, places[center].geometry.location.lng]
      googleApiloader.load()
        .then((google) => {
          let map = new google.maps.Map(this.$el.querySelector("#google-map"), {
            zoom: 12,
            center: {
              lat: centerPosition[0],
              lng: centerPosition[1]
            }
          });

          for (let place of places){
            const marker = new google.maps.Marker({
              map: map,
              position: {
                lat: place.geometry.location.lat,
                lng: place.geometry.location.lng
              },
              label: labels[labelIndex++ % labels.length],
            });
            markers.push(marker)
          }
        });
    }
  },
};
</script>

<style lang="scss">
.map-container{
  #google-map {
    height: 100%;
    width: 100%;
    left: 0;
    background-color: #cecece;
    border-radius: 5px;
    border: 1px solid green;
  }
}
</style>

