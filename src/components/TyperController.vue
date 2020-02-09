<template>
    <div>
        <md-button v-if="mode==='full'" v-on:click="popout" style="float: right;" title="Popout"><i class="material-icons">open_in_new</i></md-button>
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
                    {{ index }}/{{ length }}
                    <!--                                        <input type="range" v-model="index" :max="length">-->
                    |
                    <md-input type="number" v-model="cps" max="60000"></md-input> <span title="Characters/Second">CPS</span>
                </div>
            </div>
            <div class="md-layout-item">
                <div>
                    <input type="file" @change="onFileChange">
                </div>
                <div>
                    <md-checkbox v-model="autoscroll">Auto-Scroll</md-checkbox>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: 'TyperController',
        components: {},
        props: {
            mode: String
        },
        data() {
            return {
                language: 'javascript',
                index: 0,
                length: 0,
                playing: false,
                interval: 1000,
                cps: 1,
                autoscroll: true
            }
        },
        watch: {
            language() {
                this.notifyGlobal("language", this.language);
            },
            index() {
                this.notifyGlobal("index", this.index);
            },
            // playing() {
            //     this.notifyGlobal("playing", this.playing);
            // },
            interval() {
                this.notifyGlobal("interval", this.interval);
            },
            cps() {
                this.interval = Math.max(1, Math.round(1000 / parseFloat(this.cps)));
                window.console.log("CPS: " + this.cps + " => Interval: " + this.interval);
                this.notifyGlobal("cps", this.cps);
            },
            autoscroll() {
                this.notifyGlobal("autoscroll", this.autoscroll);
            }
        },
        methods: {
            notifyGlobal(k, v) {
                if (this.mode === "full") {
                    this.$emit("globalSync", "controller", k, v);
                } else {
                    localStorage.setItem(k, v);
                }
            },
            receiveGlobal(k, v) {
                // window.console.log("[Typer] receiveGlobal " + k + ": " + v)
                let self = this;
                switch (k) {
                    case 'length':
                        self.length = parseInt(v);
                        break;
                    case 'language':
                        self.language = v;
                        break;
                    case 'playing':
                        self.playing = (v === "true" || v === true);
                        break;
                    case 'index':
                        self.index = parseInt(v);
                        break;
                    case 'interval':
                        self.interval = parseInt(v);
                        break;
                    case 'cps':
                        self.cps = parseInt(v);
                        break;
                }
            },
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
                if (this.playing) return;
                this.playing = true;
                this.notifyGlobal("playing", true);
            },
            pause() {
                if (!this.playing) return;
                this.playing = false;
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
                return true;
            },
            onFileChange(e) {
                window.console.log("onFileChange")
                let files = e.target.files || e.dataTransfer.files;
                if (!files.length)
                    return;
                this.handleFile(files[0]);
            },
            handleFile(file) {
                window.console.log("handleFile");
                let reader = new FileReader();
                let self = this;
                reader.onload = (e) => {
                    window.console.log("file loaded")
                    this.notifyGlobal("code", e.target.result);
                    self.length = e.target.result.length;
                };

                this.pause();
                this.index = 0;

                reader.readAsText(file);
            },
            popout() {
                this.$emit("popout");
            }
        },
        mounted() {
            for (let i = 0, len = localStorage.length; i < len; ++i) {
                this.receiveGlobal(localStorage.key(i), localStorage.getItem(localStorage.key(i)))
            }
            this.playing = false;

            let self = this;

            window.addEventListener('storage', event => {
                window.console.log("[Controller] onstorage " + event.key + ": " + event.newValue);
                self.receiveGlobal(event.key, event.newValue);
            });
        }
    }
</script>
