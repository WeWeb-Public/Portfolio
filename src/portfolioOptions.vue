<template>
    <div class="portfolio-options">
        <div class="content">
            <!-- COLUMNS PER LINE -->
            <div class="content-elem">
                <div class="title">{{wwLang.getText(lang.nbColumnsPerLine)}} :</div>
                <wwManagerInput type="number" color="blue" v-model="itemsPerLine.mobile" label="Mobile" v-on:change="updateResult"/>
                <wwManagerInput type="number" color="pink" v-model="itemsPerLine.tablet" label="Tablet" v-on:change="updateResult"/>
                <wwManagerInput type="number" color="green" v-model="itemsPerLine.smallScreen" label="Small screen" v-on:change="updateResult"/>
                <wwManagerInput type="number" color="orange" v-model="itemsPerLine.bigScreen" label="Big screen" v-on:change="updateResult"/>
            </div>
            <!-- PADDINGS -->
            <div class="content-elem">
                <div class="title">{{wwLang.getText(lang.paddings)}}</div>
                <span>{{paddings}} px</span>
                <wwManagerSlider type="ratio" v-model="paddings" v-on:change="updateResult"/>
            </div>
            <!-- ITEM LOADING -->
            <div class="content-elem loading-item">
                <div class="title">{{wwLang.getText(lang.itemLoading)}}</div>
                <wwManagerSelect class="option" :options="itemsLoadingOptions" :value="itemsLoading.type" v-model="itemsLoading.type" @change="updateResult($event)"></wwManagerSelect>
                <div v-show="itemsLoading.type === 'items'">
                    <wwManagerInput class="input" type="number" color="green" v-model="itemsLoading.itemNumber" :label="wwLang.getText(lang.nbItems)" v-on:change="updateResult"/>
                </div>
            </div>
            <!-- CATEGORIES -->
            <div class="content-elem">
                <div class="title">{{wwLang.getText(lang.categories)}}</div>
                <!-- IS ALL CATEGORIES -->
                <div class="all">
                    <wwManagerRadio color="blue" v-model="categories.isAll" />
                    <span>{{wwLang.getText(lang.isAllCategories)}}</span>
                </div>
                <!-- CATEGORY LIST -->
                <div v-for="(category, index) in categories.data" :key="index" class="elem">
                    <wwManagerInput type="text" color="green" v-model="category.name" :label="wwLang.getText(lang.uniqueName)" v-on:change="updateResult"/>
                    <span v-for="(value, key) in category.displayName" :key="key">
                        <wwManagerInput color="blue" v-model="category.displayName[key]" :label="key"/>
                    </span>
                    <!-- REMOVE CATEGORY -->
                    <wwManagerButton color="red" @click="deleteCategory(index)">&times;</wwManagerButton>
                </div>
                <!-- ADD CATEGORY -->
                <div class="add-elem">
                    <wwManagerButton color="blue" @click="addCategory()">{{wwLang.getText(lang.add)}}</wwManagerButton>
                </div>
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
    data() {
        return {
            wwLang: wwLib.wwLang,
            lang: lang,
            itemsPerLine: {
                mobile: 1,
                tablet: 2,
                smallScreen: 3,
                bigScreen: 4,
            },
            itemsLoading: {
                type: 'all',
                itemNumber: 10
            },
            categories: {
                isAll: true,
                data: [{
                    name: 'animals',
                    displayName: {
                        en: 'Animals',
                        fr: 'Animaux'
                    }
                }, {
                    name: 'buildings',
                    displayName: {
                        en: 'Buildings',
                        fr: 'Batiments'
                    }
                }, {
                    name: 'landscapes',
                    displayName: {
                        en: 'Landscapes',
                        fr: ''
                    }
                }]
            },
            itemsLoadingOptions: {
                type: 'text',
                values: [{
                    default: true,
                    value: 'all',
                    text: wwLib.wwLang.getText(lang.allItems)
                // }, {
                //     value: 'scroll',
                //     text: 'On scroll'
                }, {
                    value: 'items',
                    text: wwLib.wwLang.getText(lang.nbItems)
                }]
            },
            paddings: 0
        }
    },
    methods: {
        init () {
            this.itemsPerLine = this.options.data.itemsPerLine || this.itemsPerLine
            this.categories = this.options.data.categories || this.categories
            this.paddings = this.options.data.paddings || this.paddings
            this.itemsLoading = this.options.data.itemsLoading || this.itemLoading
            this.options.result.itemsPerLine = this.itemsPerLine
            this.options.result.categories = this.categories
            this.options.result.paddings = this.paddings
            this.options.result.itemsLoading = this.itemsLoading
        },
        updateResult () {
            this.options.result.itemsPerLine = this.itemsPerLine
            this.options.result.categories = this.categories
            this.options.result.paddings = this.paddings
            this.options.result.itemsLoading = this.itemsLoading
            this.options.result.itemsLoading.itemNumber = parseInt(this.itemsLoading.itemNumber)
        },
        addCategory () {
            this.categories.data.push({
                name: '',
                displayName: {
                    en: '',
                    fr: ''
                }
            })
            this.updateResult()
        },
        deleteCategory (index) {
            this.categories.data.splice(index, 1)
            this.updateResult()
        },
    },
    mounted () {
        this.init()
    }
}
</script>

<style lang="scss" scoped>
.portfolio-options {
    .content {
        display: flex;
        padding: 20px;
        flex-wrap: wrap;
        width: 100%;
        justify-content: center;
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
            width: 60%;
            margin-bottom: 25px;

            .all {
                display: flex;
                align-items: center;
                margin: 10px 0;
            }
            .elem {
                display: flex;
                align-items: center;
                justify-content: space-between;
                margin: 10px 0;
            }
            .add-elem {
                margin-top: 20px;
                display: flex;
                justify-content: center;
            }
        }
    }
}
.input {
    width: 100%;
}
</style>