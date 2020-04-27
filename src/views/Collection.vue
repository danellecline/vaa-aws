<template>
  <div>
    <split-pane :min-percent='20' :max-percent='50' :default-percent='60' split="vertical">
      <template slot="paneL">
        <h1>Collection: {{ collectionName }}</h1>
        <div id="addreplace">
          <div id="change">
            <div>
              <input ref="newname" @keypress.enter="add" />
              <button @click="add">Add new name</button>
            </div>
          </div>
        </div>
        <div class="ui horizontal segment">
              <span>Filter by name</span>
              <div>
                <model-select
                        :options="options"
                        v-model="item"
                        placeholder="type name here to search"
                        ref="filterbyname"
                        @input="$refs.cpt.filter('filterByItem')"
                >
                </model-select>
                <div class="button-group">
                  <button @click="$refs.cpt.unfilter()">Unfilter</button>
                  <button @click="$refs.cpt.shuffle()">Shuffle</button>
                  <button @click="$refs.cpt.sort('name')">Sort by name</button>
                </div>
                <span>Total Examples: {{ totalPerClass }}</span>
              </div>
          </div>
        <div>
          <isotope ref="cpt" id="root_isotope" class="isoDefault" :options='getOptions()'
                   :list="annotation" @filter="filterOption=arguments[0]"
                   @sort="sortOption=arguments[0]">
            <div v-for="c in annotation" :key="c.id" @click="selected=c">
              <img :src="c.crop_url" />
            </div>
          </isotope>
        </div>
      </template>
      <template slot="paneR">
        <div v-if="selected" class="item">
          <Editor ref="editor" @changeRectangle="updateRectangle($event)" @changeName="updateName($event)" @deleteById="deleteById($event)"
                  :options="options"
                  :annotation="selected"
                  :canvas-height="300"
                  :canvas-width="300" ></Editor>
        </div>
      </template>
    </split-pane>
  </div>
</template>

<script>
import isotope from 'vueisotope'
import splitPane from 'vue-splitpane'
import Editor from './Editor'
import { ModelSelect } from 'vue-search-select'
import 'vue-search-select/dist/VueSearchSelect.css'

export default {
  data () {
    return {
      annotation: [
        {
          id: 0,
          name: 'Danelle',
          bounding_box: { height: 100, width: 50, x: 100, y: 135 },
          frame_crop_url: 'http://localhost:3000/static/fullframe/D0232_03HD_00-57-05.jpg',
          crop_url: 'http://localhost:3000/static/LONG_WHITE/D0232_03HD_00-48-10_0.jpg'
        },
        {
          id: 1,
          name: 'Evan',
          bounding_box: { height: 100, width: 50, x: 50, y: 35 },
          frame_crop_url: 'http://localhost:3000/static/fullframe/D0232_03HD_00-30-25.jpg',
          crop_url: 'http://localhost:3000/static/PENIAGONE_SP_A/D0232_03HD_00-45-00_0.jpg'
        },
        {
          id: 14,
          name: 'Jimmy',
          bounding_box: { height: 150, width: 150, x: 10, y: 35 },
          frame_crop_url: 'http://localhost:3000/static/fullframe/D0232_03HD_00-14-25.jpg',
          crop_url: 'http://localhost:3000/static/BENTHOCODON/D0232_03HD_01-00-05_0.jpg'
        }
      ],
      selected: null,
      sortOption: null,
      filterOption: null,
      collectionName: 'Benthic2017',
      options: [
        { text: 'Danelle', value: '0' },
        { text: 'Jimmy', value: '1' },
        { text: 'Evan', value: '2' }],
      item: {
        text: '',
        value: ''
      },
      totalPerClass: 0
    }
  },
  mounted () {
    console.log('Mounted - cache images here')
    totalPerClass = this.list.length
  },
  methods: {
    updateName (event) {
      console.log('Annotation name updated==============: ', event)
      var replace = this.selected
      replace.name = event
      this.list = [this.selected, replace]
    },
    updateRectangle (event) {
      console.log('Annotation rectangle updated==============: ', event)
      var replace = this.selected
      replace.bounding_box.width = event.width
      replace.bounding_box.height = event.height
      replace.bounding_box.x = event.left
      replace.bounding_box.y = event.top
      this.list = [this.selected, replace]
    },
    deleteById (event) {
      console.log('Delete ', event)
      this.annotation.splice(this.annotation.indexOf(event), 1)
    },
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
      var option = this.$refs.newname.value
      console.log(option)

      if (option!== '') {
        let n = {text: option, value: this.options.length}
        this.options.push(n) 
        //value: (l + 1).toString() })
        this.$refs.newname.value = ''
        this.item.text = option
//         this.filter(input.value)
//         this.$refs.editor.$options = this.options
      }
    },
    sort: function (key) {
      this.isotopeSort(key)
      this.sortOption = key
    },
    filter: function (key) {
      if (this.filterOption === key) {
        key = null
      }
      this.isotopeFilter(key)
      this.filterOption = key
    },
    shuffle: function () {
      this.isotopeShuttle()
      this.sortOption = null
    }
  },
  components: {
    isotope,
    splitPane,
    ModelSelect,
    Editor: Editor
  }
}
</script>
