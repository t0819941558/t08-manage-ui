<template>
  <div class="app-container">
    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          icon="el-icon-plus"
          size="mini"
          @click="handleAdd"
          v-hasPermi="['amount:sum:add']"
          >新增</el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="cyan"
          icon="el-icon-search"
          size="mini"
          @click="handleQuery"
          >搜索</el-button
        >
      </el-col>

      <el-col :span="1.5">
        <el-button
          type="warning"
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
          v-hasPermi="['amount:transit:export']"
          >导出</el-button
        >
      </el-col>

      <el-col :span="1.5">
        <el-tag>页面显示，单位：元</el-tag>
      </el-col>

      <div class="top-right-btn">
        <el-tooltip class="item" effect="dark" content="刷新" placement="top">
          <el-button
            size="mini"
            circle
            icon="el-icon-refresh"
            @click="handleQuery"
          />
        </el-tooltip>
        <el-tooltip
          class="item"
          effect="dark"
          :content="showSearch ? '隐藏搜索' : '显示搜索'"
          placement="top"
        >
          <el-button
            size="mini"
            circle
            icon="el-icon-search"
            @click="showSearch = !showSearch"
          />
        </el-tooltip>
      </div>
    </el-row>

    <el-table
      :data="sumList"
      style="width: 100%;margin-bottom: 20px;"
      row-key="id"
      border
      default-expand-all
      :tree-props="{ children: 'children', hasChildren: 'hasChildren' }"
    >
      <el-table-column prop="product" label="产品名称" sortable width="180">
      </el-table-column>
      <el-table-column
        prop="inventoryAmount"
        label="合计金额"
        sortable
        width="180"
      >
      </el-table-column>

      <el-table-column label="可销售库存" align="center">
        <el-table-column
          prop="saleableInventoryNum"
          label="数量"
          align="center"
        >
        </el-table-column>
        <el-table-column
          prop="saleableInventoryAmount"
          label="金额"
          align="center"
        >
        </el-table-column>
      </el-table-column>

      <el-table-column label="不可销售库存" align="center">
        <el-table-column
          prop="unsaleableInventoryNum"
          label="数量"
          align="center"
        >
        </el-table-column>
        <el-table-column
          prop="unsaleableInventoryAmount"
          label="金额"
          align="center"
        >
        </el-table-column>
      </el-table-column>

      <el-table-column prop="updateTime" label="更新时间"> </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-plus"
            @click="handleAdd(scope.row, 'group')"
            v-hasPermi="['amount:sum:add']"
            >新增分组</el-button
          >
          <el-button
            size="mini"
            type="text"
            icon="el-icon-plus"
            @click="handleAdd(scope.row, 'leaf')"
            v-hasPermi="['amount:sum:add']"
            >新增节点</el-button
          >
          <el-button
            v-show="scope.row.node_type != 'group'"
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['amount:sum:edit']"
            >修改</el-button
          >
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['amount:sum:remove']"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改库存汇总对话框 -->
    <el-dialog
      :title="title"
      :visible.sync="open"
      width="1000px"
      append-to-body
    >
      <el-table :data="newTable" border style="width: 100%">
        <el-table-column label="产品" width="180">
          <template slot-scope="scope">
            <el-input
              v-model="scope.row.product"
              placeholder="请输入产品名称"
            />
          </template>
        </el-table-column>
        <el-table-column label="可销售库存金额">
          <template slot-scope="scope">
            <el-input
              v-model="scope.row.saleableInventoryAmount"
              placeholder="请输入可销售库存金额,单位：元"
              :disabled="!scope.row.addType"
            />
          </template>
        </el-table-column>
        <el-table-column label="不可销售库存金额">
          <template slot-scope="scope">
            <el-input
              v-model="scope.row.unsaleableInventoryAmount"
              placeholder="请输入不可销售库存金额,单位：元"
              :disabled="!scope.row.addType"
            />
          </template>
        </el-table-column>
        <el-table-column label="可销售库存数量">
          <template slot-scope="scope">
            <el-input
              v-model="scope.row.saleableInventoryNum"
              placeholder="请输入可销售库存数量"
              :disabled="!scope.row.addType"
            />
          </template>
        </el-table-column>
        <el-table-column label="不可销售库存数量">
          <template slot-scope="scope">
            <el-input
              v-model="scope.row.unsaleableInventoryNum"
              placeholder="请输入不可销售库存数量"
              :disabled="!scope.row.addType"
            />
          </template>
        </el-table-column>
        <el-table-column label="合计金额">
          <template slot-scope="scope">
            <div>
              {{
                parseFloat(scope.row.saleableInventoryAmount || 0) +
                  parseFloat(scope.row.unsaleableInventoryAmount || 0)
              }}
            </div>
          </template>
        </el-table-column>
        <el-table-column label="合计数量">
          <template slot-scope="scope">
            <div>
              {{
                parseFloat(scope.row.saleableInventoryNum || 0) +
                  parseFloat(scope.row.unsaleableInventoryNum || 0)
              }}
            </div>
          </template>
        </el-table-column>
      </el-table>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  listSum,
  getSum,
  delSum,
  addSum,
  updateSum,
  exportSum
} from "@/api/amount/sum";

export default {
  name: "Sum",
  data() {
    return {
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 库存汇总表格数据
      sumList: [],
      // 菜单树选项
      sumOptions: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 查询参数
      queryParams: {
        menuName: undefined,
        visible: undefined
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {}
    };
  },
  created() {
    this.getList();
  },
  methods: {
    /** 查询库存汇总列表 */
    getList() {
      this.loading = true;
      listSum().then(response => {
        this.sumList = [response];
        this.loading = false;
      });
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
    // 表单重置
    reset() {
      this.form = {
        id: null,
        sumSaleableInventoryAmount: null,
        sumUnsaleableInventoryAmount: null,
        sumInventoryAmount: null,
        createTime: null,
        updateTime: null
      };
      this.resetForm("form");
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.resetForm("queryForm");
      this.handleQuery();
    },
    // 多选框选中数据
    handleSelectionChange(selection) {
      this.ids = selection.map(item => item.id);
      this.single = selection.length !== 1;
      this.multiple = !selection.length;
    },
    /** 新增按钮操作 */
    handleAdd(row, isGroup = "group") {
      this.reset();
      let obj = {
        saleableInventoryAmount: 0,
        unsaleableInventoryAmount: 0,
        saleableInventoryNum: 0,
        unsaleableInventoryNum: 0,
        node_type: isGroup,
        product: ""
      };
      if (row != null && row.id) {
        this.form.parentId = row.id;
        obj.parentId = row.id;
      } else {
        this.form.parentId = "";
        obj.parentId = "";
      }
      this.form.addType = isGroup;
      this.newTable = [obj];

      this.open = true;
      this.title = "添加库存汇总";
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset();
      this.open = true;
      this.modify = true;
      this.newTable[0] = row;
    },

    /** 提交按钮 */
    submitForm(e) {
      console.log(e);
      let sumSaleableInventoryAmount = 0;
      let sumUnsaleableInventoryAmount = 0;
      let sumSaleableInventoryNum = 0;
      let sumUnsaleableInventoryNum = 0;
      let sumInventoryAmount = 0;

      // const errItem = this.newTable.filter(
      //   item =>
      //     item.saleableInventoryAmount === "" ||
      //     item.unsaleableInventoryAmount === ""
      // )[0];
      // if (errItem) {
      //   this.newTabelTip = `${errItem.product}的${
      //     errItem.saleableInventoryAmount
      //       ? "不可销售库存金额"
      //       : "可销售库存金额"
      //   }不能为空`;
      //   this.$message(this.newTabelTip);
      //   return;
      // }

      if (!this.modify) {
        addSum(this.newTable[0]).then(res => {
          if (res.code === 200) {
            this.msgSuccess("添加成功");
            this.open = false;
            this.getList();
            this.newTabelTip = "";
          }
        });
      } else {
        updateSum(this.newTable[0]).then(response => {
          if (response.code === 200) {
            this.msgSuccess("修改成功");
            this.open = false;
            this.getList();
            this.newTabelTip = "";
            this.newTable = [
              {
                product: "建材",
                saleableInventoryAmount: "",
                unsaleableInventoryAmount: ""
              },
              {
                product: "日用品",
                saleableInventoryAmount: "",
                unsaleableInventoryAmount: ""
              },
              {
                product: "电器",
                saleableInventoryAmount: "",
                unsaleableInventoryAmount: ""
              },
              {
                product: "家具",
                saleableInventoryAmount: "",
                unsaleableInventoryAmount: ""
              },
              {
                product: "厨房卫浴",
                saleableInventoryAmount: "",
                unsaleableInventoryAmount: ""
              },
              {
                product: "灯具",
                saleableInventoryAmount: "",
                unsaleableInventoryAmount: ""
              },
              {
                product: "软装",
                saleableInventoryAmount: "",
                unsaleableInventoryAmount: ""
              },
              {
                product: "纺织品",
                saleableInventoryAmount: "",
                unsaleableInventoryAmount: ""
              },
              {
                product: "其它",
                saleableInventoryAmount: "",
                unsaleableInventoryAmount: ""
              }
            ];
          }
        });
      }
    },
    /** 删除按钮操作 */
    handleDelete(row) {
      const ids = row.id || this.ids;
      this.$confirm(
        '是否确认删除库存汇总编号为"' + ids + '"的数据项?',
        "警告",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      )
        .then(function() {
          return delSum(ids);
        })
        .then(() => {
          this.getList();
          this.msgSuccess("删除成功");
        })
        .catch(function() {});
    },
    /** 导出按钮操作 */
    handleExport() {
      this.$confirm("是否确认导出所有库存汇总数据项?", "警告", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(function() {
          return exportSum();
        })
        .then(response => {
          this.download(response.msg);
        })
        .catch(function() {});
    },
    /** 查询菜单下拉树结构 */
    getTreeselect() {
      listMenu().then(response => {
        this.menuOptions = [];
        const menu = { menuId: 0, menuName: "主类目", children: [] };
        menu.children = this.handleTree(response.data, "menuId");
        this.menuOptions.push(menu);
      });
    }
  }
};
</script>
