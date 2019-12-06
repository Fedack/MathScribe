<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated v-if="mobile != true">
      <q-toolbar class="bg-teal-10">
        <q-btn label="MathScribe" no-caps flat text-color="teal-2" />
      </q-toolbar>
    </q-header>

    <q-page-container class="bg-grey-10">
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
