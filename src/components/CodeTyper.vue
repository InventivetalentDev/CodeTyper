<template>
    <div>
        <div>
            <button v-on:click="step()" :disabled="index>=length">Step</button>
            <input type="range" v-model="index" :max="length">
        </div>
        <prism :language=language>{{ currentCode }}</prism>
    </div>
</template>

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
            goto(i) {
                if (i >= this.length) {
                    return false;
                }
                this.index = i;
                return true;
            },
            showCurrentCode() {
                this.currentCode = this.code.substring(0, this.index);
            }
        },
        mounted() {
            this.length = this.code.length;
        }
    }
</script>
