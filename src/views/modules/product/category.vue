<template>
  <div>
    <el-tree :data="treeData"
             :props="defaultProps"
             show-checkbox
             node-key="catId"
             :expand-on-click-node="false"
             :default-expanded-keys="expanded_keys"
    >

      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
              type="text"
              size="mini"
              v-if="node.level <= 2"
              @click="() => append(data)">
            Append
          </el-button>
          <el-button
              v-if="node.childNodes.length === 0"
              type="text"
              size="mini"
              @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <!-- 删除确认框 -->
    <el-dialog title="提示"
               :visible.sync="dialogVisible"
               width="30%">
      <el-form :model="categoryForm">
        <el-form-item label="分类名称">
          <el-input v-model="categoryForm.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitCategory">确 定</el-button>
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
      dialogVisible: false,
      expanded_keys: [], // 默认展开的节点
      categoryForm: { name: '' },
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    // 添加节点
    append (data) {
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

    // 提交分类
    submitCategory () {
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
          // 重载树数据
          this.getCategoryTree()
          this.expanded_keys = [this.categoryForm.parentCid]
          this.dialogVisible = false
        } else {
          this.$message({
            message: data.msg,
            type: 'error'
          })
        }
      })
    },

    // 删除节点
    remove: function (node, data) {
      console.log('node==>', node, 'data==>', data)
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
            /* 重载树数据 */
            this.getCategoryTree()
            // 展开当前节点
            const { catId } = node.parent.data
            this.expanded_keys = [catId]
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
