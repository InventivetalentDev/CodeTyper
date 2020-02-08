<template>
    <div>
        <div class="md-layout">
            <div class="md-layout-item">
                <div>
                    <md-button v-on:click="goto(0)" :disabled="index===0||playing" title="Go to First Frame"><i class="material-icons">fast_rewind</i></md-button>
                    <md-button v-on:click="goto(index-1)" :disabled="index===0||playing" title="Go to Previous Frame"><i class="material-icons">skip_previous</i></md-button>
                    <md-button v-on:click="playOrPause()"><i class="material-icons" title="Play/Pause">{{ playing ? 'pause' : 'play_arrow' }}</i></md-button>
                    <md-button v-on:click="goto(index+1)" :disabled="index>=length||playing" title="Go to Next Frame"><i class="material-icons">skip_next</i></md-button>
                    <md-button v-on:click="goto(length)" :disabled="index>=length||playing" title="Go to Last Frame"><i class="material-icons">fast_forward</i></md-button>
                </div>
                <div>
<!--                    {{ index }}/{{ length }}-->
<!--                    <input type="range" v-model="index" :max="length">-->
<!--                    |-->
                    <input type="number" v-model="cps" max="60000"><span title="Characters/Second">CPS</span>
                </div>
            </div>
            <div class="md-layout-item">
                <input type="file" @change="onFileChange">
            </div>
        </div>
        <div class="code-container">
            <PrismComponent :language=language>{{ currentCode }}</PrismComponent>
        </div>
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
        props: {},
        data() {
            return {
                code: '',
                language: 'javascript',
                index: 0,
                length: 0,
                playing: false,
                interval: 1000,
                cps:1,
                timer:null,
                currentCode: ""
            }
        },
        watch: {
            code() {
                window.console.log("code change")
                this.length = this.code.length;
            },
            index() {
                this.showCurrentCode()
            },
            cps(){
                this.interval = Math.max(1, Math.round(1000/parseFloat(this.cps)));
                window.console.log("CPS: " + this.cps + " => Interval: " + this.interval);
            }
        },
        methods: {
            step() {
                return this.goto(this.index + 1);
            },
            playOrPause() {
                if (this.playing) {
                    this.pause();
                } else {
                    this.play();
                }
            },
            play() {
                if(this.playing)return;
                this.timer = setInterval(this.step, this.interval);
                this.playing=true;
            },
            pause() {
                if(!this.playing)return;
                clearInterval(this.timer);
                this.playing= false;
            },
            goto(i) {
                if (i > this.length) {
                    return false;
                }
                if (i < 0) {
                    return false;
                }
                this.index = i;
                return true;
            },
            showCurrentCode() {
                this.currentCode = this.code.substring(0, this.index);
            },
            onFileChange(e) {
                let files = e.target.files || e.dataTransfer.files;
                if (!files.length)
                    return;
                this.handleFile(files[0]);
            },
            handleFile(file) {
                let reader = new FileReader();
                let self = this;
                reader.onload = (e) => {
                    self.code = e.target.result;
                };

                this.pause();
                this.index = 0;
                this.code = '';

                reader.readAsText(file);
            }
        },
        mounted() {
            this.length = this.code.length;
        }
    }
</script>
