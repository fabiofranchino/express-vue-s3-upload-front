<template>
  <div>
    <input type="file" class="filer" @change="onFile" />
    <button @click="upload" v-if="file">Upload</button>
    <button @click="abort" v-if="file">Abort</button>
    <p v-if="file">{{file.name}} - {{file.type}} - {{loaded}} / {{file.size}}</p>
  </div>
</template>



<script>
import Vue from 'vue'
import axios from "axios";
const CancelToken = axios.CancelToken;

export default {
  data() {
    return {
      file: null,
      loaded: 0,
      sourceToken: null
    };
  },
  methods: {
    async upload() {
      this.sourceToken = CancelToken.source();
        
      // get the s3 signed token
      let res1 = await axios.get(
        `${process.env.VUE_APP_API_ENDPOINT}/s3?filename=${this.file.name}&filetype=${this.file.type}`
      );

      // try to upload on s3
      let res2 = null;
      try {
        res2 = await axios.put(res1.data.signedRequest, this.file, {
          onUploadProgress: pEvent => this.loaded = pEvent.loaded,
          headers: { "Content-Type": this.file.type },
          cancelToken: this.sourceToken.token
        });
      } catch (err) {
        // canceled by the user
        if (axios.isCancel(err)) {
          return;
        } else {
          throw err;
        }
      }

      // say Ya!
      console.log(res2);
    },

    onFile(evt) {
      this.file = evt.target.files[0]
    },

    abort() {
      this.sourceToken.cancel("Operation canceled by user.")
      this.loaded = 0
    }
  }
};
</script>