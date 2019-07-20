<template>
  <div id="app">
    <sticky parent="#app"
            :top="0">
      <sticky-menu :top="0"
                   :menus="menus"
                   :data="[{prop: '.p'},{prop: '.div'}]"
                   parent="#app"></sticky-menu>
    </sticky>
    <div style="height:800px">
      <p class="p"
         style="margin-top:30px">p content
      </p>
      <div class='div'
           style="margin-top:300px">div content</div>
    </div>
  </div>
</template>

<script>
import Sticky from './components/Sticky.vue'
import StickyMenu from './components/StickyMenu.vue'

export default {
  name: 'app',
  data () {
    return {
      menus: [{
        label: 'div',
        value: '.div'
      }, {
        label: 'p',
        value: '.p'
      }]
    }
  },
  components: {
    Sticky,
    StickyMenu
  }
}
</script>

<style lang="less">
#app {
  height: 400px;
  overflow: scroll;
}
</style>
