<template>
  <div>
    <el-tree
      :data="categories"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandKey"
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

    <el-dialog :title="提示" :visible.sync="dialogVisible" :close-on-click-modal="false" width="30%">
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
      title: "",
      dialogType: "", //edit add
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: "",
        productUnit: ""
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
      this.category.productUnit = "";
      this.category.icon = "";
      this.category.name = "";
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

    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      } else if (this.dialogType == "edit") {
        this.editCategory();
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
