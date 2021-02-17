<template>
  <q-layout view="lHh Lpr lFf">
    <q-header class="bg-white text-grey-10" bordered>
      <q-toolbar class="constrain">
        <q-btn
          class="large-screen-only q-mr-sm"
          to="/camera"
          icon="eva-camera-outline"
          flat
          size="18px"
          dense
          round/>
  <q-separator class="large-screen-only" vertical spaced/>
        <q-toolbar-title class="text-grand-hotel text-bold">
          FriendlyGram App
        </q-toolbar-title>
        <q-btn
          class="large-screen-only"
          to="/"
          icon="eva-home-outline"
          flat
          size="18px"
          dense
          round/>
      </q-toolbar>
    </q-header>

    <q-footer class="bg-white" bordered>
      <div v-if="showAppInstalled"
        class="banner-container bg-grey-4">
        <div class="constrain">
          <q-banner inline-actions dense rounded class="bg-blue-9 text-white">
            <b>Install FriendlyGram?</b>
            <template v-slot:action>
              <q-btn flat label="Yes" class="q-px-sm" />
              <q-btn flat label="Later" class="q-px-sm" />
              <q-btn flat label="Never" class="q-px-sm" />
            </template>
          </q-banner>

        </div>
      </div>



      <q-tabs class="text-grey-10 small-screen-only" active-color="primary" indicator-color="transparent">
        <q-route-tab to="/" icon="eva-home-outline"/>
        <q-route-tab to="/camera" icon="eva-camera-outline"/>
      </q-tabs>
    </q-footer>

    <q-page-container class="bg-grey-2">
      <router-view/>
    </q-page-container>
  </q-layout>
</template>

<script>
let deferredPrompt;

export default {
  name: 'MainLayout',
  data() {
    return {
      showAppInstalled: false
    }
  },
  mounted() {

    window.addEventListener('beforeinstallprompt', (e) => {
      // Prevent the mini-infobar from appearing on mobile
      e.preventDefault();
      // Stash the event so it can be triggered later.
      deferredPrompt = e;
      // Update UI notify the user they can install the PWA
      this.showAppInstalled = true
    });
  }
}
</script>
<style lang="scss">
.q-toolbar{
  @media (min-width: $breakpoint-sm-min) {
    height: 77px;
  }
}
.q-toolbar__title {
  font-size: 30px;

  @media (max-width: $breakpoint-xs-max){
    text-align: center;
  };
}

.q-footer {
  .q-tab__icon {
    font-size: 30px;
  }
}
</style>
