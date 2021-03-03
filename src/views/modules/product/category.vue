<template>
  <div>
    <el-tree :data="treeData" :props="defaultProps" show-checkbox node-key="catId" :expand-on-click-node="false"
             :draggable="true" :allow-drop="allowDrop" @node-drop="nodeDrop" :default-expanded-keys="expanded_keys">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button type="text" size="mini" v-if="node.level <= 2" @click="() => append(data)">
            Append
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(node, data)">
            Edit
          </el-button>
          <el-button v-if="node.childNodes.length === 0" type="text" size="mini" @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <!-- 新增、编辑对话框 -->
    <el-dialog :title="title" :visible.sync="dialogVisible" :close-on-click-modal='false' width="30%">
      <el-form :model="categoryForm">
        <el-form-item label="名称">
          <el-input v-model="categoryForm.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="categoryForm.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="categoryForm.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  name: 'category',
  // import引入的组件需要注入到对象中才能使用
  components: {},
  data () {
    return {
      treeData: [],
      maxLevel: 0, // 拖拽时计算最大层级
      dialogVisible: false, // dialog是否显示
      expanded_keys: [], // 默认展开的节点
      title: '', // 弹窗标题
      actionType: 'add', // add 添加分类； edit 编辑分类
      categoryForm: {
        name: '',
        icon: '',
        productUnit: '',
        sort: 0,
        catId: null
      },
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    // 成功拖动之后触发的事件
    nodeDrop (draggingNode, dropNode, dropType) {
      console.log('tree drop: ', draggingNode, dropNode, dropType)
      let parentNode = dropType === 'inner' ? dropNode : dropNode.parent
      console.log('parentNode==>', parentNode)
      let updateNode = []
      parentNode.childNodes.forEach((node, index) => {
        // 1.更改层级
        //  2.更改排序
        if (node.data.catId === draggingNode.data.catId) {
          updateNode.push({
            catId: node.data.catId,
            catLevel: parentNode.level + 1,
            sort: index,
            parentCid: parentNode.data.catId === undefined ? 0 : parentNode.data.catId
          })
          //  更改拖动节点子项的层级
          this.queryNodeLevel(node, updateNode)
        } else {
          updateNode.push({
            catId: node.data.catId,
            sort: index
          })
        }
      })
      console.log('需要更新的数据==>', updateNode)
      this.$http({
        url: this.$http.adornUrl('/product/category/update/sort'),
        method: 'post',
        data: this.$http.adornData(updateNode, false)
      }).then(({ data }) => {
        console.log('更新层级排序==>', data)
        if (data.code === 0) {
          this.$message({
            type: 'success',
            message: '拖拽成功'
          })
          this.getCategoryTree()
          this.expanded_keys = [parentNode.data.catId]
        } else {
          this.$message({
            type: 'error',
            message: data.msg
          })
        }
      })
    },

    // 递归收集需要更新的子节点层级
    queryNodeLevel (node, updateNode) {
      if (node.childNodes != null && node.childNodes.length > 0) {
        node.childNodes.forEach(node => {
          updateNode.push({
            catId: node.data.catId,
            catLevel: node.level
          })
        })
      }
    },

    // 判断节点能否被放置  可以被放置 return true; 反之 return false
    allowDrop (draggingNode, dropNode, type) {
      this.maxLevel = 0
      // console.log('判断节点能否被放置==>', draggingNode, dropNode, type)
      // 深度 = 最大层级 - 当前节点层级 + 1
      this.getNodeMaxLevel(draggingNode)
      // console.log(' this.maxLevel==>', this.maxLevel)
      let deep = this.maxLevel === 0 ? 1 : (Math.abs(this.maxLevel - draggingNode.level) + 1)
      // console.log('拖动节点深度==>', deep)
      if (type === 'inner') {
        return dropNode.level + deep <= 3
      } else {
        return dropNode.parent.level + deep <= 3
      }
    },

    // 节点最大层级
    getNodeMaxLevel (node) {
      if (node.childNodes != null && node.childNodes.length > 0) {
        node.childNodes.forEach(newNode => {
          if (this.maxLevel < newNode.level) {
            this.maxLevel = newNode.level
          }
          this.getNodeMaxLevel(newNode)
        })
      }
    },

    // 添加节点
    append (data) {
      this.title = '添加'
      this.actionType = 'add'
      this.categoryForm = {
        showStatus: 1,
        catId: null,
        parentCid: data.catId,
        catLevel: data.catLevel + 1,
        sort: 0
      }
      console.log('append==categoryForm==>', this.categoryForm, 'data==>', data)
      this.dialogVisible = true
    },

    // 添加分类
    addCategory () {
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.categoryForm, false)
      }).then(({ data }) => {
        console.log('data222==>', data)
        if (data.code === 0) {
          this.$message({
            message: '保存成功',
            type: 'success'
          })
          this.expanded_keys = [this.categoryForm.parentCid]
          this.closeDialog()
        } else {
          this.$message({
            message: data.msg,
            type: 'error'
          })
        }
      })
    },

    // 提交请求
    submitData () {
      if (this.actionType === 'add') {
        this.addCategory()
      } else if (this.actionType === 'edit') {
        this.editCategory()
      }
    },

    // 编辑节点展示数据
    edit (node, data) {
      console.log('编辑节点==>', data, node)
      this.dialogVisible = true
      this.title = '修改'
      this.actionType = 'edit'
      // 从数据库获取最新数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({ data }) => {
        console.log('获取单个分类==>', data)
        this.categoryForm.name = data.data.name
        this.categoryForm.icon = data.data.icon
        this.categoryForm.productUnit = data.data.productUnit
        this.categoryForm.catId = data.data.catId
        this.categoryForm.parentCid = data.data.parentCid
      })
    },

    // 更新数据
    editCategory () {
      let { name, icon, productUnit, catId } = this.categoryForm
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({ name, icon, productUnit, catId }, false)
      }).then(({ data }) => {
        console.log('更新数据成功==>', data)
        if (data.code === 0) {
          this.$message({
            type: 'success',
            message: `更新${name}成功`
          })
          // 展开节点
          this.expanded_keys = [this.categoryForm.parentCid]
          this.closeDialog()
        } else {
          this.$message({
            type: 'error',
            message: data.msg
          })
        }
      })
    },

    // 关闭窗口时恢复默认数据
    closeDialog () {
      // 重载树数据
      this.getCategoryTree()
      this.dialogVisible = false // dialog是否显示
      this.title = '' // 弹窗标题
      this.actionType = 'add' // add 添加分类； edit 编辑分类
      this.categoryForm = {
        name: '',
        icon: '',
        productUnit: '',
        sort: 0,
        catId: null
      }
    },

    // 删除节点
    remove (node, data) {
      console.log('删除节点==>', node, data)
      const { catId } = data
      this.$confirm(`确认是否删除[${data.name}]分类?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData([catId], false)
        }).then(({ data }) => {
          if (data.code === 0) {
            this.$message({
              message: '删除成功',
              type: 'success'
            })
            // 展开当前节点
            const { catId } = node.parent.data
            this.expanded_keys = [catId]
            this.closeDialog()
          } else {
            this.$message({
              message: data.msg,
              type: 'error'
            })
          }
        })
      }).catch(() => {

      })
    },
    getCategoryTree () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({ data }) => {
        console.log('成功请求树结构', data)
        this.treeData = data.data
      })
    }
  },
  // 计算属性类似于data概念
  computed: {},
  // 监控data中的数据变化
  watch: {},
  // 生命周期-创建完成（可以访问当前this实例）
  created () {
    this.getCategoryTree()
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
.custom-tree-node {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 14px;
  padding-right: 8px;
}
</style>
