<template>
  <div id="app">
    <CodeTyper code="let test = '1';" language="javascript"  ref="typer"/>
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
