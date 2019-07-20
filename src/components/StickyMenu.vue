<template>
  <div class="cpt-anchor">
    <el-radio-group v-model="menu"
                    size="mini"
                    @change="handleMenuChange">
      <el-radio-button v-for="menu in menus"
                       :key="menu.value"
                       :label="menu.value">
        {{ menu.label }}
      </el-radio-button>
    </el-radio-group>
  </div>
</template>

<script>
import { tween } from 'shifty'
import { get as _get } from 'noshjs'
import { getFirstScrollElement } from '@/util/index.js'

export default {
  props: {
    top: {
      type: Number,
      default: 0
    },
    menus: {
      type: Array
    },
    data: {
      type: Array
    },
    parent: {
      type: String,
      default: ''
    }
  },

  data () {
    return {
      menu: '',
      isScroll: true,
      isMounted: false,
      scrollTop: 0,
      anchorChange: false,
      rootScrollElement: ''
    }
  },

  mounted () {
    this.isMounted = true
    this.getScrollElement()
  },

  watch: {
    parent: {
      immediate: true,
      handler: 'getScrollElement'
    },

    menus: {
      immediate: true,
      handler (list) {
        this.menu = _get(list, [0, 'prop'], '')
      }
    },

    scrollTop (v) {
      if (this.anchorChange) {
        // 切换按钮会滚动视图，$nextTick 之后按钮值改变了，但滚动可能还没有结束，所以需要打个标记。
        this.isScroll = true
      }
    }
  },

  methods: {
    handleMenuChange (select) {
      this.isScroll = false
      this.anchorChange = false
      // 滚动高度等于元素距离可视区头部高度减去元素自身高度与元素上边框高度以及滚动区距离可视区头部的高度。
      const scrollElement = document.querySelector(select)
      if (scrollElement && this.rootScrollElement) {
        const offsetTop = scrollElement.offsetTop + scrollElement.clientTop
        const offsetHeight = _get(
          this.$el,
          ['parentElement', 'offsetHeight'],
          0
        )
        const top = offsetTop - this.top - offsetHeight

        // 做一个缓动处理
        tween({
          from: { x: this.rootScrollElement.scrollTop },
          to: { x: top },
          duration: 500,
          easing: 'easeOutQuint',
          step: ({ x }) => {
            this.rootScrollElement.scrollTop = x
          }
        }).then(({ x }) => {
          this.rootScrollElement.scrollTop = x
        })

        this.$nextTick(() => {
          this.anchorChange = true
        })
      }
    },

    getScrollElement () {
      if (!this.isMounted) return
      // 如果没有传入 parent 默认取第一个父级滚动元素
      const parent = this.parent
      let element = null
      if (parent) {
        element = document.querySelector(parent)
        // mount 之后 rootScrollElement 可能已经存在了，如果和上次一样就不做任何操作。
        if (element === this.rootScrollElement) return
      } else if (this.$el) {
        element = getFirstScrollElement(this.$el.parentElement)
      }
      if (element) {
        this.removeScrollEvent()
        this.rootScrollElement = element
        this.rootScrollElement.addEventListener('scroll', this.handleScroll)
      }
    },

    removeScrollEvent () {
      if (this.rootScrollElement) {
        this.rootScrollElement.removeEventListener('scroll', this.handleScroll)
      }
    },

    handleScroll (event) {
      const scrollTop = this.rootScrollElement.scrollTop
      this.scrollTop = scrollTop
      if (!this.isScroll) return
      const { data, top } = this
      // const offsetHeight = _get(this.$el, ['parentElement', 'offsetHeight'], 0)
      const scrollList = []
      data.forEach(item => {
        const element = document.querySelector(item.prop)
        if (element) {
          // const top = element.offsetTop
          const rect = element.getBoundingClientRect()
          scrollList.push(rect)
        }
      })
      // 遍历按钮元素的 top 和 bottom，查看当前滚动在那个元素的区间内。
      scrollList.some((it, index) => {
        if (index && scrollTop >= it.top && top < it.bottom) {
          const menu = _get(data, [index, 'prop'], '')
          if (menu) this.menu = menu
          return true
        } else {
          // 当小于最小高度时，就等于最小高度
          if (scrollTop >= 0 && scrollTop < it.bottom) {
            const menu = _get(data, [index, 'prop'], '')
            if (menu) this.menu = menu
            return true
          }
        }
      })
    }
  }
}
</script>

<style lang="less">
.cpt-anchor {
  padding-top: 4px;
  .cpt-tab-menus {
    margin: 0;
    .el-radio-button {
      margin-left: 10px;
      .el-radio-button__inner {
        border: none;
        border-radius: 5px 5px 0 0;
        border-bottom: 2px solid #e4e7ed;
        background-color: #f6f6f8;
        font-size: 16px;

        &:hover {
          border-bottom: 2px solid #409eff;
        }
      }

      &.is-active {
        .el-radio-button__inner {
          color: #fff;
          border: none;
          border-radius: 5px 5px 0 0;
          background-color: #409eff;
          border-bottom: 2px solid #409eff;
          box-shadow: none;
        }
      }
    }
  }
}
</style>
