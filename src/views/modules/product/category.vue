<!--
  功能：功能描述
  作者：chenyawei
  作者：chenyawei1227@163.com
  时间：2020年10月15日 14:06:55
  版本：v1.0
  修改记录：
  修改内容：
  修改人员：
  修改时间：
-->
<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      @node-click="handleNodeClick"
      show-checkbox
      node-key="catId"
      :draggable="true"
      :expand-on-click-node="false"
      :default-expanded-keys="expanedKeys">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="edit(data)">
            Edit
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标地址">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
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
// 这里可以导入其他文件（比如：组件，工具js, 第三方插件js, json文件, 图片文件等）
// 例如： import 《组件名称》 from 《组件路径》
export default {
  // 组件名称
  name: 'demo',
  // 组件参数 接收来自父组件的数据
  props: {},
  // 局部注册的组件
  components: {},
  // 组件状态值
  data () {
    return {
      title: '',
      dialogType: '', // 添加：add , 修改：edit
      category: {name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0, catId: null, icon: '', productUnit: ''},
      dialogVisible: false,
      menus: [],
      expanedKeys: [],
      defaultProps: {
        children: 'childrens',
        label: 'name'
      }
    }
  },
  methods: {
    handleNodeClick (data) {
      console.log(data)
    },
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({ data }) => {
        console.log('获取树形菜单列表成功：', data.data)
        this.menus = data.data
      })
    },
    submitData() {
      if (this.dialogType === 'add') {
        this.addCategory();
      }
      if (this.dialogType === 'edit') {
        this.editCategory();
      }
    },
    append(data) {
      console.log('新增的数据', data);
      this.title = '添加分类';
      this.dialogVisible = true;
      this.dialogType = 'add';
      this.category.name = '';
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.icon = '';
      this.category.productUnit = '';
      this.category.sort = 0;
      this.category.showStatus = 1;
    },
    // 添加三级分类
    addCategory() {
      console.log('category', this.category);
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
        this.$message({
          message: '新增菜单成功',
          type: 'success'
        });
        // 关闭对话框
        this.dialogVisible = false;
        // 刷新菜单
        this.getMenus();
        // 设置默认展开菜单
        this.expanedKeys = [this.category.parentCid];
      })
    },
    // 修改菜单
    edit(data) {
      console.log('要修改的数据：', data);
      this.title = '修改分类';
      this.dialogVisible = true;
      this.dialogType = 'edit';
      // 发送请求获取分类信息
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({data}) => {
        console.log('要回显的方法：', data);
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
      })
    },
    // 提交修改
    editCategory() {
      console.log('edit', this.category);
      var { catId, name, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({ catId, name, icon, productUnit }, false)
      }).then(({data}) => {
        this.$message({
          message: '修改菜单成功',
          type: 'success'
        });
        // 关闭对话框
        this.dialogVisible = false;
        // 刷新菜单
        this.getMenus();
        // 设置默认展开菜单
        this.expanedKeys = [this.category.parentCid];
      })
    },
    // 删除菜单
    remove(node, data) {
      console.log('remove', node, data);
      var ids = [data.catId];
      this.$confirm(`是否确定删除【{data.name}】菜单, 是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          this.$message({
            message: '菜单删除成功',
            type: 'success'
          });
          // 刷新菜单
          this.getMenus();
          // 设置需要默认需要展开的菜单
          this.expanedKeys = [node.parent.data.catId];
        })
      }).catch(() => {
      })
    }
  },
  // 计算属性
  computed: {},
  // 侦听器
  watch: {},
  // 以下是生命周期钩子   注：没用到的钩子请自行删除
  /**
   * 在实例初始化之后，组件属性计算之前，如data属性等
   */
  beforeCreate () {},
  /**
   * 组件实例创建完成，属性已绑定，但DOM还未生成，$ el属性还不存在
   */
  created () {
    this.getMenus()
  },
  /**
   * 在挂载开始之前被调用：相关的 render 函数首次被调用。
   */
  beforeMount () {},
  /**
   * el 被新创建的 vm.$ el 替换，并挂载到实例上去之后调用该钩子。
   * 如果 root 实例挂载了一个文档内元素，当 mounted 被调用时 vm.$ el 也在文档内。
   */
  mounted () {},
  /**
   * 数据更新时调用，发生在虚拟 DOM 重新渲染和打补丁之前。
   * 你可以在这个钩子中进一步地更改状态，这不会触发附加的重渲染过程。
   */
  beforeUpdate () {},
  /**
   * 由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子。
   * 当这个钩子被调用时，组件 DOM 已经更新，所以你现在可以执行依赖于 DOM 的操作。
   */
  updated () {},
  /**
   * keep-alive 组件激活时调用。 仅针对keep-alive 组件有效
   */
  activated () {},
  /**
   * keep-alive 组件停用时调用。 仅针对keep-alive 组件有效
   */
  deactivated () {},
  /**
   * 实例销毁之前调用。在这一步，实例仍然完全可用。
   */
  beforeDestroy () {},
  /**
   * Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，
   * 所有的事件监听器会被移除，所有的子实例也会被销毁。
   */
  destroyed () {}
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<!--使用了scoped属性之后，父组件的style样式将不会渗透到子组件中，-->
<!--然而子组件的根节点元素会同时被设置了scoped的父css样式和设置了scoped的子css样式影响，-->
<!--这么设计的目的是父组件可以对子组件根元素进行布局。-->

<style scoped>
</style>
