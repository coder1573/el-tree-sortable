<template>
  <div class="app-container">
    <!-- default-expand-all 节点要默认展开，否则子节点初始不渲染就会导致获取不到dom元素 -->
    <el-tree :data="treeList" default-expand-all />
  </div>
</template>

<script>
import Sortable from 'sortablejs'

export default {
  name: 'HelloWorld',
  data() {
    return {
      treeList: [{
        id: '1',
        label: '一级 1',
        children: [{id: '1-1', label: '二级 1-1'}, {id: '1-2', label: '二级 1-2'}, {id: '1-3', label: '二级 1-3'}]
      }, {
        id: '2',
        label: '一级 2',
        children: [{id: '2-1', label: '二级 2-1'}, {id: '2-2', label: '二级 2-2'}, {id: '2-3', label: '二级 2-3'}]
      }],
      defaultProps: {
        children: 'children',
        label: 'label'
      },

      // 树形数据的平铺（深度优先）
      flatDataList: [],

      // sortable 对象数组
      sortableObjList: []
    }
  },

  mounted() {
    this.$nextTick(() => {
      this.initSortable()
    })
  },

  methods: {
    initSortable() {
      // 树形数据展开
      const flatDataList = []
      this.flatTree(this.treeList, flatDataList)

      // 给 el-tree role=treeitem 节点添加属性 data-id = unitId
      document.querySelectorAll('div[role=treeitem]').forEach((el, i) => {
        el.setAttribute('data-id', flatDataList[i].id)
      })
      // el-tree 一级节点会有一个多余的dom干扰排序结果，这里 id 标记为 null
      document.querySelector('.el-tree .el-tree__drop-indicator')?.setAttribute('data-id', null)

      // 找到一级节点的父级 role=tree
      const fatherEls = document.querySelectorAll('div[role=tree]') || []
      // 找到全部下级的 el-tree role=group 节点（即 role=treeitem 的父节点）
      const childrenEls = document.querySelectorAll('div[role=group]') || []
      // 合并
      const els = [...fatherEls, ...childrenEls]

      // 创建拖拽对象
      const sortableList = []
      els.forEach((el, i) => {
        sortableList.push(
          new Sortable(el, {
            dataIdAttr: 'data-id',
            ghostClass: 'sortable-ghost',
            dragClass: 'sortable-drag',
            onEnd: (evt) => {
              // 拖拽结束，且位置发生变化
              if (evt.newIndex !== evt.oldIndex) {
                // 获取当前排序后的 id 列表
                const ids = sortableList[i].toArray().filter(t => t !== null)
                console.log('新的排序', ids)
              }
            }
          })
        )
      })
    },

    // 树形数据展开（深度优先）
    flatTree(treeList, result=[]) {
      treeList.forEach(t => {
        result.push(t)
        if (t.children && t.children.length) {
          this.flatTree(t.children, result)
        }
      })
    }
  }
}
</script>

<style>
.sortable-ghost {
  background-color: #C4E1FF;
}
.sortable-drag {
  background-color: #f5f7fa;
}
</style>
<style scoped>
.app-container{
  padding: 0 15px;
}
</style>
