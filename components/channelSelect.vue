<template>
  <div>
    <div id="copyIt" style="height: 0; opacity: 0">{{ copyValue }}</div>
    <div :class="`box${fid} box`" @mousedown="handleMouseDown">
      <div
        :class="`mask${fid} maskBox`"
        v-show="is_show_mask"
        :style="
          'width:' +
          mask_width +
          'left:' +
          mask_left +
          'height:' +
          mask_height +
          'top:' +
          mask_top
        "
      ></div>
      <span
        v-for="(item, index) in data_list"
        :key="index"
        :class="`checkboxItem${fid} itemSpan`"
      >
        <input :class="` input${item.id}`" type="checkbox" v-if="item.name" />
        <span class="inputLabel">{{
          item.name ? `${item.name}-${item.value}` : ''
        }}</span>

        <span v-if="!!item.name" @click="copyIt(item.value)" :class="isMove?'action cur':'action'">复制</span>
        <span v-if="!!item.name"  :class="isMove?'action cur':'action'" @click="reurnChannel(item)">转移</span>
        <span v-if="!!item.name"  :class="isMove?'action cur':'action'" @click="editChannel(item)">编辑</span>
      </span>
      <div :class="isMove ? 'moveClass' : 'moveClass'"></div>
    </div>
    <p
      style="text-align: center; color: #409eff; cursor: pointer"
      @click="loadMoreSubChannel()"
      v-if="!temSubChannel.isall"
    >
      加载更多
    </p>
    <p style="text-align: center" v-if="temSubChannel.isall">没有数据了</p>
  </div>
</template>

<script>
import { selectText } from '@/lib/util'
export default {
  props: ['oriDatas', 'curDatas', 'moreListNum'],
  data: function () {
    return {
      copyValue: '',
      loadNum: this.moreListNum,
      visible: false,
      checkAll: false,
      checkedCities: [],
      data_list: [],
      temSubChannel: { ...this.curDatas },
      fid: this.oriDatas.id,
      isIndeterminate: true,
      is_show_mask: false,
      box_screen_left: 0,
      box_screen_top: 0,
      start_x: 0,
      start_y: 0,
      end_x: 0,
      end_y: 0,
      scrollTop: 0,
      // mask_left: '0px',
      // mask_top: '0px',
      isMove: false,
    }
  },
  updated() {
    console.log('update')
  },
  created() {
    console.log('初始化')
    this.restList()
  },
  mounted() {
    $('body').on('click', '.inputLabel', function () {
      $(this).prev()[0].checked = !$(this).prev()[0].checked
    })
    this.resetDomBox()
    $(window).scroll(() => {
      this.resetDomBox()
    })
    // setTimeout(() => {
    this.temSubChannel.slected.forEach((x) => {
      document.querySelector(`.input${x}`).checked = true
    })
    // }, 100)
    // $('body').on('mousedown','.itemSpan',function(){
    //   $(this).addClass('customClass')
    // })
  },
  computed: {
    mask_width() {
      return `${Math.abs(this.end_x - this.start_x)}px;`
    },
    mask_height() {
      return `${Math.abs(this.end_y - this.start_y)}px;`
    },
    mask_left(){
      return `${Math.min(this.start_x, this.end_x)}px;`
    },
    mask_top(){
      return `${Math.min(this.start_y, this.end_y)}px;`
    }
  },
  methods: {
    restList() {
      this.data_list = this.oriDatas.subchannel.slice(
        0,
        this.temSubChannel.ind * this.loadNum
      )
      this.temSubChannel.isall = this.oriDatas.subchannel.length <= this.loadNum
    },
    copyIt(d) {
      this.copyValue = d
      setTimeout(() => {
        selectText('copyIt')
        this.$message.success('复制成功!')
      }, 100)
    },
    editChannel(d) {
      this.$emit('editChannel', d, this.fid)
    },
    reurnChannel(d) {
      this.$emit('reurnChannel', this.fid, d)
    },
    loadMoreSubChannel() {
      this.temSubChannel.ind++
      this.data_list = this.oriDatas.subchannel.slice(
        0,
        this.temSubChannel.ind * this.loadNum
      )
      this.temSubChannel.isall =
        this.data_list.length == this.oriDatas.subchannel.length

      this.$emit('syncDatas', this.temSubChannel, this.fid)
    },
    resetDomBox() {
      const dom_box = document.querySelector('.box' + this.fid)
      if (!dom_box) return
      this.box_screen_left = dom_box.getBoundingClientRect().left
      this.box_screen_top =
        dom_box.getBoundingClientRect().top -
        (document.body.scrollTop || document.documentElement.scrollTop)
    },
    handleCheckAllChange(val) {
      this.checkedCities = val ? cityOptions : []
      this.isIndeterminate = false
    },
    handleCheckedCitiesChange(value) {
      let checkedCount = value.length
      this.checkAll = checkedCount === this.data_list.length
      this.isIndeterminate =
        checkedCount > 0 && checkedCount < this.data_list.length
    },
    handleMouseDown(event) {
      this.scrollTop =
        document.body.scrollTop || document.documentElement.scrollTop

      this.is_show_mask = true
      this.start_x = event.offsetX
      this.start_y = event.offsetY

      this.end_x = event.offsetX
      this.end_y = event.offsetY

      const dom_box = document.querySelector('.box' + this.fid)
      if (!dom_box) return
      this.box_screen_left = dom_box.getBoundingClientRect().left
      this.box_screen_top = dom_box.getBoundingClientRect().top

      // this.mask_left = `${Math.min(this.start_x, this.end_x)}px;`

      // this.mask_top = `${Math.min(this.start_y, this.end_y)}px;`

      document
        .querySelector(`.box${this.fid}`)
        .addEventListener('mousemove', this.handleMouseMove)
      document
        .querySelector(`.box${this.fid}`)
        .addEventListener('mouseup', this.handleMouseUp)
    },
    handleMouseMove(event) {
      this.isMove = true
      const scrollHeight =
        document.body.scrollTop || document.documentElement.scrollTop
      const diffHeight = Math.abs(this.scrollTop - scrollHeight)

      this.end_x = event.offsetX
      this.end_y = event.offsetY
    },
    handleMouseUp() {
      this.isMove = false
      document
        .querySelector(`.box${this.fid}`)
        .removeEventListener('mousemove', this.handleMouseMove)
      document
        .querySelector(`.box${this.fid}`)
        .removeEventListener('mouseup', this.handleMouseUp)
      this.is_show_mask = false
      this.handleDomSelect()
      this.resSetXY()
    },
    handleDomSelect() {
      // if (
      //   Math.abs(this.start_x - this.end_x) < 3 ||
      //   Math.abs(this.start_y - this.end_y) < 3
      // )
      //   return
      const dom_mask = window.document.querySelector('.mask' + this.fid)
      const rect_select = dom_mask.getClientRects()[0]
      const add_list = []
      const del_list = []
      document
        .querySelectorAll('.checkboxItem' + this.fid)
        .forEach((node, index) => {
          const rects = node.getClientRects()[0]

          if (this.collide(rects, rect_select) === true) {
            if (this.temSubChannel.slected.includes(this.data_list[index].id)) {
              del_list.push(this.data_list[index].id)
              document.querySelector(
                `.input${this.data_list[index].id}`
              ).checked = false
            } else {
              document.querySelector(
                `.input${this.data_list[index].id}`
              ).checked = true
              add_list.push(this.data_list[index].id)
            }
          }
        })
      this.temSubChannel.slected = this.temSubChannel.slected
        .concat(add_list)
        .filter((item) => !del_list.includes(item))
      this.$emit('syncDatas', this.temSubChannel, this.fid)
    },
    // eslint-disable-next-line class-methods-use-this
    collide(rect1, rect2) {
      const maxX = Math.max(rect1.x + rect1.width, rect2.x + rect2.width)
      const maxY = Math.max(rect1.y + rect1.height, rect2.y + rect2.height)
      const minX = Math.min(rect1.x, rect2.x)
      const minY = Math.min(rect1.y, rect2.y)
      if (
        maxX - minX <= rect1.width + rect2.width &&
        maxY - minY <= rect1.height + rect2.height
      ) {
        return true
      } else {
        return false
      }
    },
    resSetXY() {
      this.start_x = 0
      this.start_y = 0
      this.end_x = 0
      this.end_y = 0
    },
  },
}
</script>

    <style scoped lang="less">
.moveClass {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
   background:rgba(62, 158, 255, 0.1);
  // pointer-events:none;
}
.box {
  width: 100%;
  padding: 20px;
  position: relative;
  overflow: hidden;
  user-select: none;
}

.box .maskBox {
  position: absolute;
  background: #409eff;
  opacity: 0.4;
  z-index: 999999999;
}
.itemSpan {
  display: inline-table;
  width: 33%;
  text-align: left;
  padding:10px 0;
  font-size: 14px;
  &.cur {
    position: relative;
    z-index: 0;
  }
}
.action {
  font-size: 14px;
  cursor: pointer;
  margin-left: 5px;
  position: relative;
  z-index: 9;
  &.cur {
    position: inherit;
    direction: none;
  }
  &:hover {
    color: #409eff;
  }
}
</style>