<template>
  <div class="home">
    <div class="top-bar">
      <h1>Web Extractor</h1>
      <div class="url-form">
        <b-field class="url-form-input">
          <b-input placeholder="Website URL" rounded v-model="urlForm.url"></b-input>
        </b-field>
        <b-button class="url-form-button" :loading="urlForm.loading" rounded type="is-primary" @click="go">{{ urlForm.button.label }}</b-button>
      </div>
    </div>
    <div class="top-debug-bar">
      <div class="debug-selector">
        Path: {{ selector.path }}<br />
        Type: {{ selector.type }}<br />
        <div v-if="selector.value !== ''">
          Value: {{ selector.value }}
        </div>
    </div> 
    </div>
    <div class="iframe-window" v-show="urlForm.loaded">
      <i-frame ref="iframe"></i-frame>
    </div>
    <!-- <div>
      <input type="text" @keyup.enter="Go" v-model="externalPage" />
      <button @click="Go">Go</button>
    </div> -->
  </div>
</template>

<script>
import EventBus from "../EventBus";
import iFrame from "@/views/iFrame.vue";

export default {
  name: "home",
  data() {
    return {
      urlForm: {
        loading: false,
        loaded: false,
        button: {
          label: "Go"
        },
        url: "",
      },
      selector: {
        path: "",
        type: "",
        value: "",
      }
    }
  },
  components: {
    iFrame
  },
  methods: {
    go() {
      this.urlForm.loading = true;
      this.$refs.iframe.Go(this.urlForm.url);
    }
  },
  created () {
    EventBus.$on('IFRAME_LOADED', ({ loading, loaded, error }) => {
      if (error) {
        alert("URL Error");
      }
      this.urlForm.loading = loading;
      this.urlForm.loaded = loaded;
    })
    EventBus.$on('IFRAME_ELEMENT_POINTING', (selection) => {
      if (this.urlForm.loaded) {
        this.selector.type = selection.type;
        this.selector.value = selection.value;
      }
    })
    EventBus.$on('IFRAME_ELEMENT_SELECTING', (selection) => {
      if (this.urlForm.loaded) {
        this.selector.path = selection.path;
        this.selector.type = selection.type;
        this.selector.value = selection.value;
      }
    })
  }
};
</script>

<style lang="scss">
.url-form {
  display: flex;
  width: 80%;
  max-width: 600px;
  justify-content: center;
  align-items: center;
  margin: auto;

  & .url-form-input {
    width: 100%;
    margin-bottom: 0!important;

    & .control input[type="text"] {

      border-top-right-radius: 0;
      border-bottom-right-radius: 0;
    }
  }

  & .url-form-button {
    border-top-left-radius: 0!important;
    border-bottom-left-radius: 0!important;
  }
}

.top-bar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  background: white;
  height: 100px;
}

.top-debug-bar {
  position: fixed;
  top: 100px;
  left: 0;
  right: 0;
  background: rgb(214, 214, 214);
  height: 100px;
}

.iframe-window {
  position: absolute;
  top: 200px;
  left: 0;
  right: 0;
  bottom: 100px;
}
</style>