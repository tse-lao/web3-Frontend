<template>
  <main>
    <div>
      <div @click="$emit('update:showCropping', false)" class="header">
        <button>
          <img src="../assets/icons/reset.png" class="icon" />
          Go Back
        </button>
      </div>
    </div>
    <div class="image-container" v-if="!loading">
      <div class="cropper">
        <div class="image-cropper" @click="changeMovable">
          <vue-cropper
            ref="cropper"
            id="cropper"
            :aspect-ratio="ratio"
            :src="backgroundImage"
            :center="false"
            :guides="false"
            :scalable="false"
            :cropBoxResizable="false"
            :cropBoxMovable="false"
            :modal="true"
            :autoCrop="true"
            dragMode="move"
            :toggleDragModeOnDblclick="false"
            :background="false"
            :containerStyle="
              'background:' +
              backgroundColor +
              ';  &-modal{background-color: none; opacity:none; border: 1px solid #000;}' +
              'border: 1px solid #000'
            "
          />
        </div>
        <div class="options">
          <div class="options">
            <div @click="eyeDropper" class="color-preview"></div>
            <input v-model="backgroundColor" />
          </div>
          <button @click="rotateImage">
            <img class="icon" src="../assets/icons/rotate.png" alt="rotate" />
          </button>
          <button @click="resetSettings">
            <img class="icon" src="../assets/icons/reset.png" alt="rotate" />
          </button>
          <button @click="cropImage">
            <img
              src="../assets/icons/edit-tools.png"
              class="icon"
              alt="create"
            />
          </button>
        </div>
      </div>

      <div v-if="cropped" class="preview">
        <img class="cropped" :src="cropImg" />

          <span class="company-name">UNBANKD</span>
          <img src="../assets/chip.png" class='chip'/>
          <span class="account-number">000 0000 000  000</span>
          <span class="bank-account-name">C.S. Backname</span>
        <button @click="uploadImage">Upload to IPFS</button>
      </div>
    </div>
  </main>
</template>
<script>
//implement the vue croppper.js
// https://github.com/fengyuanchen/cropperjs
import { Buffer } from "buffer";
import "cropperjs/dist/cropper.css";
import { create } from "ipfs-http-client";
import VueCropper from "vue-cropperjs";

export default {
  components: {
    VueCropper,
  },
  data() {
    return {
      imgSrc:
        "https://res.cloudinary.com/alchemyapi/image/upload/mainnet/32e76bd2af54947b98f96987261b912d.png",
      cropImg: "",
      loading: true,
      cropped: false,
      ratio: 8560 / 5398,
      rotate: 45,
      backgroundColor: "",
      horizontal: true,
      loaded: false,
    };
  },
  methods: {
    cropImage() {
      this.cropImg = this.$refs.cropper
        .getCroppedCanvas({ fillColor: this.backgroundColor, height: "204px" })
        .toDataURL();
      this.cropped = true;
    },
    getFirstPixel(){
      var canvas = document.createElement('canvas');
      var imgEl = new Image();
      imgEl.src = this.backgroundImage;
      imgEl.crossOrigin = "anonymous";
      
      imgEl.onload = () => {
        canvas.width = 1;
        canvas.height = 1;
        canvas.getContext('2d').drawImage(imgEl, 0, 0);
        
        var firstPixel = canvas.getContext('2d').getImageData(0,0,1,1).data;
        
        console.log(firstPixel);
        
        const hex = "#" + (1 << 24 | firstPixel[0] << 16 | firstPixel[1] << 8 | firstPixel[2]).toString(16).slice(1);
        this.backgroundColor = hex;
        this.loading = false;
      }
      
      
      
    },
    getAverageRGB() {
      var imgEl = new Image();
      imgEl.src = this.backgroundImage;
      imgEl.crossOrigin = "anonymous";

      var blockSize = 100000,
        defaultRGB = { r: 0, g: 0, b: 0 }, // for non-supporting envs
        canvas = document.createElement("canvas"),
        context = canvas.getContext && canvas.getContext("2d"),
        data,
        width,
        height,
        i = -4,
        length,
        rgb = { r: 0, g: 0, b: 0 },
        count = 0;

      if (!context) {
        return defaultRGB;
      }

      imgEl.onload = () =>{
        height = canvas.height =
          imgEl.naturalHeight || imgEl.offsetHeight || imgEl.height;
        width = canvas.width =
          imgEl.naturalWidth || imgEl.offsetWidth || imgEl.width;
        context.drawImage(imgEl, 0, 0);

        try {
          data = context.getImageData(0, 0, width, height);
        } catch (e) {
          /* security error, img on diff domain */
          return defaultRGB;
        }

        length = data.data.length;

        console.log(length);
        while ((i += blockSize * 4) < length) {
          ++count;
          rgb.r += data.data[i];
          rgb.g += data.data[i + 1];
          rgb.b += data.data[i + 2];
        }

        // ~~ used to floor values
        rgb.r = ~~(rgb.r / count);
        rgb.g = ~~(rgb.g / count);
        rgb.b = ~~(rgb.b / count);

        const hex = "#" + (1 << 24 | rgb.r << 16 | rgb.g << 8 | rgb.b).toString(16).slice(1);
        
        console.log(hex);
        this.backgroundColor = hex;
        this.loading = false
      };

    },

    resetSettings() {
      this.$refs.cropper.reset();
    },

    rotateImage() {
      this.$refs.cropper.rotate(45);
    },
    changeMovable() {
      this.$refs.cropper.setDragMode("move");
    },
    previewUpdate() {
      this.cropImage();
    },

    async eyeDropper() {
      //check if eyedropper is support3ed
      if (!window.EyeDropper) {
        console.alert("eyedropper is not supported try, Chrome");
      }

      let eyeDropper = new window.EyeDropper();
      const { sRGBHex } = await eyeDropper.open();
      this.backgroundColor = sRGBHex;

      this.cropImage();
    },

    async uploadImage() {
      const id = process.env.VUE_APP_INFURA;
      const key = process.env.VUE_APP_INFURA_SECRET;
      const auth = "Basic " + Buffer.from(id + ":" + key).toString("base64");
      const client = create({
        host: "ipfs.infura.io",
        port: 5001,
        protocol: "https",
        headers: {
          authorization: auth,
        },
      });

      try {
        const upload = await client.add(this.cropImg);
        console.log(upload);
      } catch (error) {
        console.log(error);
      }
    },
  },
  mounted() {
    this.getFirstPixel();
  },
  props: {
    backgroundImage: String,
  },
};
</script>

<style scoped>
.header {
  text-align: left;
  margin-top: 2rem;
  margin-left: 1rem;
}
.image-container {
  display: flex;
  flex-wrap: wrap;
  height: auto;
  gap: 3rem;
  display: flex;
  align-content: center;
  align-items: center;
}
.cropper {
  display: flex;
  flex-direction: column;
  padding: 12px;
  align-content: center;
  flex-wrap: wrap;
  max-width: 500px;
}
.color-preview {
  height: 2rem;
  width: 2rem;
  border: 1px solid grey;
  border-radius: 8px;
  margin: 1rem;
  border-radius: 6px;
  background-color: v-bind("backgroundColor");
}
.color-preview:hover {
  cursor: crosshair;
   opacity: 0.5;
}
.options {
  display: flex;
  align-content: center;
  gap: 1rem;
  align-items: center;
  background: rgba(0,0,0,0.1);
}
.options input {
  border: none;
  font-size: 16px;
  padding: 0.5rem;
  border-radius: 6px;
  width: 4.5rem;
}
.cropped {
  max-width: 85.6mm;
  aspect-ratio: v-bind(ratio);
  border: 1px solid grey;
  align-content: center;
  border-radius: 8px;
}
.icon {
  height: 1rem;
  width: 1rem;
}
.preview {
  display: flex;
  flex-direction: column;
  gap: 2rem;
  align-content: center;
  position: relative;
}
button {
  padding: 0.5rem 0.5rem;
  background-color: #5ba6af;
  border-radius: 4px;
  border: none;
  font-size: 16px;
  font-weight: 400;
}

 .company-name{
  position:absolute;
  right: 10px;
  top: 14px;
  font-weight: 800;
}

.chip{
  position:absolute;
  max-height: 84px;
  left: 25px;
  top: 50px;
}
.account-number{
  position:absolute;
  top: 140px;
  left: 25px;
  font-size: 20px;
  font-weight: 900;
  letter-spacing: 5px;
  text-shadow: 2px 7px 5px rgba(0,0,0,0.3), 
    0px -4px 10px rgba(255,255,255,0.3);
font-style: normal;
font-size: 16px;
line-height: 24px;
/* identical to box height, or 150% */

color: black;


  
}
.bank-account-name{
  position:absolute;
  top: 160px;
  left: 25px;
  font-size: 20px;
  font-weight: 600;
  letter-spacing: 5px;
  text-shadow: 2px 7px 5px rgba(0,0,0,0.3), 
    0px -4px 10px rgba(255,255,255,0.3);
  font-style: normal;
  font-size: 16px;
  line-height: 24px;
  /* identical to box height, or 150% */

color: black;
}
</style>