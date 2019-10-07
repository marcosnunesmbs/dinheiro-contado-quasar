<template>
  <q-layout view="lHh Lpr lFf">
    <q-layout-header>
      <q-toolbar
          color="indigo-10"
      >

        <q-toolbar-title>
          <div class="row justify-center">
            <span>Dinheiro Contado</span>
          </div>
        </q-toolbar-title>
      </q-toolbar>
    </q-layout-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { openURL } from 'quasar'

export default {
  name: 'LayoutDefault',
  data () {
    return {
      leftDrawerOpen: this.$q.platform.is.desktop
    }
  },
  methods: {
    openURL
  }
}
</script>

<style>
  .q-toolbar.row.no-wrap.items-center.relative-position.q-toolbar-normal.bg-primary.text-white {
    height: 50px!important;
  }
  .q-toolbar-title{
    font-size: 20px;
  }
</style>
