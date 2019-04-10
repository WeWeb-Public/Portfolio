<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div class="portfolio">
        <!-- wwManager:start -->
        <wwSectionEditMenu :sectionCtrl="sectionCtrl"></wwSectionEditMenu>
        <!-- wwManager:end -->

        <div class="content">
            <wwLayoutColumn tag="div" :ww-list="section.data.topWwObjs" class="top-wwobjs" @ww-add="add(section.data.topWwObjs, $event)" @ww-remove="remove(section.data.topWwObjs, $event)">
                <wwObject v-for="topWwObj in section.data.topWwObjs" :key="topWwObj.uniqueId" :ww-object="topWwObj"></wwObject>
            </wwLayoutColumn>

            <div class="categories">
                <div class="category" v-for="category in categories" :key="category.name" @click="selectedCategory = category.name">
                    <span class="name">{{wwLang.getText(category.displayName)}}</span>
                </div>
            </div>

            <div class="items-container" :style="{'min-height': itemsHeight + 'px'}">
                <div class="items">
                    <div class="item" v-for="(item,index) in section.data.items" :key="index" :data-item="index" :style="getPosition(index)" :class="{'hide': !item.show}">
                        <wwLayoutColumn tag="div" :ww-list="item.data" @ww-add="add(item.data, $event)" @ww-remove="remove(item.data, $event)">
                            <wwObject v-for="wwo in item.data" :key="wwo.uniqueId" :ww-object="wwo" @loaded="recalculatePos()" @update="recalculatePos()"></wwObject>
                        </wwLayoutColumn>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->
<script>
export default {
    name: "__COMPONENT_NAME__",
    props: {
        // The section controller object is passed to you.
        sectionCtrl: Object
    },
    data() {
        return {
            wwLang: wwLib.wwLang,
            selectedCategory: 'all',
            containerWidth: 0,
            columnsHeight: [],
            calculatePosTimeout: null,
            positions: [],
            ratios: [],
            items: [],
            itemsPerLine: 3,
            itemsHeight: 0,
            categories: [
                {
                    name: 'all',
                    displayName: {
                        en: 'All'
                    }
                },
                {
                    name: 'animals',
                    displayName: {
                        en: 'Animals'
                    }
                },
                {
                    name: 'buildings',
                    displayName: {
                        en: 'Buildings'
                    }
                },
                {
                    name: 'landscapes',
                    displayName: {
                        en: 'Landscapes'
                    }
                }
            ]
        }
    },
    computed: {
        //Get the section object here !
        // It contains all the info and data about the section
        // Use it has you like !
        section() {
            return this.sectionCtrl.get();
        },
    },
    watch: {
        selectedCategory() {

            this.filterItems();
        }
    },
    methods: {
        initData() {
            let needUpdate = false;

            //Initialize section data
            this.section.data = this.section.data || {};

            if (!this.section.data.topWwObjs) {
                this.section.data.topWwObjs = [];
                needUpdate = true;
            }

            this.section.data.items = [];
            for (let i = 0; i < 20; i++) {
                const tag1 = this.categories[Math.ceil(Math.random() * (this.categories.length - 1))].name;

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
                    tags: [tag1],
                    show: true,
                    prio: Math.random(),
                    data: [
                        wwLib.wwObject.getDefault({
                            type: 'ww-image',
                            data: {
                                url: url
                            }
                        })
                    ]
                })
            }
            needUpdate = true;

            if (needUpdate) {
                this.sectionCtrl.update(this.section);
            }

            this.filterItems();
        },
        init() {

            this.$nextTick(this.onResize)

            window.addEventListener('resize', this.onResize);
        },
        onResize() {
            this.containerWidth = this.$el.querySelector('.items').getBoundingClientRect().width;

            if (window.innerWidth < 768) {
                this.itemsPerLine = 1;
            }
            else if (window.innerWidth < 992) {
                this.itemsPerLine = 2;
            }
            else {
                this.itemsPerLine = 3;
            }

            this.calculatePos();
        },
        calculatePos() {
            if (!this.$el) {
                return;
            }
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
        recalculatePos() {
            this.ratios = [];
            clearTimeout(this.calculatePosTimeout);
            this.calculatePosTimeout = setTimeout(this.calculatePos, 100);
        },
        getPosition(index) {
            return this.positions[index] || {}
        },
        filterItems() {
            if (this.selectedCategory == 'all') {
                for (let item of this.section.data.items) {
                    item.show = true;
                }
            }
            else {
                for (let item of this.section.data.items) {
                    item.show = item.tags.indexOf(this.selectedCategory) !== -1;
                }
            }
            this.section.data.items.sort(function (i1, i2) {
                return i1.prio < i2.prio ? 1 : -1;
            })
            this.ratios = [];
            this.$nextTick(this.calculatePos);
            this.$forceUpdate();
        },

        /*=============================================m_ÔÔ_m=============================================\
          TOP WWBJECTS
        \================================================================================================*/
        add(list, options) {
            list.splice(options.index, 0, options.wwObject);

            this.sectionCtrl.update(this.section);

            this.recalculatePos();
        },
        remove(options) {
            list.splice(options.index, 1);

            this.sectionCtrl.update(this.section);

            this.recalculatePos();
        },
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
            overflow-y: hidden;

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
                    transition: all 0.5s ease;

                    &.hide {
                        opacity: 0;
                        pointer-events: none;
                    }

                    @media (min-width: 768px) {
                        width: 50%;
                    }

                    @media (min-width: 992px) {
                        width: 33.333%;
                    }
                }
            }
        }
    }
}
</style>
