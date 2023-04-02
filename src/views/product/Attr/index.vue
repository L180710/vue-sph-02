<template>
  <div>
    <el-card style="margin: 20px 0">
      <CategorySelect @getCategoryId="getCategoryId" :show="!isShowTable"></CategorySelect>
    </el-card>
    <el-card>
      <!-- 表格：展示平台属性 -->
      <div v-show="isShowTable">
        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!category3Id"
          @click="addAttr"
          >添加属性</el-button
        >
        <!-- 表格：展示派台属性 -->
        <el-table style="width: 100%" border :data="attrList">
          <el-table-column type="index" label="序号" width="80" align="center">
          </el-table-column>
          <el-table-column prop="attrName" label="属性名称" width="150">
          </el-table-column>
          <el-table-column prop="prop" label="属性值列表" width="width">
            <template slot-scope="{ row, index }">
              <el-tag
                type="success"
                v-for="(attrValue, index) in row.attrValueList"
                :key="attrValue.id"
                style="margin: 0 5px"
                >{{ attrValue.valueName }}</el-tag
              >
            </template>
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="150">
            <template slot-scope="{ row, $index }">
              <el-button
                type="warning"
                icon="el-icon-edit"
                size="mini"
                @click="updateAttr(row)"
              ></el-button>
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <!-- 添加属性 | 修改属性的结构 -->
      <div v-show="!isShowTable">
        <el-form :inline="true" ref="form" label-width="80px" :model="attrInfo">
          <el-form-item label="属性名">
            <el-input
              placeholder="请输入属性名"
              v-model="attrInfo.attrName"
            ></el-input>
          </el-form-item>
        </el-form>
        <el-button
          type="primary"
          icon="el-icon-plus"
          @click="addAttrValue"
          :disabled="!attrInfo.attrName"
          >添加属性值</el-button
        >
        <el-button @click="isShowTable = true">取消</el-button>
        <el-table
          style="width: 100%; margin: 20px 0"
          border
          :data="attrInfo.attrValueList"
        >
          <el-table-column
            align="center"
            type="index"
            label="序号"
            width="80"
          ></el-table-column>
          <el-table-column width="width" prop="prop" label="属性值名称">
            <template slot-scope="{ row, $index }">
              <!-- 这里结构需要用到 span 与 input 进行来回的切换 -->
              <el-input
                v-model="row.valueName"
                placeholder="请输入属性值名称"
                size="mini"
                v-if="row.flag"
                :ref="$index"
                @blur="toLook(row)"
                @keyup.native.enter="toLook(row)"
              ></el-input>
              <span
                v-else
                @click="toEdit(row, $index)"
                style="display: block"
                >{{ row.valueName }}</span
              >
            </template>
          </el-table-column>
          <el-table-column width="width" prop="prop" label="操作">
            <template slot-scope="{ row, $index }">
              <!-- 气泡确认框 -->
              <el-popconfirm
                :title="`确定删除${row.valueName}`"
                @onConfirm="deleteAttrValue($index)"
              >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  slot="reference"
                ></el-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <el-button type="primary" @click="addOrUpdateAttr" :disabled="attrInfo.attrValueList.length < 1">保存</el-button>
        <el-button @click="isShowTable = true">取消</el-button>
      </div>
    </el-card>
  </div>
</template>

<script>
// 安心引入 lodash 当中的深拷贝
import cloneDeep from "lodash/cloneDeep";
export default {
  name: "Attr",
  data() {
    return {
      flag: true,
      category1Id: "",
      category2Id: "",
      category3Id: "",
      // 接收平台属性的字段
      attrList: [],
      // 这个属性控制 table 表格显示与隐藏
      isShowTable: true,
      // 收集新增属性 | 修改属性使用的
      attrInfo: {
        attrName: "", // 属性名
        attrValueList: [
          // 属性值，因为属性值可以有多个，因此用数组，每一个属性值都是一个对象需要 attrId  valueName
        ],
        categoryId: 0, // 三级分类 id
        categoryLevel: 3, // 因为服务器也需要区分几级 id
      },
    };
  },
  methods: {
    // 自定义事件的回调
    getCategoryId({ categoryId, level }) {
      // 区分三级分类相应的 id，以及父组件进行存储
      if (level == 1) {
        this.category1Id = categoryId;
        this.category2Id = "";
        this.category3Id = "";
      } else if (level == 2) {
        this.category2Id = categoryId;
        this.category3Id = "";
      } else {
        // 代表三级分类 id 有了
        this.category3Id = categoryId;
        // 发送请求获取平台属性数据
        this.getAttrList();
      }
    },
    // 获取平台属性的数据
    // 当用户确定三级分类的数据时，可以向服务器发送请求获取平台属性进行展示
    async getAttrList() {
      // 获取分类的 ID
      const { category1Id, category2Id, category3Id } = this;
      let result = await this.$API.attr.reqAttrList(
        category1Id,
        category2Id,
        category3Id
      );
      if (result.code == 200) {
        this.attrList = result.data;
      }
    },
    // 添加属性值回调
    addAttrValue() {
      // 向属性值的数组里面添加元素
      this.attrInfo.attrValueList.push({
        attrId: this.attrInfo.id,
        valueName: "",
        /*
         flag 属性给每一个属性值添加一个标志 flag，用户切换查看模式与编辑模式，
         好处是每一个属性值可以控制自己的模式切换.
         当前 flag 属性，响应式数据（数据变化视图跟着变化）
        */
        flag: true,
      });
      this.$nextTick(() => {
        this.$refs[this.attrInfo.attrValueList.length - 1].focus();
      });
    },
    // 添加属性按钮的回调
    addAttr() {
      // 切换 table 显示与隐藏
      this.isShowTable = false;
      // 清除数据
      this.attrInfo = {
        attrName: "", // 属性名
        attrValueList: [
          // 属性值，因为属性值可以有多个，因此用数组，每一个属性值都是一个对象需要 attrId  valueName
        ],
        categoryId: this.category3Id, // 三级分类 id
        categoryLevel: 3, // 因为服务器也需要区分几级 id
      };
    },
    // 修改某一个属性
    updateAttr(row) {
      this.isShowTable = false;
      // 将选中属性赋值给 attrInfo
      // 由于数据结构当中存在对象里面套数据，数据里面有套对象，因此需要使用深拷贝解决这类问题
      this.attrInfo = cloneDeep(row);
      // 修改某一个属性的时候，将相应属性值元素添加上 flag 这个标记
      this.attrInfo.attrValueList.forEach((item) => {
        // 这样书写也可以给属性值添加 flag，但是会发现视图不会跟着变化（因为 flag 不是响应式数据）
        // 因为 Vue 无法探测普通新增 property，这样书写的属性并非响应式（数据变化视图跟着变——
        this.$set(item, "flag", false);
      });
    },
    // 失去焦点的事件 -- 切换为查看模式，显示 span
    toLook(row) {
      // 如果属性值为空不能作为新的属性值，需要给用户提示，让他输入一个其他的属性值
      if (row.valueName.trim() == "") {
        this.$message("请你输入一个正常的属性值");
        return;
      }

      // 新增的属性值不能与已有属性值重复
      let isRepeat = this.attrInfo.attrValueList.some((item) => {
        /*
          新增的属性值不能与已有的属性值重复
          需要将 row 从数组里面判断的时候去除
          row 最新新增的属性值【数组的最后一项元素】
          判断的时候，需要把已有的数组当中新增的这个属性值去除
        */
        if (row !== item) {
          return row.valueName == item.valueName;
        }
      });
      if (isRepeat) return;
      // row 形参是当前用户添加的最新属性值
      // 当前编辑模式变为查看模式【让 input 显示，显示 span】
      row.flag = false;
    },
    // 点击 span 回调，变为编辑模式
    toEdit(row, index) {
      row.flag = true;
      /*
        获取 input 节点，实现自动聚焦
        需要注意：点击 span 的时候，切换 input 变为编辑模式，但是需要注意，对于浏览器而言。页面重绘与重排好时间的
        点击 span 的时候，重绘重排一个 input 它是需要耗费时间的，因此我们不可能一点击 span 立马获取到 input
        $nextTick()，当节点渲染完毕了，会执行一次
      */
      this.$nextTick(() => {
        // 获取相应的 input 表单元素实现聚焦
        this.$ref[index].focus();
      });
    },
    // 确认气泡框按钮确认的回调
    deleteAttrValue(index) {
      // 当前删除属性值的操作是不需要发请求的
      this.attrInfo.attrValueList.splice(index, 1);
    },
    // 保存按钮，进行添加属性或者修改属性操作
    async addOrUpdateAttr() {
      // 整理参数1：如果用户添加很多属性值，且属性值为空的不应该提交给服务器
      // 提交给服务器数据当中不应该出现 flag 字段
      this.attrInfo.attrValueList.filter(item => {
        // 过滤掉属性值不是空的
        if(item.valueName != '') {
          // 删除 flag 属性
          delete item.flag;
          return true;
        }
      });

      try {
        // 发送请求
        await this.$API.attr.reqAddOrUpdateAttr(this.attrInfo);
        // 展示平台属性的信号量进行切换
        this.isShowTable = true;
        // 提示消息
        this.$message({type: 'success', message: '保存成功'});
      } catch(error) {
        this.$message('保存失败');
      }
    }
  },
};
</script>

<style>
</style>
