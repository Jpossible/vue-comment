<template>
    <div class="select-list-box" ref="selectbox">
      <ul class="select-list">
        <li
          :class="[ Hindex === index ? 'hover' : '' ]"
          v-for="(item, index) in listData"
          :key="index"
          @click="itemClick(item.name)"
          @mouseenter="itemHover(index)"
        >
          {{item.email}}
        </li>
      </ul>
    </div>
</template>

<script>
export default {
  name: 'JSelectList',
  data () {
    return {
      Hindex: 0,
      listData: [
        {
          name: 'fjj',
          email: 'fjj@163.com'
        },
        {
          name: 'jpossible',
          email: 'jpossible@163.com'
        },
        {
          name: 'jpossible1',
          email: 'jpossible1@163.com'
        }
      ]
    }
  },
  methods: {
    itemClick (name) {
      this.$emit('select', name)
    },
    itemHover (index) {
      this.Hindex = index
    },
    keyCtrlEvent (e) { // which up:38 down:40 enter:13
      switch (e.which) {
        case 40:
          this.Hindex = this.Hindex < this.listData.length - 1 ? ++this.Hindex : 0
          break
        case 38:
          this.Hindex = this.Hindex > 0 ? --this.Hindex : this.listData.length - 1
          break
        case 13:
          this.itemClick(this.listData[this.Hindex].name)
          break
      }
    }
  },
  mounted () {
    document.addEventListener('keydown', this.keyCtrlEvent)
  },
  destroyed () {
    document.removeEventListener('keydown', this.keyCtrlEvent)
  }
}
</script>

<style lang="less">
  .select-list-box{
    border: 1px solid #dadada;
    background-color: #fff;
    .select-list{
      list-style: none;
      margin: 0;
      padding: 0;
    }
    .select-list li{
      height: 40px;
      line-height: 40px;
    }
    .select-list li.hover{
      background-color: #dadada;
      cursor: pointer;
    }
  }
</style>
