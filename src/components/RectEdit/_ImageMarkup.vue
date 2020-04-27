<template>
  <div class="custom-editor">
    <canvas id="c" @update="updateAnnotation(annotation)"> </canvas>
  </div>
</template>

<script>
import { fabric } from 'fabric'
import Shape from '../../assets/js/shape'
import Text from '../../assets/js/text'
import CropImage from '../../assets/js/crop'
import CanvasHistory from '../../assets/js/canvasHistory'

export default {
  name: 'Editor',
  props: {
    canvasWidth: Number,
    canvasHeight: Number,
    annotation: Object
  },
  data () {
    return {
      canvas: null,
      pointerX: null,
      pointerY: null,
      createRect: false,
      createText: false,
      currentActiveMethod: null,
      currentActiveTool: null,
      objects: [],
      width: null,
      height: null,
      params: {},
      color: '#000000',
      strokeWidth: 4,
      fontSize: 32,
      croppedImage: false,
      history: [],
      imageURL: ''
    }
  },
  mounted () {
    this.canvas = new fabric.Canvas('c')
  },
  methods: {
    setBackgroundImage (imageUrl) {
      if (imageUrl !== this.imageURL) {
        this.clear()
        this.imageURL = imageUrl
        this.canvas.setDimensions({ width: this.canvasWidth, height: this.canvasHeight })
        this.canvas.backgroundColor = '#fff'
        let canvasProperties = { width: this.canvas.width, height: this.canvas.height }
        let currentCanvas = { json: this.canvas.toJSON(), canvas: canvasProperties }
        new CanvasHistory(this.canvas, currentCanvas)
        let img = new Image()
        this.toDataUrl(imageUrl, (dataUri) => {
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
        var params = {
          fill: 'transparent',
          width: this.annotation.width,
          height: this.annotation.height,
          angle: 0,
          top: this.annotation.x,
          left: this.annotation.y,
          strokeWidth: 4,
          stroke: 'black',
          opacity: 1,
          noScaleCache: false,
          strokeUniform: true
        }
        this.drawRect(params)
      }
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
      this.canvas.clear()
      this.cancelCroppingImage()
    },
    set (type, params) {
      this.canvas.off('mouse:down')
      switch (type) {
        case 'rect':
          //this.clear()
          this.cancelCroppingImage()
          this.currentActiveTool = type
          this.params = {
            fill: (params && params.fill) ? params.fill : 'transparent',
            stroke: (params && params.stroke) ? params.stroke : this.color,
            strokeWidth: (params && params.strokeWidth) ? params.strokeWidth : this.strokeWidth,
            angle: (params && params.angle) ? params.angle : 0,
            width: (params && params.width) ? params.width : null,
            height: (params && params.height) ? params.height : null,
            top: (params && params.top) ? params.top : 0,
            left: (params && params.left) ? params.left : 0,
            opacity: (params && params.opacity) ? params.opacity : 1,
            strokeUniform: (params && params.strokeUniform) ? params.strokeUniform : true,
            noScaleCache: (params && params.noScaleCache) ? params.noScaleCache : false,
            strokeDashArray: (params && params.strokeDashArray) ? params.strokeDashArray : false,
            borderRadius: (params && params.borderRadius) ? params.borderRadius : 0
          }
          this.customRect(type, this.params)
          break
        case 'selectMode':
          this.currentActiveTool = type
          this.drag()
          break
        default:
      }
    },
    saveImage () {
      this.cancelCroppingImage()
      return this.canvas.toDataURL('image/jpeg', 1)
    },
    customRect (type, params) {
      this.currentActiveMethod = this.customRect
      this.canvas.isDrawingMode = false
      if (params.width && params.height) {
        this.drawRect(params)
      } else {
        this.createRect = true
        new Shape(this.canvas, this.createRect, type, params)

        // http://fabricjs.com/fabric-intro-part-2#events
        this.canvas.on('mouse:up', function (options) {
          console.log('Rectangle updated ' + options.target.top + ',' + options.target.left + 'x' + Math.ceil(options.target.top) + Math.ceil(options.target.left))
          this.annotation.width = Math.ceil(options.target.width)
          this.annotation.height = Math.ceil(options.target.width)
          this.annotation.x = Math.ceil(options.target.top)
          this.annotation.y = Math.ceil(options.target.left)
          this.$emit('change', this.annotation)
        })
      }
    },
    cancelCroppingImage () {
      this.croppedImage = false
      new CropImage(this.canvas, false, false, true)
    },
    drag () {
      this.currentActiveMethod = this.drag
      this.canvas.isDrawingMode = false
      this.canvas.forEachObject(object => {
        object.selectable = true
        object.evented = true
      })
      if (this.createRect) {
        this.createRect = false
        new Shape(this.canvas, false)
        console.log('Creating rectangle')
      }
      if (this.createText) {
        this.createText = false
        new Text(this.canvas, false)
      }
      this.cancelCroppingImage()
    },
    drawRect (params) {
      this.drag()
      this.canvas.discardActiveObject()
      if (!this.canvas.getActiveObject()) {
        this.rectangle = new fabric.Rect({
          width: params.width,
          height: params.height,
          strokeWidth: params.strokeWidth,
          stroke: params.stroke,
          fill: params.fill,
          opacity: params.opacity,
          left: params.left,
          top: params.top,
          noScaleCache: params.noScaleCache
        })
        console.log('Rectangle added')
        this.canvas.add(this.rectangle)
        this.canvas.renderAll()
        // this.annotation.width = params.width
        // this.annotation.height = params.height
        // this.annotation.x = params.left
        // this.annotation.y = params.top
        // this.$emit('change', this.annotation)
        // v-on:change="$emit('change', $event.target.annotation)
        // // give data back to the parent
        // console.log('changeRectangle')
        // this.$emit('change-rectangle', this.rectangle)
      }
    },
    updateRectangle () {
      // give data back to the parent
      this.$emit('update', this.annotation)
    }
  }

}
</script>
