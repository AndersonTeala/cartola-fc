<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>

        <q-toolbar-title>
          <router-link to="/">
            <img src="cartola-fc.png" alt="" style="height:2em;" />
          </router-link>
        </q-toolbar-title>

        <div>Cartola-FC v{{ $q.version }}</div>
      </q-toolbar>
    </q-header>
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import EssentialLink from 'components/EssentialLink.vue'

export default {
  name: 'MainLayout',
  components: { EssentialLink },
  data () {
    return {
      //
    }
  }
}
</script>
