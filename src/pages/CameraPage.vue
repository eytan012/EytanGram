<template>
  <q-page class="constrain-more q-pa-md ">
    <div class="camera-frame q-pa-md">
      <video v-show="!imageCaptured" class="full-width" ref="video" autoplay/>
      <canvas v-show="imageCaptured" ref="canvas" class="full-width" height="200"/>

    </div>
    <div class="text-center q-pa-md">
      <q-btn v-if="hasCameraSupport" @click="captureImage" :disable="imageCaptured" round color="grey-10" icon="eva-camera"
             size="lg"/>
      <div v-else class="text-center q-pa-md">
        <q-file v-model="imageUpload" accept="image/*" label="Choose an Image" outlined @input="captureImageFallBack">
          <template v-slot:prepend>
            <q-icon name="attach_file"/>
          </template>
        </q-file>
      </div>
    </div>
    <div>
      <button id="btn-front">Selfie</button>
      <button id="btn-back">Main</button>
    </div>
    <div class="row justify-center q-ma-md">
      <q-input v-model="post.caption" class="col col-sm-6" label="caption *" dense clearable>
        <template v-slot:append>
          <q-btn round dense flat icon="location"/>
        </template>
      </q-input>
    </div>
    <div class="row justify-center q-ma-md">
      <q-input :loading="locationLoading" v-model="post.location" class="col col-sm-6" label="Location" dense>
        <template v-slot:append>
          <q-btn v-if="!locationLoading && locationSupported" @click="getLocation" round dense flat icon="eva-navigation-2-outline"/>
        </template>
      </q-input>

    </div>
    <div class="row justify-center q-mt-lg">
      <q-btn @click="addPost"  :disable="!post.photo || !post.caption" color="primary" label="New Post" rounded
             unelevated/>
    </div>

  </q-page>
</template>

<script>
import {uid} from 'quasar'
require('md-gum-polyfill')

export default {
  name: 'CameraPage',
  data() {
    return {
      post: {
        id: uid(),
        caption: '',
        location: '',
        photo: null,
        date: Date.now()
      },
      imageCaptured: false,
      imageUpload: [],
      hasCameraSupport: true,
      locationLoading: false
    }
  },
  computed: {
    locationSupported() {
      if ('geolocation' in navigator) return true
      return false
    }
  },
  methods: {

    iniCamera() {
      navigator.mediaDevices.getUserMedia({
        video: true,
        facingMode: {
          exact: 'environment'
        }
      }).then(stream => {
        this.$refs.video.srcObject = stream
      }).catch(error => {
        this.hasCameraSupport = false;
      })
    },
    captureImage() {
      let video = this.$refs.video
      let canvas = this.$refs.canvas
      canvas.width = video.getBoundingClientRect().width
      canvas.height = video.getBoundingClientRect().height
      let context = canvas.getContext('2d')
      context.drawImage(video, 0, 0, canvas.width, canvas.height)
      this.imageCaptured = true
      this.post.photo = this.dataURItoBlob(canvas.toDataURL())
      this.disableCamera()
    },
    captureImageFallBack(file) {
      this.post.photo = file
      let canvas = this.$refs.canvas
      let context = canvas.getContext('2d')

      let reader = new FileReader()
      reader.onload = event => {
        let img = new Image()
        img.onload = () => {
          canvas.width = img.width
          canvas.height = img.height
          context.drawImage(img, 0, 0)
          this.imageCaptured = true
        }
        img.src = event.target.result;
      }
      reader.readAsDataURL(file);
    },
    disableCamera() {
      this.$refs.video.srcObject.getAudioTracks().forEach(track => {
        track.stop()
      })
    },
    dataURItoBlob(dataURI) {
      // convert base64 to raw binary data held in a string
      // doesn't handle URLEncoded DataURIs - see SO answer #6850276 for code that does this
      var byteString = atob(dataURI.split(',')[1]);

      // separate out the mime component
      var mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0]

      // write the bytes of the string to an ArrayBuffer
      var ab = new ArrayBuffer(byteString.length);

      // create a view into the buffer
      var ia = new Uint8Array(ab);

      // set the bytes of the buffer to the correct values
      for (var i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i);
      }

      // write the ArrayBuffer to a blob, and you're done
      var blob = new Blob([ab], {type: mimeString});
      return blob;

    },
    getLocation() {
      this.locationLoading = true
      navigator.geolocation.getCurrentPosition(postion => {
        this.getCityAndCountry(postion)
      }, err => {
        this.locationError()
      }, {timeout: 7000})
    },
    getCityAndCountry(position) {
      let apiUrl = `https://geocode.xyz/${position.coords.latitude},${position.coords.longitude}?json=1`
      this.$axios.get(apiUrl).then(result => {
        this.locationSuccess(result)
      }).catch(err => {
        this.locationError()
      })
    },
    locationSuccess(result) {
      this.post.location = result.data.city
      if (result.data.country) {
      }
      this.post.location += `, ${result.data.country}`
      this.locationLoading = false
    },
    locationError() {
      this.$q.dialog({
        title: 'Alert',
        message: 'Somthing Went Wrong. Please Try Again'
      })
      this.locationLoading = false
    },
    addPost(){
      this.$q.loading.show()
      let formData = new FormData()
      formData.append('id', this.post.id)
      formData.append('caption', this.post.caption)
      formData.append('location', this.post.location)
      formData.append('date', this.post.date)
      formData.append('file', this.post.photo, this.post.id + '.png')

    this.$axios.post(`${process.env.API}/createPost`, formData).then(res => {
      console.log(res)
      this.$router.push('/')

      this.$q.notify({
        message: 'Post is in the air',
        color: 'black',
        timeout: 2000,
      })
      this.$q.loading.hide()
    }).catch(err => {
     this.$q.dialog({
        title: 'Error',
        message: 'Something Went Wrong. Please Try Again'
      })
      this.$q.loading.hide()
    })
    }
  },

  mounted() {
    this.iniCamera()
  },
  beforeDestroy() {
    if (this.hasCameraSupport) {
      this.disableCamera()
    }
  }
}
</script>

<style lang="scss">
.camera-frame {
  border: 1px solid $grey-10;
  border-radius: 10px;
}

</style>
