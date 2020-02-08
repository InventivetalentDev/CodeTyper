<template>
  <div id="app">
    <md-app>
      <md-app-content>
        <CodeTyper  ref="typer"/>
      </md-app-content>
    </md-app>
  </div>
</template>

<script>
import CodeTyper from "./components/CodeTyper";

export default {
  name: 'App',
  data() {
    return {
    }
  },
  methods:{
  },
  components: {
    CodeTyper
  }
}
</script>

<style>
  html,body{
    background: #252525;
  }
</style>
