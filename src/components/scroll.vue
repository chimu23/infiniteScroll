<template>
  <div class="viewport" ref="viewport" @scroll="handleScroll">
    <div class="content" :style="{ transform: `translate3d(0,${offset}px,0)` }">
      <div
        v-for="item in visibleData"
        :vid="item.index"
        :key="item.index"
        ref="item"
      >
        <slot name="items" :item="item"></slot>
      </div>
    </div>
    <div class="scroll-bar" ref="scrollBar"></div>
  </div>
</template>
<script>
export default {
  //remain 可视长度 size 每项高度 itmes 数组 variable 每项高度不确定
  props: ['remain', 'size', 'items', 'variable'],
  data() {
    return {
      // 数据开始index
      start: 0,
      // 结束index
      end: this.remain,
      // 偏移量
      offset: 0,
      // 位置信息
      positions: [],
    }
  },
  mounted() {
    //   可视数据的高度
    this.$refs.viewport.style.height = this.size * this.remain + 'px'
    //   实际全部数据的高度
    this.$refs.scrollBar.style.height = this.size * this.items.length + 'px'
    //   缓存每一项的高度
    this.cacheList()
    this.handleScroll()
    this.calcHeight()
  },
  computed: {
    //   前面预留渲染
    prevCount() {
      return Math.min(this.remain, this.start)
    },
    //   后面预留渲染
    nextCount() {
      return Math.min(this.remain, this.items.length - this.end)
    },
    //  可见数据（实际展示的数据）
    visibleData() {
      let start = this.start - this.prevCount
      let end = this.end + this.nextCount
      return this.items.slice(start, end)
    },
  },
  updated() {
    this.calcHeight()
  },
  methods: {
    //   计算每一项的真实高度
    calcHeight() {
      this.$nextTick(() => {
        let nodes = this.$refs.item
        if (!(nodes && nodes.length > 0)) return
        nodes.forEach((node) => {
          let { height } = node.getBoundingClientRect() //真实高度
          let nodeId = node.getAttribute('vid') - 0
          let oldHeight = this.positions[nodeId].height
          let diffHeight = oldHeight - height
          // 高度不同才重新赋值
          if (diffHeight) {
            this.positions[nodeId].height = height
            this.positions[nodeId].bottom -= diffHeight
            for (let i = nodeId + 1; i < this.positions.length; i++) {
              this.positions[i].top = this.positions[i - 1].bottom
              this.positions[i].bottom -= diffHeight
            }
          }
        })

        this.$refs.scrollBar.style.height =
          this.positions[this.positions.length - 1].bottom + 'px'
      })
    },
    cacheList() {
      this.positions = this.items.map((item, index) => ({
        height: this.size,
        top: index * this.size,
        bottom: (index + 1) * this.size,
      }))
    },
    //  根据scrollTop，获取start（因为高度不确定，只能根据实际的每一项的高度来计算，并且使用二分查找，减少性能消耗）
    getStartIndex(value) {
      let start = 0
      let end = this.positions.length - 1
      let tmp = null //最接近的index
      while (start <= end) {
        let middleIndex = parseInt((start + end) / 2)
        let middleVal = this.positions[middleIndex].bottom
        // 因为比较是bottom，所以当value >= bottom的时候，已经是滑过这一项了，直接返回 middleIndex + 1
        if (middleVal == value) {
          return middleIndex + 1
        } else if (middleVal < value) {
          // 上一个if已经比较过middleIndex是否相等了，能走到这里，说明不相等，直接+-1，不再比较当前middleIndex
          start = middleIndex + 1
        } else if (middleVal > value) {
          if (tmp == null || tmp > middleIndex) {
            tmp = middleIndex
          }
          end = middleIndex - 1
        }
      }
      return tmp
    },
    //  滑动时候，根据ScrollTop计算当前应该显示的数据
    handleScroll() {
      let scrollTop = this.$refs.viewport.scrollTop || 0
      if (this.variable) {
        this.start = this.getStartIndex(scrollTop)
        this.end = this.start + this.remain
        this.offset = this.positions[this.start - this.prevCount]
          ? this.positions[this.start - this.prevCount].top
          : 0
      } else {
        this.start = Math.floor(scrollTop / this.size)
        this.end = this.start + this.remain
        this.offset = this.start * this.size - this.prevCount * this.size
      }
    },
  },
}
</script>
<style lang='scss'>
.viewport {
  position: relative;
  overflow-y: scroll;
}
.content {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
}
</style>