<template>
  <div
    :style="getOutStyleDefault(config.style)"
    :class="{'gap_class':canvasStyleData.panel.gap==='yes'}"
    class="component"
    @click="handleClick"
    @mousedown="elementMouseDown"
  >
    <edit-bar v-if="config === curComponent" :element="config" @showViewDetails="showViewDetails" />
    <de-out-widget
      v-if="config.type==='custom'"
      :id="'component' + config.id"
      class="component-custom"
      :style="getComponentStyleDefault(config.style)"
      :out-style="config.style"
      :element="config"
      :in-screen="inScreen"
    />

    <component
      :is="config.component"
      v-else
      ref="wrapperChild"
      :out-style="config.style"
      :style="getComponentStyleDefault(config.style)"
      :prop-value="config.propValue"
      :is-edit="false"
      :element="config"
      :search-count="searchCount"
    />
  </div>
</template>

<script>
import { getStyle } from '@/components/canvas/utils/style'
import runAnimation from '@/components/canvas/utils/runAnimation'
import { mixins } from '@/components/canvas/utils/events'
import { mapState } from 'vuex'
import DeOutWidget from '@/components/dataease/DeOutWidget'
import EditBar from '@/components/canvas/components/Editor/EditBar'

export default {
  components: { DeOutWidget, EditBar },
  mixins: [mixins],
  props: {
    config: {
      type: Object,
      require: true,
      default: null
    },
    filter: {
      type: Object,
      require: false,
      default: null
    },
    searchCount: {
      type: Number,
      required: false,
      default: 0
    },
    inScreen: {
      type: Boolean,
      required: false,
      default: true
    }
  },
  computed: {
    ...mapState([
      'canvasStyleData',
      'curComponent'
    ])
  },
  mounted() {
    runAnimation(this.$el, this.config.animations)
  },
  methods: {
    getStyle,

    getOutStyleDefault(style) {
      const result = {};
      ['width', 'left'].forEach(attr => {
        result[attr] = style[attr] + 'px'
      });
      ['height', 'top'].forEach(attr => {
        result[attr] = style[attr] + 'px'
      })
      result['rotate'] = style['rotate']
      result['opacity'] = style['opacity']

      return result
      // return style
    },

    getComponentStyleDefault(style) {
      return getStyle(style, ['top', 'left', 'width', 'height', 'rotate'])
      // return style
    },

    handleClick() {
      const events = this.config.events
      Object.keys(events).forEach(event => {
        this[event](events[event])
      })
    },
    elementMouseDown(e) {
      // private 设置当前组件数据及状态
      this.$store.commit('setClickComponentStatus', true)
      if (this.config.component !== 'v-text' && this.config.component !== 'rect-shape' && this.config.component !== 'de-input-search' && this.config.component !== 'de-select-grid' && this.config.component !== 'de-number-range' && this.config.component !== 'de-date') {
        e.preventDefault()
      }
      // 阻止冒泡事件
      e.stopPropagation()
      this.$store.commit('setCurComponent', { component: this.config, index: this.index })
    },
    showViewDetails() {
      this.$refs.wrapperChild.openChartDetailsDialog()
    }
  }
}
</script>

<style lang="scss" scoped>
.component {
    position: absolute;
}

.component:hover {
  box-shadow:0px 0px 7px #0a7be0;
}
.gap_class{
  padding:5px;
}
.component-custom {
  outline: none;
  width: 100% !important;
  height: 100%;
}
</style>
