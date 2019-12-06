<template>
  <div class="full-width" style="height: 89vh">
    <div
      class="flex row full-height q-gutter-sm"
      :class="{'no-wrap': mobile != true}"
      style="min-height: inherit"
    >
      <div
        class="writing-container shadow-2 col-9"
        touch-action="none"
        style="touch-action: none; min-height: 70vh; min-width: 400px"
        ref="editor"
      >
        <q-input
          class="q-mx-sm"
          borderless
          readonly
          v-model="toSend"
          dense
          style="display: flex; flex-grow: 1;"
        />
      </div>

      <div class="q-gutter-sm" :class="{'col-3': mobile != true, 'col-auto': mobile}" ref="output">
        <div class="flex row">
          <q-btn label="Solve" @click="convertEditor" />
          <q-space></q-space>
          <q-btn v-shortkey="['ctrl', 'z']" @shortkey="undo()" icon="mdi-undo" @click="undo" />
          <q-btn
            v-shortkey="['ctrl', 'shift', 'z']"
            @shortkey="redo()"
            icon="mdi-redo"
            @click="redo"
          />
          <q-btn
            v-shortkey="['ctrl', 'alt','shift', 'z']"
            @shortkey="clear()"
            icon="mdi-eraser"
            @click="clear"
          />
          <q-btn icon="mdi-key" @click="keyDialog = true" />
          <q-btn icon="mdi-refresh" @click="refresh" />
        </div>
        <q-scroll-area
          class="full-width full-height"
          style="max-height: 89vh"
          v-if="this.$q.platform.is.mobile != true"
        >
          <div v-if="outPut !== ''" class="q-gutter-sm q-px-sm">
            <q-card v-for="pod in outPut.queryresult.pods" :key="pod.id">
              <q-card-section>{{pod.title}}</q-card-section>
              <div v-if="pod.subpods.length > 0">
                <q-card-section v-for="subpod in pod.subpods" :key="subpod.title">
                  <img :src="subpod.img.src" />
                </q-card-section>
              </div>
            </q-card>
          </div>
        </q-scroll-area>
      </div>
    </div>
    <q-dialog
      v-model="dialog"
      persistent
      :maximized="maximizedToggle"
      transition-show="slide-up"
      transition-hide="slide-down"
    >
      <q-card class="bg-primary text-white">
        <q-bar>
          <q-space />

          <q-btn
            dense
            flat
            icon="minimize"
            @click="maximizedToggle = false"
            :disable="!maximizedToggle"
          >
            <q-tooltip v-if="maximizedToggle" content-class="bg-white text-primary">Minimize</q-tooltip>
          </q-btn>
          <q-btn
            dense
            flat
            icon="crop_square"
            @click="maximizedToggle = true"
            :disable="maximizedToggle"
          >
            <q-tooltip v-if="!maximizedToggle" content-class="bg-white text-primary">Maximize</q-tooltip>
          </q-btn>
          <q-btn dense flat icon="close" v-close-popup>
            <q-tooltip content-class="bg-white text-primary">Close</q-tooltip>
          </q-btn>
        </q-bar>

        <q-card-section>
          <div v-if="outPut !== ''" class="q-gutter-sm q-px-sm">
            <q-card v-for="pod in outPut.queryresult.pods" :key="pod.id">
              <q-card-section style="color: black">{{pod.title}}</q-card-section>
              <div v-if="pod.subpods.length > 0">
                <q-card-section v-for="subpod in pod.subpods" :key="subpod.title">
                  <img :src="subpod.img.src" />
                </q-card-section>
              </div>
            </q-card>
          </div>
        </q-card-section>
      </q-card>
    </q-dialog>
    <q-dialog v-model="keyDialog">
      <q-card>
        <q-card-section>
          <p>This program requires you to enter an API key for Wolfram.</p>
          <p>
            Register on
            <a
              href="http://developer.wolframalpha.com/portal/myapps/index.html"
              target="_blank"
            >Wolfram</a> and once logged in, under My Apps click on Get an AppID. Follow the instructions and enter your App ID below.
          </p>
        </q-card-section>
        <q-card-section>
          <q-input v-model="wolframkey" hint="App ID" />
        </q-card-section>
        <q-card-actions align="right" class="bg-white text-primary">
          <q-btn flat label="OK" @click="setWolframKey" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>

<script>
import * as MyScriptJS from 'myscript'
import axios from 'axios'
import Cookies from 'js-cookie'
export default {
  name: 'MyScriptJSVueComponent',
  data () {
    return {
      msg: 'Basic example of vue integration of MyScriptJS',
      wolfram: '',
      toSend: '',
      outPut: '',
      mobile: false,
      maximizedToggle: true,
      dialog: false,
      keyDialog: false,
      wolframkey: ''
    }
  },
  created () {
    // eslint-disable-next-line
  },
  mounted () {
    // Fired every second, should always be true
    // eslint-disable-next-line
    this.mobile = this.$q.platform.is.mobile
    var currentkey = Cookies.get('wolframkey')
    if (currentkey) {
      this.wolframkey = Cookies.get('wolframkey')
    } else {
      this.keyDialog = true
    }
    MyScriptJS.register(this.$refs.editor, {
      recognitionParams: {
        triggers: {
          exportContent: 'DEMAND'
        },
        type: 'MATH',
        protocol: 'WEBSOCKET',
        apiVersion: 'V4',
        server: {
          scheme: 'https',
          host: 'webdemoapi.myscript.com',
          applicationKey: 'f1355ec8-c74a-4da9-8d63-691ab05952eb',
          hmacKey: '752acf37-5a45-481b-9361-fcb32cd7f6a1'
        },
        v4: {
          math: { mimeTypes: ['application/x-latex'] }
        }
      }
    })
    var thisVue = this
    this.$refs.editor.addEventListener('exported', function (evt) {
      if (evt.detail) {
        thisVue.toSend = evt.detail.exports['application/x-latex'].replace(/ /g, '')
      } else {
        thisVue.toSend = ''
      }
    })
  },
  methods: {
    setWolframKey () {
      Cookies.set('wolframkey', this.wolframkey, { expires: 365 })
    },
    convertEditor () {
      console.log(this.$refs.editor)
      var thisVue = this
      this.$refs.editor.editor.export_()
      if (this.wolframkey !== '') {
        axios.get('https://cors-anywhere.herokuapp.com/http://api.wolframalpha.com/v2/query?podstate=Result__Step-by-step+solution', {
          params: { appid: this.wolframkey, input: this.toSend, output: 'json' } }).then(function (response) {
          console.log(response.data)
          thisVue.outPut = response.data
          if (thisVue.mobile) {
            thisVue.dialog = true
          }
          if (response.data.queryresult.success === false) {
            thisVue.$q.notify({
              message: 'Invalid Query',
              color: 'red',
              position: 'top'
            })
          }
        }).catch(error => {
          console.log(error)
          this.$q.notify({
            message: error,
            color: 'red'
          })
        })
      } else {
        this.keyDialog = true
      }
    },
    undo () {
      this.$refs.editor.editor.undo()
    },
    redo () {
      this.$refs.editor.editor.redo()
    },
    clear () {
      this.$refs.editor.editor.clear()
    },
    refresh () {
      MyScriptJS.register(this.$refs.editor, {
        recognitionParams: {
          triggers: {
            exportContent: 'DEMAND'
          },
          type: 'MATH',
          protocol: 'WEBSOCKET',
          apiVersion: 'V4',
          server: {
            scheme: 'https',
            host: 'webdemoapi.myscript.com',
            applicationKey: 'f1355ec8-c74a-4da9-8d63-691ab05952eb',
            hmacKey: '752acf37-5a45-481b-9361-fcb32cd7f6a1'
          },
          v4: {
            math: { mimeTypes: ['application/x-latex'] }
          }
        }
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
@import url("https://cdnjs.cloudflare.com/ajax/libs/myscript/4.3.0/myscript.min.css");
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.writing-container {
  min-height: 600px;
  min-width: 300px;
}
</style>
