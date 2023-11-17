<template>
  <div style=" padding: 5px 10px; height: 100vh;">
    <div style="float: left; width: 30%; height: 100%; padding: 20px 20px">
      <p v-if="!ifShowResubmit">Please input the start point and end point</p>
      <div>
        <div style=" margin-top: 20px;">
          <div>Starting location</div>
          <input type="text" placeholder="Please input the starting location" class="input-box" v-model="origin">
        </div>
        <div style=" margin-top: 10px" >
          <div>Drop-off point</div>
          <input type="text" placeholder="Please input the drop-off point" class="input-box" v-model="destination">
        </div>
      </div>
      <div style="margin-top: 50px" v-show="!ifShowResubmit">
        <v-btn @click="getRoute" color="#009688" height="20px" style="text-transform: none">Submit</v-btn>
        <v-btn @click="reset" color="#009688" height="20px" style="text-transform: none">Reset</v-btn>
      </div>
      <div v-show="ifShowResubmit" style="margin-top: 10px;">
        <div>
          total distance: {{totalDistance}}
        </div>
        <div style="margin-top: 10px">
          total time: {{totalTime}}
        </div>
        <div style="margin-top: 10px">
          <v-btn @click="getRoute" color="#009688" height="20px" style="text-transform: none">Re-Submit</v-btn>
          <v-btn @click="reset" color="#009688" height="20px" style="text-transform: none">Reset</v-btn>
        </div>
      </div>
    </div>
    <v-container style="float: left; width: 70%; height: 100%;" class="modal-body pa-0">
      <google-map class="map-container" :locations="locations"></google-map>
    </v-container>
  </div>
</template>

<script>
export default {
  name: "webRoute",
  layout: false,
  data(){
    return{
      origin: '',
      destination: '',
      locations: [],
      ifShowResubmit: false,
      totalDistance: 0,
      totalTime: 0,
    }
  },
  methods: {
    async getRoute(){
      if (this.origin && this.destination){
        const token = await this.getToken()
        if (token){
          await this.getPath(token)
        }
      }else{
        this.$toast.error(`Please input both of Starting location and Drop-off point`, {
          position: "top-center",
          duration: 3000,
        });
      }
    },

    reset(){
      this.origin = ''
      this.destination = ''
      this.ifShowResubmit = false
    },

    async getToken(){
      try {
        this.$toast.show('Trying ...', {
          position: "top-center",
          duration: 3000,
        })
        const response = await this.$axios.$post("https://sg-mock-api.lalamove.com/route",{
          "origin": this.origin,
          "destination": this.destination
        })
        if (response && response.token){
          return response.token
        }
      }catch (error){
        this.$toast.error(`Internal server error, please retry`, {
          position: "top-center",
          duration: 3000,
        });
      }
    },

    async getPath(token){
      while (true){
        try {
          const response = await this.$axios.$get(`https://sg-mock-api.lalamove.com/route/${token}`)
          if (response && response.status === "success"){
            this.$toast.success(`Success.`, {
              position: "top-center",
              duration: 3000,
            });
            this.totalDistance = response.total_distance
            this.totalTime = response.total_time
            this.locations = await this.getAddress(response.path)
            this.ifShowResubmit = true
            break
          }else if (response && response.status === "in progress"){
            this.$toast.show('Retrying ...', {
              position: "top-center",
              duration: 3000,
            })
            await this.sleep(2000)
          }else if (response && response.status === "failure"){
            this.$toast.error(`Location not accessible by car, please re-submit`, {
              position: "top-center",
              duration: 3000,
            });
            this.origin = ''
            this.destination = ''
            break
          }
        }catch (error){
          this.$toast.error(`Internal server error, please retry`, {
            position: "top-center",
            duration: 3000,
          });
          break
        }
      }
    },

    async getAddress(paths){
      let realPaths = []
      for (const path of paths){
        const response = await this.$axios.get(`https://maps.googleapis.com/maps/api/geocode/json?latlng=${path[0]},${path[1]}&key=${this.$config.GOOGLEMAPSAPIKEY}`)
        realPaths.push(response.data.results[0])
      }
      return realPaths
    },

    sleep(ms){
      return new Promise(resolve => setTimeout(resolve, ms));
    }
  }
}
</script>

<style scoped lang="scss">
.input-box{
  color: #4a4a4a;
  font-weight: 400;
  font-size: 15px;
  width: 60%;
  border: 1px solid #979797;
  margin-top: 10px;
  height: 30px;
}

.modal-body{
  .map-container {
    width: 100%;
    height: 100%;
  }
}

</style>
