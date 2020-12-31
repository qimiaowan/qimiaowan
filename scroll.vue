<template>
  <div class="scroll-wrap" :style="{height:wrapHeight}">
    <div ref='list-wrap' class="scroll-list" @mouseenter="Stop()" @mouseleave="Up()" :style="top" >
      <div class="scroll-item" v-for="(curData, idx) in listDataLen" :key="idx">
        <template  v-for="(item, index) in curData">
          <slot :item="item" :index="index"></slot>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ScrollList',
  data () {
    return {
      activeIndex: 1,
      timerList: null,
      time: 30,
      h: 0
    }
  },
  beforeDestroy () {
    if (this.timerList) {
      clearInterval(this.timerList)
      this.timerList = null
    }
  },
  created () {
    this.$nextTick(() => {
      this.ScrollUp()
    })
  },
  computed: {
    listDataLen () {
      return this.listData.reduce((prev, item, index) => {
        const len = Math.ceil(this.listData.length / 2)
        if (index < len) {
          prev[0].push(item)
        } else {
          prev[1].push(item)
        }
        return prev
      }, [[], []])
    },
    top () {
      return `transform:translateY(${-this.activeIndex}px)`
    }
  },
  props: {
    listData: {
      type: Array,
      default () {
        return []
      }
    },
    wrapHeight: {
      type: String,
      default () {
        return '100%'
      }
    }
  },
  methods: {
    comHeight (child) {
      return Math.ceil(child.getBoundingClientRect().height)
    },
    ScrollUp () {
      this.h = this.comHeight(this.$refs['list-wrap'].children[0])
      if (this.h <= 0) {
        return
      }
      if (this.timerList) {
        clearInterval(this.timerList)
      }
      this.timerList = setInterval(_ => {
        if (this.activeIndex % this.h === 0) {
          const elem = this.$refs['list-wrap']
          const child = elem.children[0]
          this.activeIndex -= this.comHeight(child)
          elem.insertBefore(elem.children[elem.children.length - 1], child)
          this.h = this.comHeight(elem.children[0])
        }
        this.activeIndex += 1
      }, this.time)
    },
    Stop () {
      clearInterval(this.timerList)
    },
    Up () {
      this.ScrollUp()
    }
  }
}
</script>

<style scoped>
  .scroll-wrap, .scroll-item {
    overflow: hidden;
  }
</style>
