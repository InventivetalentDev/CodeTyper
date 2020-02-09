<template>
    <div id="app">
        <md-app>
            <md-app-content>
                <TyperController v-if="mode==='full'||mode==='controller'" ref="controller" :mode="mode" @popout="popout" @globalSync="globalSync"/>
                <CodeTyper v-if="mode==='full'||mode==='typer'" ref="typer" :mode="mode" @popout="popout" @globalSync="globalSync"/>
            </md-app-content>
        </md-app>
    </div>
</template>

<script>
    import CodeTyper from "./components/CodeTyper";
    import TyperController from "./components/TyperController";

    export default {
        name: 'App',
        data() {
            return {
                mode: 'full'
            }
        },
        methods: {
            popout() {
                if (this.mode === "full") {
                    location.hash = "typer";
                    this.mode = "typer";

                    let u = new URL(location.href);
                    u.hash = "controller";
                    let win = window.open(u.toString(), 'CodeTyper_Controller', "width=900,height=170,left=150,top=200,toolbar=0,status=0");

                    let self = this;

                    let checkTimer;

                    checkTimer = setInterval(() => {
                        if (win.closed) {
                            self.mode = "full";
                            location.hash = "";
                            clearInterval(checkTimer);
                        }
                    }, 500);


                }
            },
            globalSync(source, k, v) {
              if (source === "typer") {
                this.$refs.controller.receiveGlobal(k, v);
              }
              if (source === "controller") {
                this.$refs.typer.receiveGlobal(k, v);
              }
            }
        },
        components: {
            TyperController,
            CodeTyper
        },
        mounted() {
            let hash = window.location.hash;
            if (hash === "#controller") {
                this.mode = "controller";
            } else if (hash === "#typer") {
                this.mode = "typer";
            }
        }
    }
</script>

<style>
    html, body {
        background: #252525;
    }
</style>
