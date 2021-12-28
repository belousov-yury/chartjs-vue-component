<template>
  <button @click="changeTheme">Сменить тему</button>
  <LineChart class="test" :dark-theme="isDark"/>
</template>

<script>
import LineChart from '@/components/LineChart'
export default {
  name: 'App',
  components: {
    LineChart
  },
  data () {
    return {
      isDark: false
    }
  },
  methods: {
    changeTheme () {
      this.isDark = !this.isDark
    }
  }

}
</script>

<style>
#app {
  font-family: Sans-Serif;
}
.test {
}
</style>
