<template>
  <div class="input-area" @mouseleave="leaveRange">
    <div
      ref="inputBox"
      class="comment-range"
      contenteditable="true"
      @click="clickEvent"
      @keydown="keyDownEvent"
      @keyup="keyUpEvent"
      @input="inputEvent"
      @blur="blurEvent"
    ></div>
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
      preNode: null,
      keyCtrl: [37, 38, 39, 40],
      className: 'at-sign',
      timeout: false
    }
  },
  methods: {
    _getCurRange () {
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
      if (ele.previousSibling && ele.previousSibling.innerHTML !== '') {
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
      if (ele.previousSibling) {
        let nodeValue = ele.previousSibling.textContent
        nodeValue = nodeValue.slice(0, nodeValue.length - 1)
        let newNode = nodeValue ? document.createTextNode(nodeValue) : document.createElement('span')// 上一个节点无内容时会导致无法删除BUG 创建新文本节点
        this.$refs.inputBox.replaceChild(newNode, ele.previousSibling)
      }
    },
    stopDefault (e) {
      e.preventDefault()
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
      if (this.isFireFox) {
        if (this.keyCtrl.includes(e.which)) {
          this.timeout = true
        }
        if (this.timeout && !this.keyCtrl.includes(e.which)) {
          e.preventDefault()
          return
        }
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
      if (!this.showList) {
        if (e.which === 13) {
          e.preventDefault()
        } else if (this.keyCtrl.includes(e.which)) {
          this._getCurRange()
        }
      } else {
        this._getCurRange()
        e.preventDefault()
      }
    },
    keyUpEvent (e) {
      if (this.isFireFox) {
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
      if (e && e.inputType === 'deleteContentBackward' && this.isFireFox) {
        this.fireFoxDel(e)
      }
    },
    fireFoxDel (e) {
      if (getSelection().anchorNode.parentNode.className === this.className) {
        this.$refs.inputBox.removeChild(getSelection().anchorNode.parentNode)
        e.preventDefault()
      }
    },
    atEvent (e) { // 用户选择相关操作
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
    .comment-range{
      min-height: 30px;
      min-width: 100px;
      padding: 10px;
      border: 1px solid #dadada;
      word-break: break-word;
    }
    .at-sign{
      color: blue;
    }
  }
</style>
