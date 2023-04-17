<template>
    <div class="IndexPage">
        <nav class="navbar" role="navigation" aria-label="main navigation">
            <div class="navbar-brand">
                <a class="navbar-item" href="https://bulma.io">
                <img src="https://bulma.io/images/bulma-logo.png" width="112" height="28">
                </a>
            </div>
            <div class="navbar-end">
            <div class="navbar-item">
                <div class="dict-button-box" @click="toggleDictPad()">
                    
                    <button class="button dict-button">
                        <span class="icon">
                            <Icon icon="mingcute:book-4-fill" />
                        </span>
                        <span>提示词词典</span>
                    </button>
                </div>
            </div>
            </div>
        </nav>
        <PromptEditor />
        <section class="PromptDictPad" v-if="needDictPad" v-show="showDictPad">
            <div class="title">
                <Icon icon="mingcute:book-4-fill" />
                提示词词典
                <!--                <a class="github-dict" href="https://github.com/Moonvy/OpenPromptStudio" target="_blank">-->
                <!--                    <Icon icon="radix-icons:github-logo" />一起维护词典</a-->
                <!--                >-->
                <button class="icon close-button" @click="toggleDictPad(false)">
                    <Icon icon="radix-icons:cross-1" />
                </button>
            </div>
            <PromptDict />
        </section>
        <footer class="footer">
            <a href="https://github.com/Moonvy/OpenPromptStudio" target="_blank">
                OpenPromptStudio / v{{ version }} /
            </a>
            <a href="https://moonvy.com/?homepage"> made by </a>
        </footer>
    </div>
</template>
<script lang="ts">
import Vue, { PropType } from "vue"
import vPromptEditor from "../../Compoents/PromptEditor/PromptEditor.vue"
import vPromptDict from "../../Compoents/PromptDict/PromptDict.vue"

import pkg from "../../../package.json"
export default Vue.extend({
    data() {
        return {
            showDictPad: false,
            needDictPad: false,
            version: pkg.version,
        }
    },
    methods: {
        toggleDictPad(show?: boolean) {
            this.showDictPad = show ?? !this.showDictPad
            if (this.showDictPad) this.needDictPad = true
        },
    },
    components: {
        PromptEditor: vPromptEditor,
        PromptDict: vPromptDict,
    },
})
</script>
