<!-- Created on 2023/03/20 - 18:43 -->
<template>
    <div class="PromptWork" :class="{ isPNGExporting }" @click="onWorkClick">
        <div class="columns">
            <div class="AddArea Area column">
                <div class="field is-horizontal">
                    <div class="field-body level">
                        <div class="WorkInfoArea Area field has-addons level-left">
                            <div class="control">
                                <a class="button is-static">工作区名称</a>
                            </div>
                            <div class="WorkName control">
                                <input type="text" class="input" v-model="promptWork.data.name" />
                            </div>
                        </div>
                        <div class="field level-right">
                            <div class="control">
                                <button @click="doDeleteWorkspace()" v-tooltip="`删除工作区`" class="button">
                                    <span class="icon">
                                        <Icon icon="radix-icons:trash" />
                                    </span>
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="field">
                    <div class="control">
                        <textarea
                            class="textarea"
                            v-model="inputText"
                            placeholder="输入提示词"
                            rows="6"
                            @paste="onUserInput"
                            @input="onUserInput"
                            @keydown.enter="onUserInputDebounce"
                            @change="onUserInputDebounce"
                            spellcheck="false"
                        >
                        </textarea>
                    </div>
                </div>
                <article class="message">
                    <div class="message-body">
                        <div class="output">
                            <div class="pl" v-if="outputText == inputText">输出与输入相同</div>
                            <template v-else> {{ outputText }}</template>
                        </div>      
                    </div>
                </article>

                <div class="options">
                    <!--                <button @click="doExportPrompt">{{ "导出" }}</button>-->
                    <!--                <button @click="doImportByInput">{{ "导入" }}</button>-->
                    <div class="line level">
                        <div class="level-left">
                        <div class="level-item">
                            <button @click="copy(outputText)" title="复制" class="button is-primary">
                                <Icon icon="radix-icons:clipboard-copy" /> 复制
                            </button>
                        </div>
                        <div class="level-item">
                            <div class="level-item">
                                <div class="select">
                                    <select v-model="inputParser" class="parser-select" v-tooltip="`提示词语法类型`">
                                        <option value="midjourney">Midjourney</option>
                                        <option value="stable-diffusion-webui">stable-diffusion-webui</option>
                                    </select>
                                </div>
                            </div>
                            <button @click="doDisableAll()" v-tooltip="'全部禁用'" class="button level-item">
                                <span class="icon">
                                    <Icon icon="radix-icons:shadow-none" />
                                </span>
                            </button>
                            <button
                                @click="doSwitchIO()"
                                v-tooltip="'用输出替换输入'"
                                class="button level-item"
                                :class="{ disabled: outputText == inputText }"
                            >
                                <span class="icon">    
                                    <Icon icon="radix-icons:arrow-up" />
                                </span>
                            </button>
                            <button
                                @click="doClear()"
                                v-tooltip="'清空输入'"
                                class="button level-item"
                                :class="{ disabled: inputText?.length == '' }"
                            >
                                <span class="icon">    
                                    <Icon icon="radix-icons:crumpled-paper" />
                                </span>
                            </button>
                        </div>
                        </div>
                        <div class="level-right">
                            <div class="level-item">
                                <div class="field has-addons">
                                    <div class="control">
                                        <button @click="toPng" v-tooltip="`导出 PNG 图片`" class="button">
                                            <Icon icon="fluent:image-16-regular" />
                                        </button>
                                    </div>
                                    <div class="control">
                                        <button @click="toPng({ scale: 2 })" v-tooltip="`导出 PNG 图片（2X）`" class="button">
                                            <Icon icon="fluent:hd-24-regular" />
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="line more-options">

                        <!-- <button @click="doDeleteWorkspace()" v-tooltip="`删除工作区`" class="button">
                            <span class="icon">
                                <Icon icon="radix-icons:trash" />
                            </span>
                        </button> -->
                    </div>
                </div>
            </div>
            
            <div class="main column" ref="main">
                <div
                    class="PromptListArea Area"
                    :class="[promptWork.groups.length <= 1 ? 'noGroup' : 'hasGroup']"
                    @contextmenu.prevent
                >
                    <div class="PromptGroup" v-for="group in promptWork.groups">
                        <div class="PromptGroupTitle" v-if="promptWork.groups.length > 1">
                            <div class="name">
                                <input value="权重组" /> <span v-if="group.name">{{ group.name }}</span>
                                <span class="groupLv" v-if="group.groupLv && group.groupLv > 1">{{ group.groupLv }}</span>
                            </div>
                        </div>
                        <PromptList
                            v-for="promptList in group.lists"
                            :key="promptList.data.id"
                            :list="promptList"
                            @update="doExportPrompt()"
                            class="field"
                        >
                            <div class="name-bar label">
                                <div class="name" :class="[`type-${promptList.data.id}`]" :title="promptList.data.name">
                                    <span class="content">{{ promptList.data.name ?? promptList.data.id }}</span>
                                </div>
                            </div>
                            <div
                                class="list PromptList-list field is-grouped is-grouped-multiline"
                                @dblclick="
                                    onPromptListDblick($event, promptList, {
                                        group: group.id,
                                        lv: group.groupLv,
                                        subType: promptList.data.id,
                                    })
                                "
                            >
                                <PromptItem
                                    @click="onItemClick(item)"
                                    @update="onItemUpdate(item)"
                                    @contextmenu="doOpenItemMenu($event, promptList)"
                                    v-for="item in promptList.items"
                                    :item="item"
                                    :list="promptList"
                                    :key="item.data.word.id"
                                />
                            </div>
                        </PromptList>
                    </div>
                </div>
            </div>
        </div>
        <PromptMenu ref="menu"></PromptMenu>
    </div>
</template>
<script lang="ts">
import Vue, { PropType } from "vue"
import { PromptWork } from "../../Sub/PromptWork"
import vPromptItem from "../PromptItem/PromptItem.vue"
import vAddButton from "./Components/AddButton.vue"
import vPromptList from "../PromptList/PromptList.vue"
import debounce from "lodash/debounce"
import throttle from "lodash/throttle"
import { PromptItem } from "../../Sub/PromptItem"
import { useClipboard } from "@vueuse/core"
let { copy } = useClipboard()
import { copyBlobToClipboard } from "copy-image-clipboard"
import { toPng, toJpeg, toBlob, toPixelData, toSvg } from "html-to-image"
import { getImageSize } from "html-to-image/src/util"
// @ts-ignore
import download from "downloadjs"
import { PromptList } from "../../Sub/PromptList"
import vPromptMenu from "../PromptMenu/PromptMenu.vue"

export default Vue.extend({
    props: {
        promptWork: { type: Object as PropType<PromptWork>, required: true },
    },
    inject: ["PromptEditor"],
    data() {
        return {
            inputText: "",
            outputText: "",
            inputParser: this.promptWork?.data?.parser ?? "midjourney",
            isPNGExporting: false,
        }
    },
    created() {
        ;(<any>this).doImportByInputThrottle = debounce(
            () => {
                this.doImportByInput()
            },
            30,
            { maxWait: 50 }
        )
        ;(<any>this).doAddInputDebounce = debounce(() => {
            this.doImportByInputThrottle()
        }, 300)
        this.inputText = this.promptWork.data.initText ?? ""
        this.doImportByInputThrottle()
    },
    watch: {
        inputParser(val) {
            this.promptWork.data.parser = val
            this.doImportByInputThrottle()
        },
    },
    methods: {
        doImportByInputThrottle() {},
        async doImportByInput() {
            console.log("[doImportByInput]")
            await this.promptWork.importPrompts(this.inputText, { parser: <any>this.inputParser })
            this.doExportPrompt()
        },
        doExportPrompt() {
            let text = this.promptWork.exportPrompts()
            this.outputText = text
        },
        async onUserInput() {
            setTimeout(() => {
                ;(<any>this).doImportByInputThrottle()
            }, 0)
        },
        async onUserInputDebounce() {
            // console.log("[onUserInputDebounce]")
            ;(<any>this).doAddInputDebounce()
        },
        async doClear() {
            this.inputText = ""
            await this.doImportByInputThrottle()
            this.doExportPrompt()
        },
        doSwitchIO() {
            this.inputText = this.promptWork.exportPrompts()
        },
        doDisableAll() {
            this.promptWork.disableAll()
            this.doExportPrompt()
        },
        doDeleteWorkspace() {
            this.$emit("delete", this.promptWork)
        },
        // 提示词条目被点击
        onItemClick(item: PromptItem) {
            // console.log("onItemClick", item)
            item.data.disabled = !item.data.disabled
            this.doExportPrompt()
        },
        // 提示词条目内容更新
        async onItemUpdate(item: PromptItem) {
            this.doExportPrompt()
        },
        // 提示词列表空白处双击新建
        onPromptListDblick(e: any, promptList: PromptList, data: any) {
            if (e?.target?.classList?.contains("list")) {
                this.doAddNewByList(promptList, data)
            }
        },
        copy(text: string) {
            copy(text)
        },
        async toPng(options?: { scale: number }) {
            this.isPNGExporting = true
            let enablePngExportFixed = (<any>this).PromptEditor?.promptEditor?.data?.enablePngExportFixed
            let enablePngExportCopy = (<any>this).PromptEditor?.promptEditor?.data?.enablePngExportCopy
            try {
                let el = <any>this.$refs.main
                let { width, height } = getImageSize(el)

                let scale = options?.scale ?? 1
                if (enablePngExportFixed) width = 1240
                width = width * scale
                height = height * scale

                let re = await toBlob(el, {
                    width,
                    height,
                    style: { transform: `scale(${scale})`, transformOrigin: "top left" },
                })
                this.isPNGExporting = false

                if (enablePngExportCopy) {
                    copyBlobToClipboard(re!)
                } else {
                    download(re, `${this.promptWork.data.name}-OPS-Prompts_${width}x${height}.png`)
                }
            } catch (e) {
                console.error(e)
                this.isPNGExporting = false
            }
        },

        doAddNewByList(promptList: PromptList, data: any) {
            let item = promptList.pushPrompt("", data)
            item.state.isEdit = "text"
        },

        doOpenItemMenu(options: { item: PromptItem; el: any; event: any }, promptList: PromptList) {
            ;(this.$refs as any).menu.open({ ...options, promptList })
        },

        onWorkClick() {
            let els = document.body.querySelectorAll(".PromptWork")
            els.forEach((el) => el.classList.toggle("active", false))
            this.$el.classList.toggle("active", true)
        },
    },
    components: { PromptMenu: vPromptMenu, PromptItem: vPromptItem, AddButton: vAddButton, PromptList: vPromptList },
})
</script>
