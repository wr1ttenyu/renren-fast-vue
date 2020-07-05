<template>
  <div>
    <el-switch
      v-model="draggable"
      active-color="#13ce66"
      inactive-color="#ff4949"
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
    ></el-switch>

    <el-button v-if="draggable" @click="batchSave">批量保存</el-button>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree
      :data="categories"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandKey"
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      ref="menuTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >Append</el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">Edit</el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >Delete</el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      :close-on-click-modal="false"
      width="30%"
    >
      <el-form ref="form" :model="category" label-width="80px">
        <el-form-item label="分类名称">
          <el-input v-model="category.name"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit"></el-input>
        </el-form-item>
        <el-form-item label="是否展示">
          <el-input v-model="category.showStatus"></el-input>
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
import Vue from "vue";
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';
export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  data() {
    return {
      maxLevel: 0,
      draggable: false,
      updateNodes: [],
      pCid: [],
      title: '',
      dialogType: '', //edit add
      category: {
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: '',
        productUnit: ''
      },
      dialogVisible: false,
      categories: [],
      expandKey: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  methods: {
    // 获取数据列表
    getCategoryData() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get"
      }).then(({ data }) => {
        this.categories = data.data;
      });
    },

    // 打开新增对话框
    append(data) {
      this.dialogType = "add";
      this.title = "添加分类";
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.showStatus = 1;
      this.category.sort = 0;
      this.category.catId = null;
      this.category.productUnit = '';
      this.category.icon = '';
      this.category.name = '';
    },

    allowDrop(draggingNode, dropNode, type) {
      this.maxLevel = 0;
      // 被拖动的节点层数 + 父节点 不大于3
      this.countNodeDeep(draggingNode);
      let nodeDeep = Math.abs(this.maxLevel - draggingNode.level) + 1;
      if (type == "inner") {
        return nodeDeep + dropNode.level <= 3;
      } else {
        return nodeDeep + dropNode.parent.level <= 3;
      }
    },

    countNodeDeep(draggingNode) {
      // 当前节点加子节点深度
      if (
        draggingNode.childNodes != null &&
        draggingNode.childNodes.length > 0
      ) {
        for (let index = 0; index < draggingNode.childNodes.length; index++) {
          if (draggingNode.childNodes[index].level > this.maxLevel) {
            this.maxLevel = draggingNode.childNodes[index].level;
          }
          this.countNodeDeep(draggingNode.childNodes[index]);
        }
      } else {
        this.maxLevel = draggingNode.level;
      }
    },

    edit(data) {
      this.dialogType = "edit";
      this.title = "修改分类";
      this.dialogVisible = true;

      // 发送请求获取最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
        params: this.$http.adornParams({})
      }).then(({ data }) => {
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.catLevel = data.data.catLevel;
        this.category.showStatus = data.data.showStatus;
        this.category.sort = data.data.sort;
        this.category.productUnit = data.data.productUnit;
        this.category.icon = data.data.icon;
        this.category.parentCid = data.data.parentCid;
      });
    },

    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`是否删除[${data.name}]分类?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            this.$message({
              message: "菜单删除成功",
              type: "success"
            });
            this.getCategoryData();
            // 设置默认需要展开的菜单
            this.expandKey = [node.parent.data.catId];
          });
        })
        .catch(() => {});
    },

    addCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        this.$message({
          message: "菜单保存成功",
          type: "success"
        });
        this.dialogVisible = false;
        this.getCategoryData();
        // 设置默认需要展开的菜单
        this.expandKey = [this.category.parentCid];
      });
    },

    editCategory() {
      var { catId, name, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ catId, name, icon, productUnit }, false)
      }).then(({ data }) => {
        this.$message({
          message: "菜单保存成功",
          type: "success"
        });
        this.dialogVisible = false;
        this.getCategoryData();
        // 设置默认需要展开的菜单
        this.expandKey = [this.category.parentCid];
      });
    },

    batchSave() {
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({ data }) => {
        this.$message({
          message: "菜单顺序等修改成功",
          type: "success"
        });
        //刷新出新的菜单
        this.getCategoryData();
        //设置需要默认展开的菜单
        this.expandKey = this.pCid;
        this.updateNodes = [];
        this.maxLevel = 0;
      });
    },

    batchDelete() {
      let catIds = [];
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      console.log("被选中的元素", checkedNodes);
      for (let i = 0; i < checkedNodes.length; i++) {
        catIds.push(checkedNodes[i].catId);
      }
      this.$confirm(`是否批量删除【${catIds}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(catIds, false)
          }).then(({ data }) => {
            this.$message({
              message: "菜单批量删除成功",
              type: "success"
            });
            this.getCategoryData();
          });
        })
        .catch(() => {});
    },

    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      } else if (this.dialogType == "edit") {
        this.editCategory();
      }
    },

    handleDrop(draggingNode, dropNode, dropType, ev) {
      let pCid = 0;
      let siblings = null;
      // 获取父Id
      if (dropType == "before" || dropType == "after") {
        pCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.data.catId;
        siblings = dropNode.parent.childNodes;
      } else {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      }
      // 当前拖拽节点的最新顺序
      this.pCid.push(pCid);
      for (let i = 0; i < siblings.length; i++) {
        let si = siblings[i];
        // 节点是当前拖拽节点
        if (si.data.catId == draggingNode.data.catId) {
          let catLevel = draggingNode.data.catLevel;
          // 节点层级发生变化
          if (si.level != draggingNode.level) {
            catLevel = si.level;
            //修改他子节点的层级
            this.updateChildNodeLevel(si);
          }
          this.updateNodes.push({
            catId: si.data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel
          });
        } else {
          this.updateNodes.push({ catId: si.data.catId, sort: i });
        }
      }
    },

    updateChildNodeLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level
          });
          this.updateChildNodeLevel(node.childNodes[i]);
        }
      }
    }
  },
  watch: {},
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getCategoryData();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {} //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>
