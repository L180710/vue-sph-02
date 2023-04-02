<template>
  <div>
    <el-form ref="form" label-width="80px" :model="spu">
      <el-form-item label="SPU名称">
        <el-input placeholder="SPU名称" v-model="spu.spuName"></el-input>
      </el-form-item>
      <el-form-item label="品牌">
        <el-select placeholder="请选择品牌" v-model="spu.tmId">
          <el-option
            :label="tm.tmName"
            :value="tm.id"
            v-for="(tm, index) in tradeMarkList"
            :key="tm.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="SPU描述">
        <el-input
          type="textarea"
          rows="4"
          placeholder="描述"
          v-model="spu.description"
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-upload
          action="/dev-api/admin/product/fileUpload"
          list-type="picture-card"
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
          :on-success="handleSuccess"
          :file-list="spuImageList"
        >
          <i class="el-icon-plus"></i>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
          <img width="100%" :src="dialogImageUrl" alt="" />
        </el-dialog>
      </el-form-item>
      <el-form-item label="销售属性">
        <el-select
          :placeholder="`还有${unSelectSaleAttr.length}未选择`"
          v-model="attrIdAndAttrName"
        >
          <el-option
            :label="unselect.name"
            :value="`${unselect.id}:${unselect.name}`"
            v-for="(unselect, index) in unSelectSaleAttr"
            :key="unselect.id"
          ></el-option>
        </el-select>
        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!attrIdAndAttrName"
          @click="addSaleAttr"
          >添加销售属性</el-button
        >
        <el-table style="width: 100%" border :data="spu.spuSaleAttrList">
          <el-table-column
            type="index"
            label="序号"
            width="80px"
            align="center"
          >
          </el-table-column>
          <el-table-column prop="saleAttrName" label="属性名" width="width">
          </el-table-column>
          <el-table-column prop="prop" label="属性值名称列表" width="width">
            <template slot-scope="{ row, $index }">
              <el-tag
                :key="tag.id"
                v-for="(tag, index) in row.spuSaleAttrValueList"
                closable
                :disable-transitions="false"
                @close="row.spuSaleAttrValueList.splice(index, 1)"
              >
                {{ tag.saleAttrValueName }}
              </el-tag>
              <!--  @keyup.enter.native="handleInputConfirm" @blur="handleInputConfirm" -->
              <el-input
                class="input-new-tag"
                v-if="row.inputVisible"
                v-model="row.inputValue"
                ref="saveTagInput"
                size="small"
                @blur="handleInputConfirm(row)"
              >
              </el-input>
              <el-button
                v-else
                class="button-new-tag"
                size="small"
                @click="addSaleAttrValue(row)"
                >添加</el-button
              >
            </template>
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="width">
            <template slot-scope="{ row, $index }">
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
                @click="spu.spuSaleAttrList.splice($index, 1)"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="addOrUpdateSpu">保存</el-button>
        <el-button @click="cancel">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: "",
  data() {
    return {
      dialogImageUrl: "",
      dialogVisible: false,
      // spu 属性初始化的时候是一个空的对象
      spu: {
        // 存储 SPU 信息属性
        category3Id: 0,
        description: "",
        spuName: "",
        // id: 0,
        tmId: "",
        spuImageList: [
          // {
          //   id: 0,
          //   imgName: "string",
          //   imgUrl: "string",
          //   spuId: 0,
          // },
        ],
        spuSaleAttrList: [
          // {
          //   baseSaleAttrId: 0,
          //   id: 0,
          //   saleAttrName: "string",
          //   spuId: 0,
          //   spuSaleAttrValueList: [
          //     {
          //       baseSaleAttrId: 0,
          //       id: 0,
          //       isChecked: "string",
          //       saleAttrName: "string",
          //       saleAttrValueName: "string",
          //       spuId: 0,
          //     },
          //   ],
          // },
        ],
      },
      tradeMarkList: [], // 存储品牌信息
      spuImageList: [], // 存储 SPU 图片数据
      saleAttrList: [], // 销售属性的数据
      attrIdAndAttrName: "", // 收集未选择的销售属性 id
    };
  },
  computed: {
    // 计算出还未选择的销售属性
    unSelectSaleAttr() {
      // 整个平台的销售属性一共三个：颜色、尺寸、版本
      // 当前 spu 拥有属于自己销售属性 SPU.spuSaleAttrList -- 颜色
      // 数组过滤的方法，可以从已有的数组当中过滤出用户需要的元素，并返回一个新的数组

      let result = this.saleAttrList.filter((item) => {
        // every 数组的方法，会返回一个布尔值【真、假】
        return this.spu.spuSaleAttrList.every((item1) => {
          return item.name != item1.saleAttrName;
        });
      });
      return result;
    },
  },
  methods: {
    handleRemove(file, fileList) {
      /*
      file 参数：代表是删除的那个图片
      fileList 参数：照片删除某一张图片以后，剩余其他图片
      收集图片墙的数据
      */
      this.spuImageList = fileList;
    },
    handlePictureCardPreview(file) {
      // 将图片地址赋值给这个属性
      this.dialogImageUrl = file.url;
      // 对话框显示
      this.dialogVisible = true;
    },
    // 初始化 SpuForm 数据
    async initSpuData(spu) {
      // 获取 SPU 信息的数据
      let spuResult = await this.$API.spu.reqSpu(spu.id);
      if (spuResult.code == 200) {
        this.spu = spuResult.data;
      }

      // 获取品牌信息
      let tradeMarkResult = await this.$API.spu.reqTradeMarkList();
      if (tradeMarkResult.code == 200) {
        this.tradeMarkList = tradeMarkResult.data;
      }

      // 获取 spu 图片数据
      let spuImageResult = await this.$API.spu.reqSpuImageList(spu.id);
      if (spuImageResult.code == 200) {
        let listArr = spuImageResult.data;
        /* 由于图片墙显示图片数据需要数组，数组里面的元素要有 name 与 url 字段，需要把服务器返回的数据进行修改 */
        listArr.forEach((item) => {
          item.name = item.imgName;
          item.url = item.imgUrl;
        });
        // 把整理好的数据赋值
        this.spuImageList = listArr;
      }

      // 获取平台全部的销售属性
      let saleResult = await this.$API.spu.reqBaseSaleAttrList();
      if (saleResult.code == 200) {
        this.saleAttrList = saleResult.data;
      }
    },
    // 照片墙图片上传成功的回调
    handleSuccess(response, file, fileList) {
      // 收集图片的信息
      this.spuImageList = fileList;
    },
    // 添加新的销售属性
    addSaleAttr() {
      /*
      已经收集需要添加的销售属性的信息
      把收集到的销售属性进行分割
      */
      const [baseSaleAttrId, saleAttrName] = this.attrIdAndAttrName.split(":");
      // 向 SPU 对象的 spuSaleAttrList 属性添加新的销售属性
      let newSaleAttr = {
        baseSaleAttrId,
        saleAttrName,
        spuSaleAttrValueList: [],
      };
      // 添加新的销售属性
      this.spu.spuSaleAttrList.push(newSaleAttr);
      // 清空数据
      this.attrIdAndAttrName = "";
    },
    // 添加按钮的回调
    addSaleAttrValue(row) {
      // 点击销售属性值当中添加按钮的时候，需要 button 变为 input，通过当前销售属性 inputVisible 控制
      // 挂载在销售属性身上的响应式数据 inputVisible，控制 button 与 input 切换
      this.$set(row, "inputVisible", true);
      // 通过响应式数据 inputValue 字段收集新增的销售属性值
      this.$set(row, "inputValue", "");
    },
    // el-input 失去焦点的事件
    handleInputConfirm(row) {
      // 解构出销售属性当中三级数据
      const { baseSaleAttrId, inputValue } = row;
      // 新增的销售属性值名称不能为空
      if (inputValue.trim() == "") {
        this.$message("属性值不能为空");
        return;
      }
      // 属性值不能重复，这里也可以用 some
      let result = row.spuSaleAttrValueList.every((item) => {
        item.saleAttrValueName != inputValue;
      });
      if (!result) {
        return;
      }

      // 新增的销售属性
      let newSaleAttrValue = { baseSaleAttrId, saleAttrValueName: inputValue };
      // 新增
      row.spuSaleAttrValueList.push(newSaleAttrValue);
      // 修改 inputVisible 为 false，显示 button
      row.inputVisible = false;
    },
    // 保存按钮事件
    async addOrUpdateSpu() {
      /*
      整理参数：需要整理图片墙的数据
      携带参数：对于图片，需要携带 imageName 与 imageUrl 字段
      */
      this.spu.spuImageList = this.spuImageList.map((item) => {
        return {
          imageName: item.name,
          imageUrl: (item.response && item.response.data) || item.url,
        };
      });
      // 发请求
      let result = await this.$API.spu.reqAddOrUpdateSpu(this.spu);
      if (result.code == 200) {
        // 提示
        this.$message({ type: "success", message: "保存成功" });
        // 通知父组件会到场景 0
        this.$emit("changeScene", {scene: 0, flag: this.spu.id ? '修改' : '添加'});
      }
      // 清除数据
      Object.assign(this._data, this.$options.data());
    },
    // 点击添加 SPU 按钮的时候，发请求函数
    async addSpuData(category3Id) {
      // 添加 SPU 的时候收集三级分类的 id
      this.spu.category3Id = category3Id;

      // 获取品牌信息
      let tradeMarkResult = await this.$API.spu.reqTradeMarkList();
      if (tradeMarkResult.code == 200) {
        this.tradeMarkList = tradeMarkResult.data;
      }

      // 获取平台全部的销售属性
      let saleResult = await this.$API.spu.reqBaseSaleAttrList();
      if (saleResult.code == 200) {
        this.saleAttrList = saleResult.data;
      }
    },
    // 取消按钮
    cancel() {
      // 取消按钮的回调，通知父亲切换场景为 0
      this.$emit('changeScene', {scene: 0, flag: ''});
      /*
      清除数据
      组件实例 this._data 可以操作 data 当中响应式数据
      this.$options 可以获取配置对象，配置对象的 data 函数执行，返回响应式数据为空的
      */
      Object.assign(this._data, this.$options.data());
    }
  },
};
</script>


<style>
.el-tag + .el-tag {
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>
