<template>
    <div>
        <div class="md-layout">
            <div class="md-layout-item">
                <div>
                    <md-button v-on:click="goto(0)" :disabled="index===0" title="Go to First Frame"><i class="material-icons">fast_rewind</i></md-button>
                    <md-button v-on:click="goto(index-1)" :disabled="index===0" title="Go to Previous Frame"><i class="material-icons">skip_previous</i></md-button>
                    <md-button v-on:click="playOrPause()"><i class="material-icons" title="Play/Pause">{{ playing ? 'pause' : 'play_arrow' }}</i></md-button>
                    <md-button v-on:click="goto(index+1)" :disabled="index>=length" title="Go to Next Frame"><i class="material-icons">skip_next</i></md-button>
                    <md-button v-on:click="goto(length)" :disabled="index>=length" title="Go to Last Frame"><i class="material-icons">fast_forward</i></md-button>
                </div>
                <div>
                    {{ index }}/{{ length }}
                    <input type="range" v-model="index" :max="length">
                </div>
            </div>
            <div class="md-layout-item">
                <input type="file" @change="onFileChange">
            </div>
        </div>
        <div class="code-container">
            <prism :language=language>{{ currentCode }}</prism>
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

    import Prism from 'vue-prism-component'

    export default {
        name: 'CodeTyper',
        components: {
            Prism
        },
        props: {
            code: String,
            language: String
        },
        data() {
            return {
                index: 0,
                length: 0,
                playing: false,
                interval: 500,
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

            },
            pause() {

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

                this.playing = false;
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
