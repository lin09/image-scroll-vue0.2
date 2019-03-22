<template>
  <div class="image-scroll">
    <div class="images"
      :class="{ move: move, 'up-move': upMove, 'no-animation': noAnimation }"
      :style="{ width: items.length + '00%' }"
      ref="images">
      <a class="image" v-for="(item, i) in items" :key="i" :style="{ order: item.order, height: height }" :href="item.href">
        <img :src="item.imgUrl"/>
      </a>
    </div>
    <div v-if="pagination" class="pagination">
      <div class="point" v-for="(item, i) in items" :key="i" :class="{ active: i === index }"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ImageScroll',
  props: {
    images: Array,
    height: String,
    pagination: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      items: [ ...this.images ],
      // 当前显示项的下标
      index: 0,
      // 标记停止自动滚动
      isStop: false,
      // 用于滚动动画切换，与 flex order 配置使用
      move: true,
      // 回滚
      upMove: false,
      // 去掉动画，例如触摸定位使用
      noAnimation: false
    }
  },
  created() {
    // 修正排序
    this.correct()
  },
  mounted () {
    if (this.images.length < 2) {
      this.move = false
      return
    }

    // 启动自动滚动
    this.start.timeout = setTimeout(this.start, 3000)

    // 触摸开始X坐标
    let startX
    // 开始触摸
    const touchstart = evt => {
      // 停止全部等待
      this.stopTime()
      // 标记停止
      this.isStop = true
      // 去掉动画
      this.noAnimation = true
      // 记录坐标
      startX = evt.changedTouches[0].pageX
    }

    // 结束触摸
    const touchend = evt => {
      // 停止触摸事件
      removeEvent()
      // 停止全部等待
      this.stopTime()

      if (evt.changedTouches[0].pageX <= startX) {
        this.move = false
        this.$refs.images.style.transform = `translateX(${ evt.changedTouches[0].pageX - startX }px)`
          this.index ++
          if (this.index >= this.items.length) {
            this.index = 0
          }
          this.correct()
          .then(() => {
            // 还原动画
            this.noAnimation = false
          this.move = true
          this.$refs.images.style.transform = ``
            // 标记不停止
            this.isStop = false
            // 设置等待下一个滚动
            this.start.timeout = setTimeout(this.start, 3000)
            // 启动触摸事件
            addEvent()
          })
      } else {
        // 还原动画
        this.noAnimation = false
        // 去掉 style.transform ，这里不用停止动画，因为跟样式 up-move 一样值
        this.$refs.images.style.transform = ''
        // 调用滚动回上一个
        this.up()
        // 启动触摸事件
        .then(addEvent)
      }
    }

    // 触摸跟随
    const touchmove = evt => {
      // 停止全部等待
      this.stopTime()
      // 标记停止
      this.isStop = true
      // 去掉动画
      this.noAnimation = true
      this.$refs.images.style.transform = `translateX(${ evt.changedTouches[0].pageX - startX - 750 }px)`
    }

    const addEvent = () => {
      this.$refs.images.addEventListener('touchstart', touchstart)
      this.$refs.images.addEventListener('touchend', touchend)
      this.$refs.images.addEventListener('touchmove', touchmove)
    }
    const removeEvent = () => {
      this.$refs.images.removeEventListener('touchstart', touchstart)
      this.$refs.images.removeEventListener('touchend', touchend)
      this.$refs.images.removeEventListener('touchmove', touchmove)
    }
    addEvent()
  },
  methods: {
    // 停止全部等待
    stopTime () {
      clearTimeout(this.correct.timeout)
      clearTimeout(this.start.timeout)
      clearTimeout(this.nextOrder.timeout)
      clearTimeout(this.up.timeout)
    },
    // 修正排序
    correct () {
      // 停止全部等待
      this.stopTime()
      return new Promise(resolve => {
        // 当前排序为1
        this.items[this.index].order = 1

        // 排在当前左边的
        let before = this.index - 1
        if (before < 0) {
          // 最后一个排到前面时，排序要比当前小
          before = this.items.length - 1
          this.items[before].order = 0
        } else {
          // 其它跟当前一样
          this.items[before].order = 1
        }

        this.items.forEach((item, index) => {
          if (index === before || index === this.index) {
            return
          }
          if (index > this.index) {
            // 大于当前为1，跟当前一样
            item.order = 1
          } else {
            // 小于当前时，排序值要比当前大
            item.order = 2
          }
        })
        // Vue 不能检测变动的数组时，用新数组替换旧数组
        this.items = [].concat(this.items)
        // 设置等待
        this.correct.timeout = setTimeout(resolve, 100)
      })
    },
    // 开始自动滚动
    start () {
      // 停止全部等待
      this.stopTime()

      if (this.isStop) {
        return
      }

      // 把滚动完成的调到最后
      this.nextOrder()
      .then(() => {
        // 开始滚动下一个
        this.index ++
        if (this.index >= this.items.length) {
          this.index = 0
        }
        this.move = true

        this.start.timeout = setTimeout(this.start, 3000)
      })
    },
    // 把滚动完成的调到最后
    nextOrder () {
      // 停止全部等待
      this.stopTime()
      return new Promise(resolve => {
        // 修正排序
        this.correct()
        .then(() => {
          // 滚动完成的下标
          let before = this.index - 1
          if (before < 0) {
            before = this.items.length - 1
          }
          // 把滚动完成的调到最后，排序调到最后同时去掉move（抵消位差）
          this.items[before].order ++
          this.move = false

          // Vue 不能检测变动的数组时，用新数组替换旧数组
          this.items = [].concat(this.items)
          // 设置等待
          this.nextOrder.timeout = setTimeout(resolve, 100)
        })
      })
    },
    // 回滚上一个
    up () {
      // 停止全部等待
      this.stopTime()
      return new Promise (resolve => {
        // 回滚位置
        this.upMove = true
        // 还原动画
        this.noAnimation = false

        this.up.timeout = setTimeout(() => {
          // 回滚完成，修改下标
          this.index --
          if (this.index < 0) {
            this.index = this.items.length - 1
          }
          // 去掉回滚位置同时修正排序（抵消位差），去掉动画，使过程无动画影响
          this.upMove = false
          this.noAnimation = true
          this.correct()
          .then(() => {
            // 还原动画
            this.noAnimation = false
            // 重新自动滚动
            this.isStop = false
            this.start.timeout = setTimeout(this.start, 3000)
            resolve()
          })
        }, 500)
      })
    }
  }
}
</script>

<style scoped>
.image-scroll {
  position: relative;
  width: 750px;
  overflow: hidden;
}
.images {
  display: flex;
  transform: translateX(0);
}
.move {
  transition: transform 500ms ease;
  transform: translateX(-750px);
}
.up-move {
  transform: translateX(0);
}
.no-animation {
  transition: none;
}
.image {
  padding: 0 24px;
  width: 750px;
  height: 292px;
}
img {
  width: 100%;
  height: 100%;
}
.pagination {
  position: absolute;
  bottom: 20px;
  display: flex;
  justify-content: center;
  height: 4px;
  width: 100%;
}
.point {
  margin: 0 5px;
  width: 30px;
  height: 4px;
  background-color: rgba(255, 255, 255, 0.3);
}
.point.active {
  background-color: rgba(255, 255, 255, 0.6);
}
</style>
