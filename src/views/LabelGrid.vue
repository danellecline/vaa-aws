<template>
  <div>
    <split-pane :min-percent='20' :max-percent='50' :default-percent='60' split="vertical">
      <template slot="paneL">
        <h1>Collection: {{ collectionName }}</h1>
        <div id="addreplace">
          <div id="change">
            <div>
              <input id="itemForm" @keypress.enter="Add" />
              <button @click="add">Add new name</button>
            </div>
            <button @click="replace">Replace</button>
            <button @click="remove">Remove</button>
          </div>
        </div>
        <div class="ui horizontal segment">
              <span>Filter by name</span>
              <div>
                <model-select
                        :options="options"
                        v-model="item"
                        placeholder="type name here to search"
                        :class="[filterOption==='filterByItem' ? 'is-checked' : '']"
                        @click="$refs.cpt.filter('filterByItem')"
                >
                </model-select>
                <div class="button-group">
                  <button @click="$refs.cpt.unfilter()">Unfilter</button>
                  <button @click="$refs.cpt.shuffle()">Shuffle</button>
                  <button @click="$refs.cpt.sort('name')">Sort by name</button>
                </div>
                <span>Name: {{ item.text }} Total Examples: {{ item.value }}</span>
              </div>
          </div>
        <div>
          <isotope ref="cpt" id="root_isotope" class="isoDefault" :options='getOptions()'
                   :list="annotation" @filter="filterOption=arguments[0]"
                   @sort="sortOption=arguments[0]">
            <div v-for="c in annotation" :key="c.id" @click="selected=c">
              <img :src="c.url" />
            </div>
          </isotope>
        </div>
      </template>
      <template slot="paneR">
        <div v-if="selected" class="item">
          <RectEditor ref="editor" :options="options" :annotation="selected" :canvas-height="300" :canvas-width="300" ></RectEditor>
        </div>
      </template>
    </split-pane>
  </div>
</template>

<script>
import isotope from 'vueisotope'
import splitPane from 'vue-splitpane'
import RectEditor from '../components/RectEdit/_Editor'
import { ModelSelect } from 'vue-search-select'
import 'vue-search-select/dist/VueSearchSelect.css'

export default {
  data () {
    return {
      annotation: [
        {
          id: 0,
          name: 'Danelle',
          height: 100,
          width: 50,
          x: 10,
          y: 35,
          ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-57-05.jpg',
          url: 'http://localhost:3000/static/LONG_WHITE/D0232_03HD_00-48-10_0.jpg'
        },
        {
          id: 1,
          name: 'AEvan',
          height: 100,
          width: 50,
          x: 10,
          y: 35,
          ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
          url: 'http://localhost:3000/static/PENIAGONE_SP_A/D0232_03HD_00-45-00_0.jpg'
        },
        {
          id: 2,
          name: 'BEvan',
          height: 100,
          width: 50,
          x: 10,
          y: 35,
          ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
          url: 'http://localhost:3000/static/PENIAGONE_SP_A/D0232_03HD_00-45-00_0.jpg'
        },
        {
          id: 3,
          name: 'CEvan',
          height: 100,
          width: 50,
          x: 10,
          y: 35,
          ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
          url: 'http://localhost:3000/static/PENIAGONE_SP_A/D0232_03HD_00-45-00_0.jpg'
        },
        {
          id: 4,
          name: 'DEvan',
          height: 100,
          width: 50,
          x: 10,
          y: 35,
          ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
          url: 'http://localhost:3000/static/PENIAGONE_SP_A/D0232_03HD_00-45-00_0.jpg'
        },
        {
          id: 5,
          name: 'Evan',
          height: 100,
          width: 50,
          x: 10,
          y: 35,
          ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
          url: 'http://localhost:3000/static/PENIAGONE_SP_A/D0232_03HD_00-45-00_0.jpg'
        },
        {
          id: 6,
          name: 'Evan',
          height: 100,
          width: 50,
          x: 10,
          y: 35,
          ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
          url: 'http://localhost:3000/static/PENIAGONE_SP_A/D0232_03HD_00-45-00_0.jpg'
        },
        {
          id: 7,
          name: 'Evan',
          height: 100,
          width: 50,
          x: 10,
          y: 35,
          ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
          url: 'http://localhost:3000/static/PENIAGONE_SP_A/D0232_03HD_00-45-00_0.jpg'
        },
        {
          id: 8,
          name: 'Evan',
          height: 100,
          width: 50,
          x: 10,
          y: 35,
          ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
          url: 'http://localhost:3000/static/PENIAGONE_SP_A/D0232_03HD_00-45-00_0.jpg'
        },
        {
          id: 9,
          name: 'Evan',
          height: 100,
          width: 50,
          x: 10,
          y: 35,
          ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
          url: 'http://localhost:3000/static/PENIAGONE_SP_A/D0232_03HD_00-45-00_0.jpg'
        },
        {
          id: 10,
          name: 'Evan',
          height: 100,
          width: 50,
          x: 10,
          y: 35,
          ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
          url: 'http://localhost:3000/static/PENIAGONE_SP_A/D0232_03HD_00-45-00_0.jpg'
        },
        {
          id: 11,
          name: 'Evan',
          height: 100,
          width: 50,
          x: 10,
          y: 35,
          ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
          url: 'http://localhost:3000/static/PENIAGONE_SP_A/D0232_03HD_00-45-00_0.jpg'
        },
        {
          id: 12,
          name: 'Evan',
          height: 100,
          width: 50,
          x: 10,
          y: 35,
          ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
          url: 'http://localhost:3000/static/PENIAGONE_SP_A/D0232_03HD_00-45-00_0.jpg'
        },
        {
          id: 13,
          name: 'Evan',
          height: 100,
          width: 50,
          x: 10,
          y: 35,
          ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
          url: 'http://localhost:3000/static/PENIAGONE_SP_A/D0232_03HD_00-45-00_0.jpg'
        },
        {
          id: 14,
          name: 'AAAJimmy',
          height: 100,
          width: 50,
          x: 10,
          y: 35,
          ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
          url: 'http://localhost:3000/static/BENTHOCODON/D0232_03HD_01-00-05_0.jpg'
        }
      ],
      selected: null,
      sortOption: null,
      filterOption: null,
      collectionName: 'Benthic2017',
      options: [
        { text: 'Danelle', value: '4' },
        { text: 'AAAJimmy', value: '3' },
        { text: 'Evan', value: '0' },
        { text: 'AEvan', value: '1' },
        { text: 'BEvan', value: '2' },
        { text: 'CEvan', value: '5' }],
      item: {
        text: '',
        value: ''
      }
    }
  },
  // mounted() {
  //   this.$refs.sort.editor.set(this.editor.mode,this.editor.options);
  // },
  methods: {
    getOptions: function () {
      var _this = this
      return {
        layoutMode: 'masonry',
        masonry: {
          gutter: 1
        },
        getSortData: {
          id: 'id',
          name: 'name'
        },
        getFilterData: {
          'show all': function () {
            return true
          },
          filterByItem: function (itemElem) {
            return itemElem.name.includes(_this.item.text)
          }
        }
      }
    },
    add: function () {
      var input = this.$ref.itemForm

      if (input.value !== '') {
        this.options.push({ text: input.value, value: '0' })
        input.value = ''
        this.filter(input.value)
      }
    },
    replace: function () {
      this.list = [{ name: 'AAAJimmy', id: 14 }, {
        name: 'AAAJimmyFoobar',
        value: '0',
        height: 100,
        width: 50,
        x: 10,
        y: 35,
        ref: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
        url: 'http://localhost:3000/static/LONG_WHITE/D0232_03HD_00-48-10_0.jpg'
      }]
    },
    remove: function () {
      if (this.list.length) {
        this.list.splice(0, 1)
      }
    },
    sort: function (key) {
      this.isotopeSort(key)
      this.sortOption = key
    },
    filter: function (key) {
      if (this.filterOption == key) {
        key = null
      }
      this.isotopeFilter(key)
      this.filterOption = key
    },
    shuffle: function () {
      this.isotopeShuttle()
      this.sortOption = null;
    }
  },
  components: {
    isotope,
    splitPane,
    ModelSelect,
    RectEditor
  }
}
</script>
