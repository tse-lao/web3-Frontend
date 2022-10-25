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
    <div class="image-container">
      <div class="cropper">
        <div class="image-cropper">
          <vue-cropper
          ref="cropper"
          id="cropper"
          :aspect-ratio="ratio"
          :src="backgroundImage"
          preview=".preview"
          :scalable="false"
          :highlight="false"
          :modal=false
          :background=false
          :containerStyle="'background:'+ backgroundColor+';  &-modal{background-color: none; opacity:none}'"
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
          <button @click="changeRatio">
              <img v-if="horizontal" src="../assets/icons/vertical.png" class="icon"  alt="horizontal" />
              <img v-else src="../assets/icons/horizontal.png" class="icon"  alt="vertical" />
          </button>
          <button @click="cropImage">
            <img src="../assets/icons/edit-tools.png" class="icon" alt="create" />
          </button>
        </div>
      </div>

      <div v-if="cropped" class="preview">
        <img class="cropped" :src="cropImg" />
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
      cropped: false,
      ratio: 8.56 / 5.398,
      rotate: 45,
      background: false,
      backgroundColor: "#F2F2F2",
      horizontal: true, 
    };
  },
  methods: {
    cropImage() {
      this.cropImg = this.$refs.cropper
        .getCroppedCanvas({ fillColor: this.backgroundColor, height: "204px" })
        .toDataURL();
      this.cropped = true;
    },

    changeRatio() {
      console.log("clicked")
      if (this.horizontal) {
        console.log("clicked horizontal")
          this.ratio = 5.398 / 8.56;
          this.horizontal = false;
          this.$refs.cropper.setAspectRatio(this.ratio)

      } else {
        this.ratio = 8.56 / 5.398;
        this.horizontal = true;
        this.$refs.cropper.setAspectRatio(this.ratio)
      }
    },
    
    resetSettings(){
      this.$refs.cropper.reset();
    },
    
    rotateImage(){
      this.$refs.cropper.rotate(45)
    },

    async eyeDropper() {
      //check if eyedropper is support3ed
      if (!window.EyeDropper) {
        console.alert("eyedropper is not supported try, Chrome");
      }

      let eyeDropper = new window.EyeDropper();
      const { sRGBHex } = await eyeDropper.open();
      console.log(sRGBHex);
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
        console.log(error)
      }
    },
  },
  props: {
    backgroundImage: String,
  },
};
</script>

<style scoped>
.header{
  text-align:left;
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
}
.color-preview {
  height: 3rem;
  width: 3rem;
  border: 1px solid grey;
  border-radius: 8px;
  margin: 2rem;
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
}
.options input {
  border: none;
  font-size: 16px;
  width: 4.5rem;
}
.cropped {
  max-width: 8.56cm;
  aspect-ratio: v-bind(ratio);
  border: 1px solid grey;
  align-content: center;
}
.icon{
  height: 1rem;
  width: 1rem;
}
.preview{
  display: flex;
  flex-direction: column;
  gap: 2rem;
  align-content: center;
}
button {
  padding: 1rem 1rem;
  background-color: #5ba6af;
  border-radius: 4px;
  border: none;
  font-size: 16px;
  font-weight: 400;
}
</style>