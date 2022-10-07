<template>
  <div>
    <el-tree 
      :data="menus" 
      :props="defaultProps" 
      :expand-on-click-node=false 
      :default-expanded-keys="expandedKey" 
      show-checkbox node-key="catId"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">Append</el-button>
        <el-button v-if="node.childNodes.length==0" type="text" size="mini" @click="() => remove(node, data)">Delete</el-button>
        </span>
      </span>
    </el-tree>
    <!--对话框组件-->
	  <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
      <el-form :model="categroy">
        <el-form-item label="分类名称">
          <el-input v-model="categroy.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      expandedKey: [],
      categroy: { name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0 },
      dialogVisible: false
    }
  },
  methods: {
    handleNodeClick (data) {
      console.log(data)
    },
    // 获取后台数据
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        console.log('成功了获取到菜单数据....', data)
        this.menus = data.data
      })
    },
    // 添加节点
    append (data) {
      console.log('append', data)
      this.dialogVisible = true
      this.categroy.parentCid = data.catId
      this.categroy.catLevel = data.catLevel * 1 + 1
    },
    addCategory () {
      console.log('提交的数据', this.categroy)
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.categroy, false)
      }).then(({ data }) => {
        this.$message({
          message: '添加成功',
          type: 'success'
        })
            // 刷新出新的菜单
        this.getMenus()
            // 设置需要默认展开的菜单
        this.expandedKey = [this.categroy.parentCid]
        this.dialogVisible = false
      })
    },
    // 删除节点
    remove (node, data) {
      var ids = [data.catId]
      this.$confirm(`是否删除【${data.name}】菜单？`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl('/product/category/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            this.$message({
              message: '菜单删除成功',
              type: 'success'
            })
            // 刷新出新的菜单
            this.getMenus()
            // 设置需要默认展开的菜单。删除后给它赋值父节点id，令树形控件刷新后展开
            this.expandedKey = [node.parent.data.catId]
          })
        })
        .catch(() => {})
    }
  },
  // 监听属性 类似于data概念
  computed: {},
  // 监控data中的数据变化
  watch: {},
  // 生命周期 - 创建完成（可以访问当前this实例）
  created () {
    // 创建完成时，就调用getMenus函数
    this.getMenus()
  },
  // 生命周期 - 挂载完成（可以访问DOM元素）
  mounted () {

  },
  beforeCreate () {}, // 生命周期 - 创建之前
  beforeMount () {}, // 生命周期 - 挂载之前
  beforeUpdate () {}, // 生命周期 - 更新之前
  updated () {}, // 生命周期 - 更新之后
  beforeDestroy () {}, // 生命周期 - 销毁之前
  destroyed () {}, // 生命周期 - 销毁完成
  activated () {} // 如果页面有keep-alive缓存功能，这个函数会触发
}
</script>

<style>
</style>