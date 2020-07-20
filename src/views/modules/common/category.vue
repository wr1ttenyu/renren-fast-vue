<template>
  <div>
    <el-tree
      :data="categories"
      :props="defaultProps"
      node-key="catId"   
      ref="menuTree"
      @node-click="nodeclick"
    ></el-tree>
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
    nodeclick(data, node, component) {
      console.log("子组件category的节点被点击", data, node, component);
      //向父组件发送事件；
      this.$emit("tree-node-click", data, node, component);
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
