<template>
    <div class="item-options">
        <div class="content">
            <!-- CATEGORIES -->
            <div class="content-elem tags">
                <div class="title">{{wwLang.getText(lang.categories)}}</div>
                <div v-for="tag in tags.data" :key="tag" class="tag">
                    <wwManagerRadio :value="isTag(tag)" @change="changeTag(tag)"/>
                    <span class="tag-name">{{wwLang.getText(tag.displayName)}}</span>
                </div>
            </div>
            <!-- PRIO -->
            <div class="content-elem prio">
                <div class="title">{{wwLang.getText(lang.priority)}}</div>
                <span>{{wwLang.getText(lang.priorityMessage)}}</span>
                <wwManagerInput class="input" type="number" color="green" v-model="prio" label="Prio" v-on:change="updateResult"/>
            </div>
        </div>
    </div>
</template>

<script> 
import lang from './lang.json'

export default {
    name: "portfolioOptions",
    props: {
        options: {
            type: Object,
            default () {
                return {}
            }
        },
    },
    data () {
        return {
            wwLang: wwLib.wwLang,
            lang: lang,
            item: {},
            tags: {},
            prio: ''
        }
    },
    methods: {
        init () {
            this.item = this.options.data.item
            this.tags = this.options.data.tags
            this.prio = this.item.prio
            this.options.result.prio = this.prio
            this.options.result.tags = this.item.tags
        },
        updateResult () {
            this.options.result.prio = this.prio
            this.options.result.tags = this.item.tags
        },
        isTag (tag) {
            return this.item.tags.indexOf(tag.name) !== -1
        },
        changeTag (tag) {
            if (this.isTag(tag)) {
                const index = this.item.tags.indexOf(tag.name)
                this.item.tags.splice(index, 1)
            } else {
                this.item.tags.push(tag.name)
            }
            this.updateResult()
        }
    },
    mounted () {
        this.init()
    }
}
</script>

<style lang="scss" scoped>
.item-options {
    display: inline-flex;
    justify-content: center;
    align-items: flex-start;
    .content {
        display: flex;
        padding: 20px;
        flex-wrap: wrap;
        width: 30%;
        font-family: "Monserrat", sans-serif;
        .title {
            color: #e02a4d;
            font-family: "Monserrat", sans-serif;
            font-size: 1.2rem;
            text-transform: uppercase;
            font-weight: bold;
            margin-bottom: 10px;
        }
        .content-elem {
            width: 100%;
            margin-bottom: 25px;
            .tag {
                display: flex;
                align-items: center;
                margin: 10px 0;
                .tag-name {
                    padding: 0 10px;
                    text-transform: capitalize;
                }
            }
            .input {
                width: 100%;
            }
        }
    }
}
</style>