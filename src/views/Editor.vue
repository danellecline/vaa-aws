<template>
    <div id="recteditor" class="main">
        <div class="editor-container">
            <div class="editor">
                <Tool :event="() => clear()" :iconClass="'fas fa-trash-alt fa-lg'" />
                <Tool
                        :event="() => setTool('selectMode')"
                        :iconClass="'fas fa-arrows-alt fa-lg'"
                />
                <Tool :event="() => saveImage()" :iconClass="'fas fa-save fa-lg'" />
            </div>
          <div>
            <model-select
                :options="options"
                :value="item"
                placeholder="type name here to rename"
                ref="modelselect"
                @input="handleText"
               >
              </model-select>
            <span>ID: {{annotation.id}}</span>
            <canvas id="recteditorcanvas" />
            </div>
        </div>
    </div>
</template>

<script>
import Tool from '../components/RectEdit/_Tool'
import { ModelSelect } from 'vue-search-select'
import 'vue-search-select/dist/VueSearchSelect.css'
import '@fortawesome/fontawesome-free/css/all.css'
import '@fortawesome/fontawesome-free/js/all.js'
import { fabric } from 'fabric'
import CanvasHistory from '../assets/js/canvasHistory'

export default {
  data () {
    return {
      searchText: '',
      item: {
        text: '',
        value: ''
      },
      canvas: null,
      pointerX: null,
      pointerY: null,
      createRect: false,
      currentActiveTool: null,
      objects: [],
      width: null,
      height: null,
      color: '#FFFFFF',
      strokeWidth: 4,
      fontSize: 20,
      history: [],
      imageUrl: '',
      opacity: 0.3,
      scaleFactor: 1.0,
      rectWidth: 0,
      rectHeight: 0,
      rectTop: 0,
      rectLeft: 0,
      name: '',
      startup: true,
      rectUpdate: false,
      nameUpdate: false
    }
  },
  props: {
    canvasWidth: {
      default: 600
    },
    iconClass: {
      type: String
    },
    canvasHeight: {
      default: 600
    },
    options: {
      type: Array
    },
    annotation: {
      type: Object
    }

  },
  mounted () {
    console.log('Editor mounted. Creating canvas.')
    this.canvas = new fabric.Canvas('recteditorcanvas')
    this.startup = true
    this.update()
  },
  updated () {
    console.log('=====Updated')
    this.update()
  },
  methods: {
    update () {
      if (this.startup || (this.imageUrl !== this.annotation.frame_crop_url)) {
        this.startup = false
        console.log('=====Update')
        console.log(this.annotation.name)
        this.clear()
        this.setBackgroundImage(this.annotation.frame_crop_url)
        this.imageUrl = this.annotation.frame_crop_url
        this.rectHeight = this.annotation.bounding_box.height
        this.rectWidth = this.annotation.bounding_box.width
        this.rectLeft = this.annotation.bounding_box.x
        this.rectTop = this.annotation.bounding_box.y
        var option
        for (option of this.options) {
          if (option.text === this.annotation.name) {
            this.$refs.modelselect.selectItem(option)
            console.log('Selecting item ' + option.text)
          }
        }
      }
    },
    handleText ($event) {
      if ($event.text !== this.item.text) {
        this.nameUpdate = true
      }
      this.item = $event
      this.updateText(this.item.text)
    },
    updateText (text) {
      console.log('Updating text')
      var objects = this.canvas.getObjects('text')
      for (let i in objects) {
        this.canvas.remove(objects[i])
      }
      objects = this.canvas.getObjects('rect')
      for (let i in objects) {
        this.canvas.remove(objects[i])
      }
      this.draw(text)
      if (this.name !== text) {
        this.nameUpdate = true
      }
    },
    saveImage () {
      if (this.rectUpdate) {
        var rectangle = { top: this.rectTop, left: this.rectLeft, width: this.rectWidth, height: this.rectHeight }
        this.$emit('changeRectangle', rectangle)
        console.log('Rectangle changed')
        this.rectUpdate = false
      }
      if (this.nameUpdate) {
        this.$emit('changeName', this.item.text)
        console.log('changed name')
        this.nameUpdate = false
      }
    },
    setTool (type) {
      this.set(type)
    },
    setBackgroundImage (url) {
      this.canvas.setDimensions({ width: this.canvasWidth, height: this.canvasHeight })
      this.canvas.backgroundColor = '#fff'
      let canvasProperties = { width: this.canvas.width, height: this.canvas.height }
      let currentCanvas = { json: this.canvas.toJSON(), canvas: canvasProperties }
      new CanvasHistory(this.canvas, currentCanvas)
      let img = new Image()
      this.toDataUrl(url, (dataUri) => {
        img.src = dataUri
        let inst = this
        img.onload = function () {
          if (inst.canvas.width <= img.width || inst.canvas.height <= img.height) {
            let canvasAspect = inst.canvas.width / inst.canvas.height
            let imgAspect = img.width / img.height
            let top, left
            if (canvasAspect >= imgAspect) {
              this.scaleFactor = inst.canvas.height / img.height
              top = 0
              left = -((img.width * this.scaleFactor) - inst.canvas.width) / 2
            } else {
              this.scaleFactor = inst.canvas.width / img.width
              left = 0
              top = -((img.height * this.scaleFactor) - inst.canvas.height) / 2
            }
            inst.canvas.setBackgroundImage(dataUri, inst.canvas.renderAll.bind(inst.canvas), {
              top: top,
              left: left,
              scaleX: this.scaleFactor,
              scaleY: this.scaleFactor
            })
            inst.canvas.renderAll()
          } else {
            let center = inst.canvas.getCenter()
            inst.canvas.setBackgroundImage(dataUri, inst.canvas.renderAll.bind(inst.canvas), {
              top: center.top,
              left: center.left,
              originX: 'center',
              originY: 'center'
            })
            inst.canvas.renderAll()
          }
        }
      })
    },
    toDataUrl (url, callback) {
      let xhr = new XMLHttpRequest()
      xhr.onload = function () {
        let reader = new FileReader()
        reader.onloadend = () => {
          callback(reader.result)
        }
        reader.readAsDataURL(xhr.response)
      }
      xhr.open('GET', url)
      xhr.responseType = 'blob'
      xhr.send()
    },
    clear () {
      if (this.canvas !== null) {
        var objects = this.canvas.getObjects('rect')
        for (let i in objects) {
          this.canvas.remove(objects[i])
        }
        objects = this.canvas.getObjects('text')
        for (let i in objects) {
          this.canvas.remove(objects[i])
        }
      }
    },
    set (type) {
      this.canvas.off('mouse:down')
      switch (type) {
        case 'rect':
          this.currentActiveTool = type
          this.params = {
            fill: 'transparent',
            stroke: this.color,
            strokeWidth: this.strokeWidth,
            angle: 0,
            width: null,
            height: null,
            top: 0,
            left: 0,
            opacity: 1,
            strokeUniform: true,
            noScaleCache: false,
            strokeDashArray: true,
            borderRadius: 0
          }
          this.customRect(type, this.params)
          break
        case 'selectMode':
          console.log('Select mode')
          this.currentActiveTool = type
          this.drag()
          break
        default:
      }
    },
    drag () {
      this.canvas.isDrawingMode = false
      this.canvas.forEachObject(object => {
        object.selectable = true
        object.evented = true
      })
      let inst = this
      this.canvas.on('object:modified', function (evt) {
        console.log('object modified')
        if (evt.target != null && evt.target.get('type') === 'rect') {
          var w = Math.ceil(evt.target.width)
          var h = Math.ceil(evt.target.height)
          var t = Math.ceil(evt.target.top)
          var l = Math.ceil(evt.target.left)
          if ((w !== inst.rectWidth) ||
            (h !== inst.rectHeight) ||
            (t !== inst.rectTop) ||
            (l !== inst.rectLeft) ) {
            inst.rectWidth = Math.ceil(evt.target.width)
            inst.rectHeight = Math.ceil(evt.target.height)
            inst.rectTop = Math.ceil(evt.target.top)
            inst.rectLeft = Math.ceil(evt.target.left)
            console.log('Rectangle updated ' + inst.rectTop + ',' + inst.rectLeft + ' ' + inst.rectWidth + ' x ' + inst.rectHeight)
            var objects = inst.canvas.getObjects('text')
            for (let i in objects) {
              objects[i].top = inst.rectTop
              objects[i].left = inst.rectLeft
            }
            inst.rectUpdate = true
          }
        }
      })
    },
    draw (t) {
      this.drag()
      this.canvas.discardActiveObject()
      if (!this.canvas.getActiveObject()) {
        console.log('Adding rectangle')
        var rectangle = new fabric.Rect({
          fill: 'transparent',
          width: this.rectWidth,
          height: this.rectHeight,
          angle: 0,
          top: this.rectTop,
          left: this.rectLeft,
          strokeWidth: 4,
          stroke: this.color,
          opacity: this.opacity,
          noScaleCache: false,
          strokeUniform: true
        })
        this.canvas.add(rectangle)

        console.log('Adding text ' + t)
        this.canvas.add(new fabric.Text(t, {
          fill: this.color,
          fontSize: this.fontSize,
          opacity: this.opacity,
          top: this.rectTop,
          left: this.rectLeft
        }))
      }
      this.canvas.renderAll()
    }
  },
  components: {
    Tool,
    ModelSelect
  }
}
</script>

<style src="../assets/app.scss" lang="scss"></style>
