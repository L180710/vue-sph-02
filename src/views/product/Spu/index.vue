<template>
  <div>
    <el-card style="margin: 20px 0">
      <!-- 三级联动已经是全局组件了 -->
      <CategorySelect
        @getCategoryId="getCategoryId"
        :show="scene != 0"
      ></CategorySelect>
    </el-card>
    <el-card>
      <!-- 底部这里将来是有三部分进行切换 -->
      <div v-show="scene == 0">
        <!-- 展示 SPU 列表的结构 -->
        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!category3Id"
          @click="addSpu"
          >添加 SPU</el-button
        >
        <el-table style="width: 100%" border :data="records">
          <el-table-column type="index" label="序号" width="80" align="center">
          </el-table-column>
          <el-table-column prop="spuName" label="SPU名称" width="width">
          </el-table-column>
          <el-table-column prop="description" label="SPU描述" width="width">
          </el-table-column>
          <el-table-column label="操作" width="width">
            <template slot-scope="{ row, $index }">
              <!-- 这里按钮将来用 hintButton 替换 -->
              <hint-button
                type="success"
                icon="el-icon-plus"
                size="mini"
                title="添加Spu"
                @click="addSku(row)"
              ></hint-button>
              <hint-button
                type="warning"
                icon="el-icon-edit"
                size="mini"
                title="修改Spu"
                @click="updateSpu(row)"
              ></hint-button>
              <hint-button
                type="info"
                icon="el-icon-info"
                size="mini"
                title="查看当前Spu全部列表"
                @click="handler(row)"
              ></hint-button>
              <el-popconfirm
                title="这是一段内容确定删除吗？"
                @onConfirm="deleteSpu(row)"
              >
                <hint-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  title="删除Spu"
                  slot="reference"
                ></hint-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          style="text-align: center"
          :current-page="page"
          :page-sizes="[3, 5, 10]"
          :page-size="limit"
          :total="total"
          layout="prev, pager, next, jumper, ->, sizes, total"
          @current-change="getSpuList"
          @size-change="handleSizeChange"
        ></el-pagination>
      </div>
      <SpuForm
        v-show="scene == 1"
        @changeScene="changeScene"
        ref="spu"
      ></SpuForm>
      <SkuForm
        v-show="scene == 2"
        ref="sku"
        @changeScene="changeScene"
      ></SkuForm>
    </el-card>
    <!-- table 展示 sku 列表 -->
    <el-dialog :title="`${spu.spuName}的sku列表`" :visible.sync="dialogTableVisible" :before-close="close">
      <el-table :data="skuList" style="width: 100%" border v-loading="loading">
        <el-table-column prop="skuName" label="名称" width="width"></el-table-column>
        <el-table-column prop="price" label="价格" width="width"></el-table-column>
        <el-table-column prop="weight" label="重量" width="width"></el-table-column>
        <el-table-column label="默认图片" width="width">
          <template slot-scope="{row, $index}">
            <img :src="row.skuDefaultImg" alt="" style="width:100px;height:100px">
          </template>
        </el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>

<script>
// 引入子组件
import SpuForm from "./SpuForm";
import SkuForm from "./SkuForm";
export default {
  name: "Spu",
  data() {
    return {
      // 分别是分类的 id
      category1Id: "",
      category2Id: "",
      category3Id: "",
      // 检测三级联动的可操作性
      show: true,
      page: 1, // 分页器当前第几页
      limit: 3, // 每一页需要显示多少条数据
      records: [], // spu 列表的数据
      total: 0, // 分页器移动需要展示数据的条数
      scene: 0, // 0 代表 SPU 列表数据； 1 添加 SPu | 修改 Spu， 2 添加 Sku
      dialogTableVisible: false,  // 控制对话框的显示与隐藏
      spu: {},
      skuList: [], // 存储的是 sku 列表的数据
      loading: true,

    };
  },
  components: { SpuForm, SkuForm },
  methods: {
    // 三级联动的自定义事件，可以把子组件相应的 id 传递给父组件
    getCategoryId({ categoryId, level }) {
      // categoryId：获取一、二、三级分类的 id level：为了区分是几级 id
      if (level == 1) {
        this.category1Id = categoryId;
        // 清除二、三级分类 id
        this.category2Id = "";
        this.category3Id = "";
      } else if (level == 2) {
        this.category2Id = categoryId;
        // 清除三级 id
        this.category3Id = "";
      } else {
        this.category3Id = categoryId;
        // 获取 SPU 列表数据进行展示
        this.getSpuList();
      }
    },
    // 获取 SPU 列表数据的方法
    async getSpuList(pages = 1) {
      this.page = pages;
      const { page, limit, category3Id } = this;
      // 携带三个参数：page 第几页， limit 每一页需要显示多少条数据，三级分类 id
      let result = await this.$API.spu.reqSpuList(page, limit, category3Id);
      if (result.code == 200) {
        this.records = result.data.records;
        this.total = result.data.total;
      }
    },
    // 当分页器的某一个展示数据条数发生变化的回调
    handleSizeChange(limit) {
      // 修改参数
      this.limit = limit;
      // 再发请求
      this.getSpuList();
    },
    // 添加 SPU 按钮回调
    addSpu() {
      // 切换为场景 1
      this.scene = 1;
      // 通知子组件 SpuForm 发请求 -- 两个
      this.$refs.spu.addSpuData(this.category3Id);
    },
    // 修改某一个 SPU
    updateSpu(row) {
      this.scene = 1;
      // 通过 ref 属性 获取子组件 SpuForm 子组件
      this.$refs.spu.initSpuData(row);
    },
    // 自定义事件回调（SpuForm）
    changeScene(scene, flag) {
      // flag 这形参为了区分保存按钮是添加还是修改
      // 切换结构
      this.scene = scene;
      // 子组件通知父组件切换场景，需要再次获取 spu 列表数据进行展示
      if (flag == "修改") {
        this.getSpuList(this.page);
      } else {
        this.getSpuList();
      }
    },
    // 删除 spu 的回调
    async deleteSpu(row) {
      let result = await this.$API.spu.reqDeleteSpu(row.id);
      if (result.code == 200) {
        this.$message({ type: "success", message: "删除成功" });
        // 代表 spu 个数大于 1 删除的时候停留在当前页，如果 spu 个数小于 1，回到上一页
        this.getSpuList(this.records.length > 1 ? this.page : this.page - 1);
      }
    },
    // 添加 sku 按钮的回调
    addSku(row) {
      // 切换场景 2
      this.scene = 2;
      // 父组件调用子组件的方法，让子组件发请求 --- 三个请求
      this.$refs.sku.getData(this.category1Id, this.category2Id, row);
    },
    // skuForm 通知父组件修改场景
    changeScene(scene) {
      this.scene = scene;
    },
    // 查看 SKU 按钮的回调
    async handler(spu) {
      // 点击这个按钮的时候，对话框是可见的
      this.dialogTableVisible = true;
      // 保存 spu 信息
      this.spu = spu;
      // 获取 sku 列表的数据进行展示
      let result = await this.$API.spu.reqSkuList(spu.id);
      if(result.code == 200) {
        this.skuList = result.data;
        // loading 隐藏
        this.loading = false;
      }
    },
    // 关闭对话框的回调
    close(done) {
      // loading 属性再次变为真
      this.loading = true;
      // 清除 sku 列表的数据
      this.skuList = [];
      // 管理对话框
      done();
    }
  },
};
</script>

<style>
</style>
