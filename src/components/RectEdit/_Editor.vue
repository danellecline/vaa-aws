<template>
    <div id="recteditor" class="main">
        <div class="editor-container">
            <div class="editor">
                <Tool :event="() => undo()" :iconClass="'fas fa-undo-alt fa-lg'" />
                <Tool :event="() => redo()" :iconClass="'fas fa-redo-alt fa-lg'" />
                <Tool :event="() => clear()" :iconClass="'fas fa-trash-alt fa-lg'" />
                <Tool
                        :event="() => setTool('rect')"
                        :iconClass="'far fa-square fa-lg'"
                        :class="{ 'active-tool': currentActiveMethod === 'rect' }"
                />
                <Tool
                        :event="() => setTool('selectMode')"
                        :iconClass="'fas fa-arrows-alt fa-lg'"
                        :class="{ 'active-tool': currentActiveMethod === 'selectMode' }"
                />
                <Tool :event="() => saveImage()" :iconClass="'fas fa-save fa-lg'" />
            </div>
            <Editor
                    :canvasWidth="canvasWidth"
                    :canvasHeight="canvasHeight"
                    :annotation="annotation"
                    :options="options"
                    ref="editor"
            />
        </div>
    </div>
</template>

<script>
import Editor from 'vue-image-markup'
import Tool from '../RectEdit/_Tool'
import 'vue-search-select/dist/VueSearchSelect.css'
import '@fortawesome/fontawesome-free/css/all.css'
import '@fortawesome/fontawesome-free/js/all.js'

export default {
  data () {
    return {
      currentActiveMethod: null,
      params: {},
      color: 'black',
      options: {},
      item: {
        text: '',
        value: ''
      }
    }
  },
  props: {
    canvasWidth: {
      default: 600
    },
    event: {
      type: Function
    },
    iconClass: {
      type: String
    },
    canvasHeight: {
      default: 600
    },
    annotation: {
      type: Object
    }

  },
  mounted () {
    if (this.annotation) {
      this.$refs.editor.setBackgroundImage(this.annotation.ref)

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

      this.$refs.editor.drawRect(params)
    }
    this.$watch(
      () => {
        return this.$refs.editor.croppedImage
      },
      val => {
        this.croppedImage = val
      }
    )
  },
  updated () {
    if (this.annotation) {
      this.$refs.editor.setBackgroundImage(this.annotation.ref)
    }
  },
  methods: {
    saveImage () {
      let image = this.$refs.editor.saveImage()
      this.saveImageAsFile(image)
    },
    saveImageAsFile (base64) {
      let link = document.createElement('a')
      link.setAttribute('href', base64)
      link.setAttribute('download', 'image-markup')
      link.click()
    },
    setTool (type, params) {
      this.currentActiveMethod = type
      this.$refs.editor.set(type, params)
    },
    clear () {
      this.currentActiveMethod = this.clear
      this.$refs.editor.clear()
    },
    undo () {
      this.currentActiveMethod = this.undo
      this.$refs.editor.undo()
    },
    redo () {
      this.currentActiveMethod = this.redo
      this.$refs.editor.redo()
    }
  },
  components: {
    Tool,
    Editor
  }
}
</script>


<style src="../../assets/app.scss" lang="scss"></style>
