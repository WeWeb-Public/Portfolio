<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div class="portfolio">
        <!-- wwManager:start -->
        <wwSectionEditMenu :sectionCtrl="sectionCtrl" :options="openOptions"></wwSectionEditMenu>
        <!-- wwManager:end -->
        <!-- BACKGROUND -->
        <wwObject class="background" :ww-object="section.data.background" ww-category="background"></wwObject>
        <div class="content">
            <!-- TOP WWOBJECTS -->
            <wwLayoutColumn tag="div" :ww-list="section.data.topWwObjs" class="top-wwobjs" @ww-add="add(section.data.topWwObjs, $event)" @ww-remove="remove(section.data.topWwObjs, $event)">
                <wwObject v-for="topWwObj in section.data.topWwObjs" :key="topWwObj.uniqueId" :ww-object="topWwObj"></wwObject>
            </wwLayoutColumn>
            <!-- wwManager:start -->
            <!-- RATIO PADDINGS -->
            <div v-if="editMode" class="top-ratio-slider">
                <span>{{section.data.paddings}}px</span>
                <wwManagerSlider type="ratio" v-model="section.data.paddings" v-on:change="sectionCtrl.update(section)"/>
            </div>
            <!-- wwManager:end -->
            <!-- CATEGORIES -->
            <div class="categories">
                <div class="category" @click="selectedCategory = 'all'" v-if="editMode || categories.isAll">
                    <span class="name">{{ wwLang.getText(langTextAll) }}</span>
                </div>
                <div class="category" v-for="category in categories.data" :key="category.name" @click="selectedCategory = category.name">
                    <span class="name">{{ wwLang.getText(category.displayName) }}</span>
                </div>
            </div>
            <!-- ITEMS LIST -->
            <div class="items-container" :style="{'min-height': itemsHeight + 'px'}">
                <div class="items">
                    <div v-for="(item,index) in section.data.items" :key="index" :data-item="index"
                        :ref="`item-${index}`"
                        class="item" :class="[`item-${index}`, { 'hide': !item.show }]"
                        :style="[getPosition(index), itemStyle]">
                        <!-- wwManager:start -->
                        <wwContextMenu class="ww-orange-button" tag="div"
                            :options="portefolioOptions" 
                            @duplicateItemToStart="duplicateItemToStart(index)"
                            @duplicateItemToEnd="duplicateItemToEnd(index)"
                            @openItemOptions="openItemOptions(index)"
                            @removeItem="removeItem(index)"
                            :style="{ top: `${section.data.paddings}px`, left: `${section.data.paddings}px` }" v-if="editMode">
                            <wwOrangeButton ></wwOrangeButton>
                        </wwContextMenu>
                        <!-- wwManager:end -->
                        <wwLayoutColumn tag="div" :ww-list="item.data" @ww-add="add(item.data, $event)" @ww-remove="remove(item.data, $event)">
                            <wwObject v-for="wwo in item.data" :key="wwo.uniqueId" :ww-object="wwo"></wwObject>
                        </wwLayoutColumn>
                    </div>
                </div>
            </div>
            <!-- MORE ITEMS BUTTON -->
            <div class="items-more-btn" @click="moreItems" v-if="section.data.itemsLoading.type === 'items' && maxItems < itemsLength">
                <wwObject :ww-object="section.data.moreItemsBtn"></wwObject>
            </div>
        </div>
    </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->
<script>
/* wwManager:start */
import portfolioOptions from './portfolioOptions.vue'
import portfolioItemOptions from './portfolioItemOptions.vue'

wwLib.wwPopups.addPopup('portfolioOptions', portfolioOptions)
wwLib.wwPopups.addPopup('portfolioItemOptions', portfolioItemOptions)

wwLib.wwPopups.addStory('PORTFOLIO_OPTIONS', {
    title: {
        en: 'Portfolio options',
        fr: 'Options du portfolio'
    },
    type: 'portfolioOptions',
    buttons: {
        FINISH: {
            text: {
                en: 'Finish',
                fr: 'Terminer'
            },
            next: false
        }
    }
})

wwLib.wwPopups.addStory('PORTFOLIO_ITEM_OPTIONS', {
    title: {
        en: 'Portfolio item options',
        fr: 'Options d\'élément du portfolio'
    },
    type: 'portfolioItemOptions',
    buttons: {
        FINISH: {
            text: {
                en: 'Finish',
                fr: 'Terminer'
            },
            next: false
        }
    }
})
/* wwManager:end */

export default {
    name: "__COMPONENT_NAME__",
    props: {
        // The section controller object is passed to you.
        sectionCtrl: Object
    },
    data() {
        return {
            // LANG
            wwLang: wwLib.wwLang,
            langTextAll: { en: 'All', fr: 'Tous' },
            portefolioOptions: {
               name: {  //Nom du popup, si vide le popup s'appelle 'Menu'
                   en: 'Item',
                   fr: 'Élement'
               },
               items: [  //Liste des options dans le popup
                   {
                       text: {
                           en: 'Duplicate to start',
                           fr: 'Dupliquer au début'
                       },
                       icon: 'wwi wwi-paste',
                       action: 'duplicateItemToStart'
                   },
                   {
                       text: {
                           en: 'Duplicate to end',
                           fr: 'Dupliquer à la fin'
                       },
                       icon: 'wwi wwi-paste',
                       action: 'duplicateItemToEnd'
                   },
                   {
                       text: {
                           en: 'Delete',
                           fr: 'Supprimer'
                       },
                       icon: 'wwi wwi-delete',
                       action: 'removeItem'
                   },
                   {
                       text: {
                           en: 'Options',
                           fr: 'Options'
                       },
                       icon: 'wwi wwi-edit-other',
                       action: 'openItemOptions'
                   }
               ]
            },
            // SELECTED ELEMS
            selectedCategory: 'all',
            selectedItemIdx: undefined,
            // ITEMS
            items: [],
            itemsPerLine: 3,
            itemsHeight: 0,
            maxItems: 20,
            // POSITIONS
            containerWidth: 0,
            columnsHeight: [],
            calculatePosTimeout: null,
            positions: [],
            ratios: [],
            // CATEGORIES
            categories: {},
            defaultCategories: {
                isAll: true,
                data: [
                {
                    name: 'animals',
                    displayName: {
                        en: 'Animals',
                        fr: 'Animaux'
                    }
                },
                {
                    name: 'buildings',
                    displayName: {
                        en: 'Buildings',
                        fr: 'Batiments'
                    }
                },
                {
                    name: 'landscapes',
                    displayName: {
                        en: 'Landscapes',
                        fr: 'Landscapes'
                    }
                }]
            }
        }
    },
    computed: {
        //Get the section object here !
        // It contains all the info and data about the section
        // Use it has you like !
        section () {
            return this.sectionCtrl.get();
        },
        editMode () {
            return this.sectionCtrl.getEditMode()
        },
        itemStyle () {
            return {
                padding: `${this.section.data.paddings}px`,
                width: `${100 / this.itemsPerLine}%`
            }
        },
        itemsLength () {
            return this.section.data.items.map(item => item.show).length
        }
    },
    watch: {
        selectedCategory () {
            this.filterItems();
        },
        maxItems () {
            this.filterItems();
        }
    },
    methods: {
        initData () {
            let needUpdate = false;

            //Initialize section data
            this.section.data = this.section.data || {};

            if (!this.section.data.topWwObjs) {
                this.section.data.topWwObjs = [];
                needUpdate = true;
            }
            if (!this.section.data.background) {
                this.section.data.background = wwLib.wwObject.getDefault({ type: 'ww-color' })
                needUpdate = true;
            }
            if (!this.section.data.moreItemsBtn) {
                this.section.data.moreItemsBtn = wwLib.wwObject.getDefault({ type: 'ww-button' })
                needUpdate = true;
            }
            if (!this.section.data.itemsPerLine) {
                this.section.data.itemsPerLine = { mobile: 1, tablet: 2, smallScreen: 3, bigScreen: 4 }
                needUpdate = true;
            }
            if (!this.section.data.categories) {
                this.section.data.categories = this.defaultCategories
                needUpdate = true;
            }
            if (!this.section.data.paddings) {
                this.section.data.paddings = 10
                needUpdate = true;
            }
            if (!this.section.data.itemsLoading) {
                this.section.data.itemsLoading = { type: 'all', itemNumber: 20 }
                needUpdate = true;
            }
            
            this.categories = this.section.data.categories
            this.maxItems = this.section.data.itemsLoading.itemNumber
            this.selectedCategory = (this.section.data.categories.isAll) ? 'all' : this.section.data.categories.data[0].name
            
            if (!this.section.data.items) {
                this.section.data.items = []
                for (let i = 0; i < 20; i++) {
                    const tag1 = this.categories.data[Math.ceil(Math.random() * (this.categories.data.length)) - 1].name;

                    let url
                    switch (tag1) {
                        case 'animals':
                            url = 'https://wewebapp.s3.eu-west-3.amazonaws.com/designs/36/sections/fyI72yGW5pW1PFQw0yhEgTfeAqS7Us17.jpg';
                            break;
                        case 'buildings':
                            url = 'https://wewebapp.s3.eu-west-3.amazonaws.com/designs/36/sections/E8XWzommGohbNpA70Wxua24uDwyps84R.jpg';
                            break;
                        case 'landscapes':
                            url = 'https://wewebapp.s3.eu-west-3.amazonaws.com/designs/36/sections/E6oWIaGBTlTxFPzsAh8VUH8d4A2QKjZN.jpg';
                            break;

                        default:
                            break;
                    }

                    this.section.data.items.push({
                        tags: wwLib.wwUtils.getUniqueId() % 2 === 1 ? [tag1] : [],
                        category: 'toadopt',
                        show: true,
                        prio: Math.random(),
                        data: [
                            wwLib.wwObject.getDefault({
                                type: 'ww-image',
                                data: {
                                    url: url
                                }
                            })
                        ],
                        mainImage: wwLib.wwObject.getDefault({
                                type: 'ww-image',
                                data: {
                                    url: url
                                }
                            }),
                        blocks: []
                    })
                }
                needUpdate = true;
            }

            if (needUpdate) {
                this.sectionCtrl.update(this.section);
            }

            this.filterItems();
        },
        init () {

            this.$nextTick(this.onResize)

            window.addEventListener('resize', this.onResize);
        },
        onResize () {
            this.containerWidth = this.$el.querySelector('.items').getBoundingClientRect().width;

            if (window.innerWidth < 768) {
                this.itemsPerLine = this.section.data.itemsPerLine.mobile || 1;
            } else if (window.innerWidth < 992) {
                this.itemsPerLine = this.section.data.itemsPerLine.tablet || 2;
            } else if (window.innerWidth < 1224) {
                this.itemsPerLine = this.section.data.itemsPerLine.smallScreen || 3;
            } else {
                this.itemsPerLine = this.section.data.itemsPerLine.bigScreen || 4;
            }

            this.calculatePos();
        },
        calculatePos () {
            if (!this.$el) return
            this.itemsHeight = 0;
            this.columnsHeight = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
            this.positions = [];

            for (let i = 0; i < this.section.data.items.length; i++) {
                this.positions[i] = {
                    top: '0px',
                    left: '0px'
                }
                if (!this.section.data.items[i].show) {
                    continue;
                }
                let col = 0;
                const rect = this.$el.querySelector('[data-item="' + i + '"').getBoundingClientRect();

                let minColHeight = 10000000000;
                for (let j = 0; j < this.itemsPerLine; j++) {
                    if (this.columnsHeight[j] < minColHeight - 150/*- rect.height / 2 */) {
                        minColHeight = this.columnsHeight[j];
                        col = j;
                    }
                }

                this.positions[i] = {
                    top: (this.columnsHeight[col]) + 'px',
                    left: (this.containerWidth / this.itemsPerLine * col) + 'px'
                }
                this.columnsHeight[col] += rect.height;

                if (col == 0) {
                    this.itemsHeight += this.containerWidth / this.itemsPerLine / 1.5;
                }
            }
            let maxColHeight = 0;
            for (let j = 0; j < this.itemsPerLine; j++) {
                if (this.columnsHeight[j] > maxColHeight) {
                    maxColHeight = this.columnsHeight[j];
                }
            }
            this.itemsHeight = maxColHeight + 10
            this.recalculatePos();
        },
        recalculatePos () {
            this.ratios = [];
            clearTimeout(this.calculatePosTimeout);
            this.calculatePosTimeout = setTimeout(this.calculatePos, 100);
        },
        getPosition (index) {
            return this.positions[index] || {}
        },
        /*=============================================m_ÔÔ_m=============================================\
          MANAGE ITEMS
        \================================================================================================*/
        getMaxPrio () {
           const itemMaxPrio = this.section.data.items.reduce((a , b) => a.prio > b.prio ? a : b)
           return itemMaxPrio.prio
        },
        getMinPrio () {
           const itemMinPrio = this.section.data.items.reduce((a , b) => a.prio < b.prio ? a : b)
           return itemMinPrio.prio
        },
        moreItems () {
            this.maxItems += this.section.data.itemsLoading.itemNumber
            this.filterItems()
        },
        filterItems () {
            // CATEGORIES
            if (this.selectedCategory === 'all') {
                for (let item of this.section.data.items) {
                    item.show = true;
                }
            }
            else {
                for (let item of this.section.data.items) {
                    item.show = item.tags.indexOf(this.selectedCategory) !== -1;
                }
            }

            // MAX ITEMS
            if (this.section.data.itemsLoading.type === 'scroll') {
                // console.log('scroll')
            } else if (this.section.data.itemsLoading.type === 'items') {
                let i = 0
                for (const elem of this.section.data.items) {
                    if (elem.show) { ++i }
                    if (i > this.maxItems) { elem.show = false }
                }
            }

            // SORT ITEMS PRIO
            this.section.data.items.sort(function (i1, i2) {
                return i1.prio < i2.prio ? 1 : -1;
            })

            this.ratios = [];
            this.$nextTick(this.calculatePos);
            this.$forceUpdate();
        },
        /* wwManager:start */
        duplicateItemToStart (index) {
            const item = JSON.parse(JSON.stringify(this.section.data.items[index]))
            wwLib.wwUtils.changeUniqueIds(item.data)
            item.prio = this.getMaxPrio() + 1
            this.section.data.items.unshift(item)

            this.sectionCtrl.update(this.section);

            this.filterItems();
            this.recalculatePos();
        },
        duplicateItemToEnd (index) {
            const item = JSON.parse(JSON.stringify(this.section.data.items[index]))
            wwLib.wwUtils.changeUniqueIds(item.data)
            item.prio = this.getMinPrio() - 1 
            this.section.data.items.push(item)

            this.sectionCtrl.update(this.section);

            this.filterItems();
            this.recalculatePos();
        },
        removeItem (index) {
            this.section.data.items.splice(index, 1);
            this.sectionCtrl.update(this.section);

            this.recalculatePos();
        },
        /* wwManager:end */
        /*=============================================m_ÔÔ_m=============================================\
          TOP WWBJECTS
        \================================================================================================*/
        /* wwManager:start */
        add (list, options) {
            list.splice(options.index, 0, options.wwObject);

            this.sectionCtrl.update(this.section);

            this.recalculatePos();
        },
        remove (list, options) {
            list.splice(options.index, 1);

            this.sectionCtrl.update(this.section);

            this.recalculatePos();
        },
        /* wwManager:end */
        /*=============================================m_ÔÔ_m=============================================\
          POPUP
        \================================================================================================*/
        /* wwManager:start */
        async openOptions() {
            try {
                let options = {
                    firstPage: 'PORTFOLIO_OPTIONS',
                    data: {
                        itemsPerLine: this.section.data.itemsPerLine,
                        categories: this.section.data.categories,
                        paddings: this.section.data.paddings,
                        itemsLoading: this.section.data.itemsLoading
                    },
                }
                const result = await wwLib.wwPopups.open(options)
                if (result.itemsPerLine) {
                    this.section.data.itemsPerLine = result.itemsPerLine;
                    this.sectionCtrl.update(this.section);
                    this.onResize()
                }
                if (result.categories) {
                    this.section.data.categories = result.categories;
                    this.categories = result.categories;
                    this.sectionCtrl.update(this.section);
                }
                if (result.paddings) {
                    this.section.data.paddings = result.paddings;
                    this.sectionCtrl.update(this.section);
                }
                if (result.itemsLoading) {
                    this.section.data.itemsLoading = result.itemsLoading
                    if (this.section.data.itemsLoading.type === 'items') {
                        this.maxItems = this.section.data.itemsLoading.itemNumber
                    }
                    this.sectionCtrl.update(this.section)
                    this.filterItems()
                    this.recalculatePos()
                }
                console.log(result);
            } catch (err) {
            }
        },
        async openItemOptions(index) {
            try {
                let options = {
                    firstPage: 'PORTFOLIO_ITEM_OPTIONS',
                    data: {
                        item: this.section.data.items[index],
                        tags: this.section.data.categories
                    }
                }
                const result = await wwLib.wwPopups.open(options)
                
                if (result.prio) {
                    this.section.data.items[index].prio = result.prio;
                    this.sectionCtrl.update(this.section);
                    this.filterItems()
                    this.recalculatePos()
                }
                if (result.tags) {
                    this.section.data.items[index].tags = result.tags;
                    this.sectionCtrl.update(this.section);
                    this.filterItems()
                    this.recalculatePos()
                }
                console.log(result);
            } catch (err) {
            }
        }
        /* wwManager:end */
    },
    mounted() {
        this.init();
    },
    created() {
        this.initData();
    },
    beforeDestroy() {
        window.removeEventListener('resize', this.onResize);
    }
};
</script>

<!-- This is your CSS -->
<!-- Add "scoped" attribute to limit CSS to this component only -->
<!-- Add lang="scss" or others to use computed CSS -->
<style scoped lang="scss">
.portfolio {
    min-height: 100px;

    .background {
        position: absolute;
        top: 0;
        left: 0;
        height: 100%;
        width: 100%;
    }

    .content {
        position: relative;

        .top-wwobjs {
            position: relative;
        }

        .categories {
            margin: 20px 0;
            display: flex;
            justify-content: center;

            .category {
                padding: 0 20px;
                position: relative;

                .name {
                    pointer-events: all;
                    cursor: pointer;
                    color: grey;
                    transition: color 0.3s ease;

                    &:hover {
                        color: black;
                    }
                }

                & + .category::before {
                    content: "•";
                    position: absolute;
                    top: 50%;
                    left: 0;
                    transform: translate(-50%, -50%);
                }
            }
        }

        .items-container {
            display: flex;
            justify-content: center;
            transition: all 0.5s ease;
            // overflow-y: hidden;

            .items {
                position: relative;
                flex-basis: 100%;

                @media (min-width: 992px) {
                    flex-basis: 80%;
                }

                @media (min-width: 1200px) {
                    flex-basis: 90%;
                }

                .item {
                    position: absolute;
                    width: 100%;
                    -webkit-transition: all 0.5s ease;
                    -moz-transition: all 0.5s ease;
                    -o-transition: all 0.5s ease;
                    transition: all 0.5s ease;

                    &.hide {
                        opacity: 0 !important;
                        pointer-events: none;
                    }

                    /* wwManager:start */
                    .ww-orange-button {
                        position: absolute;
                        top: 0;
                        left: 0;
                        transform: translate(-50%, -50%);
                        pointer-events: all;
                        z-index: 20;
                    }
                    /* wwManager:end */
                }
            }
        }
        .items-more-btn {
            pointer-events: all;
            cursor: pointer;
        }
    }
}

.no-anim {
    -webkit-transition: none;
    -moz-transition: none;
    -o-transition: none;
    transition: none;
}

.top-ratio-slider {
    position: absolute;
    top: 10px;
    right: 10px;
    display: flex;
    flex-wrap: nowrap;
    pointer-events: all;
    div {
        width: 150px;
    }
}
</style>
