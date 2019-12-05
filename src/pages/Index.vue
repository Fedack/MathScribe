<template>
  <q-page class="flex flex-center" padding style="padding-top: 2px">
    <MyScriptJSVueComponent />
  </q-page>
</template>

<script>
import MyScriptJSVueComponent from '../components/MyScriptJSVueComponent'
export default {
  name: 'PageIndex',
  components: {
    MyScriptJSVueComponent
  },
  methods: {
    myTweak (offset) {
      // "offset" is a Number (pixels) that refers to the total
      // height of header + footer that occupies on screen,
      // based on the QLayout "view" prop configuration

      // this is actually what the default style-fn does in Quasar
      return { minHeight: offset ? `calc(100vh - ${offset}px)` : '100vh' }
    }
  }
}
</script>
