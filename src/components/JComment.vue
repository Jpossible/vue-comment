<template>
  <div class="input-area" @mouseleave="leaveRange">
    <div
      ref="inputBox"
      class="comment-range"
      contenteditable="true"
      @click="clickEvent"
      @keydown="keyDownEvent"
      @input="inputEvent"
      @keyup="keyUpEvent"
      @blur="blurEvent"
    ></div>
    <div class="input-placeholder" v-show="isPlaceholder">
      {{placeholder}}
    </div>
    <j-select-list @select="insertNode" v-if="showList"/>
  </div>
</template>

<script>
import JSelectList from '@/components/JSelectList'
export default {
  name: 'JComment',
  components: {
    JSelectList
  },
  data () {
    return {
      isFireFox: navigator.userAgent.indexOf('Firefox') >= 0,
      inputRange: null,
      showList: false,
      timeout: false,
      keyCtrl: [37, 38, 39, 40],
      className: 'at-sign',
      placeholder: '请输入些什么11111111111111111111111111111111111',
      isPlaceholder: true
    }
  },
  methods: {
    _getCurRange () { // 获取光标所在精确位置
      this.inputRange = getSelection().getRangeAt(0) // 获取选区开始的节点
    },
    _toNodeEnd (ele) {
      let range = getSelection()
      if (ele.nextSibling) {
        range.selectAllChildren(ele.nextSibling) // 若插入元素后有其他元素则光标定位到插入元素后部
      } else {
        range.selectAllChildren(this.$refs.inputBox) // 选择范围内所有的元素
        range.collapseToEnd() // 收缩选区到最后一个元素后边
      }
    },
    _toNodeBef (ele) {
      let range = getSelection()
      if (ele && ele.previousSibling && ele.previousSibling.innerHTML !== '') {
        range.selectAllChildren(ele)
        range.collapseToStart()
        range.modify('move', 'backward', 'character')
        range.modify('move', 'forward', 'character')
      } else {
        range.selectAllChildren(this.$refs.inputBox)
        range.collapseToStart()
      }
    },
    _delNodeBef (ele) {
      if (ele && ele.previousSibling) {
        let nodeValue = ele.previousSibling.textContent
        nodeValue = nodeValue.slice(0, nodeValue.length - 1)
        let newNode = nodeValue ? document.createTextNode(nodeValue) : document.createElement('span')// 上一个节点无内容时会导致无法删除BUG 创建新文本节点
        this.$refs.inputBox.replaceChild(newNode, ele.previousSibling)
      }
    },
    createElement (name) {
      let ele = document.createElement('span')
      ele.innerText = '@' + name
      ele.contentEditable = this.isFireFox
      ele.className = this.className
      return ele
    },
    insertNode (name) {
      if (!this.inputRange) { return }
      let ele = this.createElement(name)
      this.inputRange.insertNode(ele)
      this._delNodeBef(ele)
      this._toNodeEnd(ele)
      this.showList = false
    },
    clickEvent (e) {
      if (e.target.className === this.className) { // 对单击已选标签进行光标处理
        this._toNodeEnd(e.target)
      }
      this._getCurRange() // 记录当前光标的偏移量
      this.showList = false
    },
    blurEvent () {
      this._getCurRange() // 记录失去焦点时光标的偏移量
    },
    keyDownEvent (e) {
      if (this.showList) { // 在唤出用户选择框时屏蔽输入框原本的键盘事件
        this._getCurRange()
        e.preventDefault()
        return
      }
      if (this.isFireFox) { // 火狐兼容 在火狐下按下方向键盘切换焦点位置时需上锁
        if (this.keyCtrl.includes(e.which)) {
          this.timeout = true
        }
        if (this.timeout && !this.keyCtrl.includes(e.which)) {
          e.preventDefault()
          return
        }
      }
      if (e.which === 13) { // 阻止输入框回车换行
        e.preventDefault()
      } else if (this.keyCtrl.includes(e.which)) {
        this._getCurRange()
      }
      if ((e.which === 229 && e.shiftKey === true) || (e.which === 50 && e.shiftKey === true)) {
        this.atEvent()
      }
      if (e.which === 8) {
        if (getSelection().anchorNode.previousSibling &&
            getSelection().anchorNode.previousSibling.contentEditable &&
            getSelection().anchorNode.previousSibling.contentEditable === 'false') {
          this.$refs.inputBox.removeChild(getSelection().anchorNode.previousSibling)
          e.preventDefault()
        }
      }
    },
    keyUpEvent (e) {
      if (e.which === 8 &&
        (this.$refs.inputBox.innerText === '' || this.$refs.inputBox.innerText === '\n')) { // 火狐/n兼容
        this.$refs.inputBox.innerHTML = ''
        this.isPlaceholder = true
      }
      if (this.isFireFox) { // 火狐兼容 在火狐下抬起方向键盘切换焦点位置时需解锁
        if (this.keyCtrl.includes(e.which)) {
          this.timeout = false
        }
        if (e.which === 37 || e.which === 40) {
          if (getSelection().anchorNode.parentNode.className === this.className) {
            this._toNodeBef(getSelection().anchorNode.parentNode)
          }
        }
        if (e.which === 38 || e.which === 39) {
          if (getSelection().anchorNode.parentNode.className === this.className) {
            this._toNodeEnd(getSelection().anchorNode.parentNode)
          }
        }
      }
    },
    inputEvent (e) {
      if (this.isPlaceholder) {
        this.isPlaceholder = false
      }
      if (this.isFireFox && e.inputType === 'deleteContentBackward') {
        this.fireFoxDel(e)
      }
    },
    fireFoxDel (e) {
      if (getSelection().anchorNode.parentNode.className === this.className) {
        this.$refs.inputBox.removeChild(getSelection().anchorNode.parentNode)
        e.preventDefault()
      }
    },
    atEvent () { // 用户选择相关操作
      this.showList = true
    },
    leaveRange () {
      this.showList = false
    }
  }
}
</script>

<style lang="less">
  .input-area{
    position: relative;
    min-width: 150px;
    border: 1px solid #dadada;
    border-radius: 4px;
    font-size: 16px;
    vertical-align: middle;
    word-break: break-all;
    .comment-range{
      height: 20px;
      padding: 5px;
      outline: none;
    }
    .input-placeholder{
      position: absolute;
      padding: 5px;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      z-index: -1;
      color: #a7a5a5;
      overflow: hidden;
    }
    .at-sign{
      color: blue;
    }
  }
</style>
