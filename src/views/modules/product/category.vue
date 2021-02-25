<template>
  <el-tree :data="treeData"
           :props="defaultProps"
           @node-click="handleNodeClick">

  </el-tree>
</template>

<script>
export default {
  name: 'category',
  // import引入的组件需要注入到对象中才能使用
  components: {},
  data () {
    return {
      treeData: [{
        label: '一级 1',
        children: [{
          label: '二级 1-1',
          children: [{
            label: '三级 1-1-1'
          }]
        }]
      }, {
        label: '一级 2',
        children: [{
          label: '二级 2-1',
          children: [{
            label: '三级 2-1-1'
          }]
        }, {
          label: '二级 2-2',
          children: [{
            label: '三级 2-2-1'
          }]
        }]
      }, {
        label: '一级 3',
        children: [{
          label: '二级 3-1',
          children: [{
            label: '三级 3-1-1'
          }]
        }, {
          label: '二级 3-2',
          children: [{
            label: '三级 3-2-1'
          }]
        }]
      }],
      defaultProps: {
        children: 'children',
        label: 'label'
      }
    }
  },
  methods: {
    handleNodeClick (data) {
      console.log(data)
    }
  },
  // 计算属性类似于data概念
  computed: {},
  // 监控data中的数据变化
  watch: {},
  // 生命周期-创建完成（可以访问当前this实例）
  created () {
  },
  // 生命周期-挂载完成（可以访问DOM元素）
  mounted () {
  },
  // 生命周期-创建之前
  beforeCreate () {
  },
  // 生命周期-挂载之前
  beforeMount () {
  },
  // 生命周期-更新之前
  beforeUpdate () {
  },
  // 生命周期-更新之后
  updated () {
  },
  // 生命周期-销毁之前
  beforeDestroy () {
  },
  // 生命周期-销毁完成
  destroyed () {
  },
  // 如果页面有keep-alive缓存功能，这个函数会触发
  activated () {
  }
}
</script>

<style scoped>

</style>
