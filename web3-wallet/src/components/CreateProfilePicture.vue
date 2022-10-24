<template>
  <main>
    <div @click="$emit('update:showCropping', false)">Go Back</div>
    <div class="image-container">
      <div class="cropper">
        <vue-cropper
          ref="cropper"
          id="cropper"
          :aspect-ratio="ratio"
          :src="backgroundImage"
          preview=".preview"
          :scalable="false"
          :highlight="false"
          :rotate="90"
          :background="backgroundColor"
        />

        <div class="options">
          <div class="options">
            <div @click="eyeDropper" class="color-preview"></div>
            <input v-model="backgroundColor" />
          </div>
          <button @click="changeRatio">Rotate</button>
          <button @click="cropImage">Crop Image</button>
        </div>
      </div>

      <div v-if="cropped">
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
      backgroundColor: "transparent",
      data: {
        imgSrc:
          "https://res.cloudinary.com/alchemyapi/image/upload/mainnet/32e76bd2af54947b98f96987261b912d.png",
        cropImg: "",
        cropped: false,
        data: null,
        ratio: 8.56 / 5.398,
        rotate: 45,
        backgroundColor: "transparent",
      },
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
      if (this.ratio === 8.56 / 5.398) {
        console.log("this ratio is the same;");
        this.ratio = 5.398 / 8.56;

        //we cahnge the settings of everything

        this.data.backgroundColor = this.backgroundColor;
        this.data.ratio = this.ratio;

        this.$refs.cropper.setData({ "aspect-ratio": this.ratio });

        this.$refs.cropper.rotate(45);

        console.log(this.ratio);
      } else {
        this.ratio = 8.56 / 5.398;
        console.log(this.ratio);
      }
    },

    async eyeDropper() {
      //check if eyedropper is support3ed
      if (!window.EyeDropper) {
        console.alert("eyedropper is not suppoerted try, Chrome");
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
.image-container {
  display: flex;
  flex-wrap: wrap;
  height: auto;
  gap: 3rem;
  display: flex;
}
.cropper {
  display: flex;
  flex-direction: column;
  gap: 3rem;
  padding: 12px;
  flex-wrap: wrap;
}
.color-preview {
  height: 3rem;
  width: 3rem;
  border: 1px solid grey;
  border-radius: 8px;
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
  max-height: calc(100vw - 50px);
  aspect-ratio: 8.56 /5.398;
  border: 1px solid grey;
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