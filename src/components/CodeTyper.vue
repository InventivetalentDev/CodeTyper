<template>
    <div>
        <!--        <div class="md-layout">-->
        <!--            <div class="md-layout-item">-->
        <!--                <div>-->
        <!--                    <md-button v-on:click="goto(0)" :disabled="index===0||playing" title="Go to First Frame"><i class="material-icons">fast_rewind</i></md-button>-->
        <!--                    <md-button v-on:click="goto(index-1)" :disabled="index===0||playing" title="Go to Previous Frame"><i class="material-icons">skip_previous</i></md-button>-->
        <!--                    <md-button v-on:click="playOrPause()"><i class="material-icons" title="Play/Pause">{{ playing ? 'pause' : 'play_arrow' }}</i></md-button>-->
        <!--                    <md-button v-on:click="goto(index+1)" :disabled="index>=length||playing" title="Go to Next Frame"><i class="material-icons">skip_next</i></md-button>-->
        <!--                    <md-button v-on:click="goto(length)" :disabled="index>=length||playing" title="Go to Last Frame"><i class="material-icons">fast_forward</i></md-button>-->
        <!--                </div>-->
        <!--                <div>-->
        <!--&lt;!&ndash;                    {{ index }}/{{ length }}&ndash;&gt;-->
        <!--&lt;!&ndash;                    <input type="range" v-model="index" :max="length">&ndash;&gt;-->
        <!--&lt;!&ndash;                    |&ndash;&gt;-->
        <!--                    <input type="number" v-model="cps" max="60000"><span title="Characters/Second">CPS</span>-->
        <!--                </div>-->
        <!--            </div>-->
        <!--            <div class="md-layout-item">-->
        <!--                <input type="file" @change="onFileChange">-->
        <!--            </div>-->
        <!--        </div>-->
        <md-content class="code-container md-scrollbar" id="codeContainer" v-bind:style="{'height':codeHeight+'px'}">
            <PrismComponent :language=language>{{ currentCode }}</PrismComponent>
        </md-content>
    </div>
</template>

<style>
    .code-container {
        overflow-y: auto;
        white-space: pre-wrap;
    }
</style>

<script>
    import 'prismjs'
    import 'prismjs/themes/prism-tomorrow.css'

    import PrismComponent from 'vue-prism-component'


    export default {
        name: 'CodeTyper',
        components: {
            PrismComponent
        },
        props: {
            mode: String
        },
        data() {
            return {
                code: '',
                language: 'javascript',
                index: 0,
                length: 0,
                playing: false,
                interval: 1000,
                cps: 1,
                timer: null,
                currentCode: "",
                autoscroll: true,
                codeHeight: 1000
            }
        },
        watch: {
            mode(){
                this.applyContainerSize();
            },
            code() {
                window.console.log("code change")
                this.length = this.code.length;
                this.index = 0;
            },
            index() {
                this.showCurrentCode()
            },
            length() {
                this.notifyGlobal("length", this.length);
            },
            cps() {
                this.interval = Math.max(1, Math.round(1000 / parseFloat(this.cps)));
                window.console.log("CPS: " + this.cps + " => Interval: " + this.interval);
            }
        },
        methods: {
            notifyGlobal(k, v) {
                if (this.mode === "full") {
                    this.$emit("globalSync", "typer", k, v);
                } else {
                    localStorage.setItem(k, v);
                }
            },
            receiveGlobal(k, v) {
                // window.console.log("[Typer] receiveGlobal "+k+": "+v)
                let self = this;
                switch (k) {
                    case 'length':
                        self.length = parseInt(v);
                        break;
                    case 'language':
                        self.language = v;
                        break;
                    case 'playing':
                        if (v === "true"||v===true) {
                            self.play();
                        } else {
                            self.pause();
                        }
                        break;
                    case 'index':
                        self.index = parseInt(v);
                        break;
                    case 'code':
                        self.code = v;
                        break;
                    case 'interval':
                        self.interval = parseInt(v);
                        break;
                    case 'autoscroll':
                        self.autoscroll = v==="true"||v===true;
                        break;
                }
            },
            step() {
                let b = this.goto(this.index + 1);
                if (!b) {
                    this.pause()
                }
                return b;
            },
            playOrPause() {
                if (this.playing) {
                    this.pause();
                } else {
                    this.play();
                }
            },
            play() {
                if (this.playing) return;
                this.timer = setInterval(this.step, this.interval);
                this.playing = true;
                this.notifyGlobal("index", this.index);
                this.notifyGlobal("playing", true);
            },
            pause() {
                if (!this.playing) return;
                clearInterval(this.timer);
                this.playing = false;
                this.notifyGlobal("index", this.index);
                this.notifyGlobal("playing", false);
            },
            goto(i) {
                if (i > this.length) {
                    return false;
                }
                if (i < 0) {
                    return false;
                }
                this.index = i;

                if (this.autoscroll) {
                    let el = document.getElementById("codeContainer");
                    el.scrollTop = el.scrollHeight;
                }

                return true;
            },
            showCurrentCode() {
                this.currentCode = this.code.substring(0, this.index);
            },
            applyContainerSize() {
                let innerHeight = window.innerHeight;
                if (this.mode === "typer") {
                    innerHeight-=32;
                }else if (this.mode === "full") {
                    innerHeight-=106;
                }
                window.console.log(innerHeight);
                this.codeHeight =innerHeight;
            }
        },
        mounted() {
            let self = this;

            for (let i = 0, len = localStorage.length; i < len; ++i) {
                this.receiveGlobal(localStorage.key(i), localStorage.getItem(localStorage.key(i)))
            }
            this.index=0;
            this.pause();

            this.applyContainerSize();
            window.addEventListener("resize",()=>{
                self.applyContainerSize();
            });


            window.addEventListener('storage', event => {
                window.console.log("[Typer] onstorage " + event.key + ": " + event.newValue);
                self.receiveGlobal(event.key, event.newValue);
            });
        }
    }
</script>
