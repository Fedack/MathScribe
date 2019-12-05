<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated v-if="mobile != true">
      <q-toolbar></q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { Platform } from 'quasar'
export default {
  name: 'MyLayout',

  data () {
    return {
      leftDrawerOpen: false,
      mobile: false
    }
  },
  mounted () {
    this.mobile = Platform.is.mobile
    console.log(this.mobile)
  }
}
</script>
