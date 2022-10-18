<template>
  <main>
    <div @click="$emit('update:showCropping', false)">
        Go Back
    </div>
    <div class="image-container">
      <div class="cropper">
        <vue-cropper
          ref="cropper"
          :aspect-ratio="8.56 /5.398"
          :src="backgroundImage"
          preview=".preview"
          :scalable="false"
          :highlight="false"
          :rotate=90
          :background= backgroundColor
        />

        <div class="options">
            <div class="options">
               <div @click="eyeDropper" class="color-preview"></div>
               <input v-model="backgroundColor"/>
            </div>
          <button @click="cropImage">Crop Image</button>
        </div>
      </div>

      <div v-if="cropped">
        <img class="cropped" :src="cropImg" />
      </div>
    </div>
  </main>
</template>
<script>
//implement the vue croppper.js
// https://github.com/fengyuanchen/cropperjs
import "cropperjs/dist/cropper.css";
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
      data: null,
      rotate: 45,
      backgroundColor: "transparent",
    };
  },
  methods: {
    cropImage() {
      this.cropImg = this.$refs.cropper
        .getCroppedCanvas({ fillColor: this.backgroundColor, height: "204px" })
        .toDataURL();
      this.cropped = true;
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
  },
  props:{
    backgroundImage: String
  }
};
</script>

<style scoped>
.image-container {
  width: calc(100vw - 300px);
  height: auto;
  gap: 3rem;
  display: flex;
}
.cropper{
    display: flex;
    flex-direction: column;
    gap: 3rem;
}
.color-preview {
  height: 3rem;
  width: 3rem;
  border: 1px solid grey;
  border-radius: 8px;
  background-color: v-bind("backgroundColor");
}
.color-preview:hover{

    cursor: crosshair;
  opacity: 0.5
}
.options {
  display: flex;
  align-content: center;
  gap: 1rem;
    align-items: center;
}
.options input{
    border: none;
    font-size: 16px;
    width: 4.5rem;
}
.cropped {
  width: 8.56cm;
  height: 5.398cm;
  border: 1px solid grey;
}
button{
    padding: 1rem 1rem;
    background-color: #5BA6AF;
    border-radius: 4px;
    border: none;
    font-size: 16px;
    font-weight: 400;
}
</style>