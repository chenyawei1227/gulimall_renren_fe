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
      <el-row :gutter="20">
        <el-col :span="6"><div class="grid-content bg-purple"></div>
          <category @tree-node-click="treenodeclick"></category>
        </el-col>
        <el-col :span="18"><div class="grid-content bg-purple">
          <div class="mod-config">
            <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
              <el-form-item>
                <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
              </el-form-item>
              <el-form-item>
                <el-button @click="getDataList()">查询</el-button>
                <el-button type="success" @click="getAllDataList()">查询全部</el-button>
                <el-button
                  v-if="isAuth('product:attrgroup:save')"
                  type="primary"
                  @click="addOrUpdateHandle()"
                >新增</el-button>
                <el-button
                  v-if="isAuth('product:attrgroup:delete')"
                  type="danger"
                  @click="deleteHandle()"
                  :disabled="dataListSelections.length <= 0"
                >批量删除</el-button>
              </el-form-item>
            </el-form>
            <el-table
              :data="dataList"
              border
              v-loading="dataListLoading"
              @selection-change="selectionChangeHandle"
              style="width: 100%;"
            >
              <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
              <el-table-column prop="attrGroupId" header-align="center" align="center" label="分组id"></el-table-column>
              <el-table-column prop="attrGroupName" header-align="center" align="center" label="组名"></el-table-column>
              <el-table-column prop="sort" header-align="center" align="center" label="排序"></el-table-column>
              <el-table-column prop="descript" header-align="center" align="center" label="描述"></el-table-column>
              <el-table-column prop="icon" header-align="center" align="center" label="组图标"></el-table-column>
              <el-table-column prop="catelogId" header-align="center" align="center" label="所属分类id"></el-table-column>
              <el-table-column
                fixed="right"
                header-align="center"
                align="center"
                width="150"
                label="操作"
              >
                <template slot-scope="scope">
                  <el-button type="text" size="small" @click="relationHandle(scope.row.attrGroupId)">关联</el-button>
                  <el-button
                    type="text"
                    size="small"
                    @click="addOrUpdateHandle(scope.row.attrGroupId)"
                  >修改</el-button>
                  <el-button type="text" size="small" @click="deleteHandle(scope.row.attrGroupId)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
            <el-pagination
              @size-change="sizeChangeHandle"
              @current-change="currentChangeHandle"
              :current-page="pageIndex"
              :page-sizes="[10, 20, 50, 100]"
              :page-size="pageSize"
              :total="totalPage"
              layout="total, sizes, prev, pager, next, jumper"
            ></el-pagination>
            <!-- 弹窗, 新增 / 修改 -->
            <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>

            <!-- 修改关联关系 -->
            <relation-update v-if="relationVisible" ref="relationUpdate" @refreshData="getDataList"></relation-update>
          </div>
        </div></el-col>
      </el-row>
    </div>
</template>

<script>
  /**
   * 父子组件传递
   * 1、自组建给父组件传递数据，事件机制；自组件给父组件发送一个事件，携带上数据
   * 这里可以导入其他文件（比如：组件，工具js, 第三方插件js, json文件, 图片文件等）
   * 例如： import 《组件名称》 from 《组件路径》
   */


  import Category from "../common/category";
  import AddOrUpdate from "./attrgroup-add-or-update";
  import RelationUpdate from "./attr-group-relation";

  export default {
    // 组件名称
    name: 'demo',
    // 组件参数 接收来自父组件的数据
    props: {},
    // 局部注册的组件
    components: {
      Category, AddOrUpdate, RelationUpdate
    },
    // 组件状态值
    data () {
      return {
        catId: 0,
        dataForm: {
          key: ""
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        relationVisible: false
      };
    },
    // 计算属性
    computed: {},
    // 侦听器
    watch: {},
    // 组件方法
    methods: {

      // 处理分组与属性的关联
      relationHandle(groupId) {
        this.relationVisible = true;
        this.$nextTick(() => {
          this.$refs.relationUpdate.init(groupId);
        });
      },
      // 感知树节点被点击
      treenodeclick(data, node, component) {
        if (node.level === 3) {
          this.catId = data.catId;
          this.getDataList(); // 重新查询
        }
      },
      getAllDataList () {
        this.catId = 0;
        this.getDataList();
      },
      // 获取数据列表
      getDataList() {
        this.dataListLoading = true;
        this.$http({
          url: this.$http.adornUrl(`/product/attrgroup/list/${this.catId}`),
          method: "get",
          params: this.$http.adornParams({
            page: this.pageIndex,
            limit: this.pageSize,
            key: this.dataForm.key
          })
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list;
            this.totalPage = data.page.totalCount;
          } else {
            this.dataList = [];
            this.totalPage = 0;
          }
          this.dataListLoading = false;
        });
      },
      // 每页数
      sizeChangeHandle(val) {
        this.pageSize = val;
        this.pageIndex = 1;
        this.getDataList();
      },
      // 当前页
      currentChangeHandle(val) {
        this.pageIndex = val;
        this.getDataList();
      },
      // 多选
      selectionChangeHandle(val) {
        this.dataListSelections = val;
      },
      // 新增 / 修改
      addOrUpdateHandle(id) {
        this.addOrUpdateVisible = true;
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id);
        });
      },
      // 删除
      deleteHandle(id) {
        var ids = id
          ? [id]
          : this.dataListSelections.map(item => {
            return item.attrGroupId;
          });
        this.$confirm(
          `确定对[id=${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作?`,
          "提示",
          {
            confirmButtonText: "确定",
            cancelButtonText: "取消",
            type: "warning"
          }
        ).then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/attrgroup/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.getDataList();
                }
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        });
      }
    },
    // 以下是生命周期钩子   注：没用到的钩子请自行删除
    /**
     * 在实例初始化之后，组件属性计算之前，如data属性等
     */
    beforeCreate () {
    },
    /**
     * 组件实例创建完成，属性已绑定，但DOM还未生成，$ el属性还不存在
     */
    created () {
    },
    /**
     * 在挂载开始之前被调用：相关的 render 函数首次被调用。
     */
    beforeMount () {
    },
    /**
     * el 被新创建的 vm.$ el 替换，并挂载到实例上去之后调用该钩子。
     * 如果 root 实例挂载了一个文档内元素，当 mounted 被调用时 vm.$ el 也在文档内。
     */
    mounted () {
    },
    /**
     * 数据更新时调用，发生在虚拟 DOM 重新渲染和打补丁之前。
     * 你可以在这个钩子中进一步地更改状态，这不会触发附加的重渲染过程。
     */
    beforeUpdate () {
    },
    /**
     * 由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子。
     * 当这个钩子被调用时，组件 DOM 已经更新，所以你现在可以执行依赖于 DOM 的操作。
     */
    updated () {
    },
    /**
     * keep-alive 组件激活时调用。 仅针对keep-alive 组件有效
     */
    activated () {
      this.getDataList();
    },
    /**
     * keep-alive 组件停用时调用。 仅针对keep-alive 组件有效
     */
    deactivated () {
    },
    /**
     * 实例销毁之前调用。在这一步，实例仍然完全可用。
     */
    beforeDestroy () {
    },
    /**
     * Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，
     * 所有的事件监听器会被移除，所有的子实例也会被销毁。
     */
    destroyed () {
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<!--使用了scoped属性之后，父组件的style样式将不会渗透到子组件中，-->
<!--然而子组件的根节点元素会同时被设置了scoped的父css样式和设置了scoped的子css样式影响，-->
<!--这么设计的目的是父组件可以对子组件根元素进行布局。-->
<style scoped>

</style>
