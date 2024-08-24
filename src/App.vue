<template>
  <div id="app">
    <router-view />
    <back-to-top :visibility-height="100" :back-position="0" transition-name="fade" ref="backTop"/>
    <ChatWindow />
  </div>

</template>

<script>

import BackToTop from '@/components/BackToTop'
import ChatWindow from '@/components/ChatWindow'

export default {
  name: 'App',
  components: {
    BackToTop,
    ChatWindow
  }
}
</script>
