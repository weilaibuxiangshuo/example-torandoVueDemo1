<template>
  <div id="databox">
    <div class="dataheader">
      <div class="firstheader"></div>
      <!-- 添加及搜索 -->
      <div class="secondheader">
        <el-form ref="btnForm" :inline="true" class="demo-ruleForm">
          <el-form-item>
            <el-button @click.native.prevent="addData" type="success" class="addPerStyle">添加角色</el-button>
          </el-form-item>
          <el-form-item>
            <el-input
              placeholder="角色搜索"
              v-model.trim="search"
              clearable
              @change="serchPutChange"
              style="width:350px"
            >
              <el-button slot="append" icon="el-icon-search" @click.native.prevent="btnSearch"></el-button>
            </el-input>
          </el-form-item>
        </el-form>
      </div>
      <hr />
    </div>
    <div class="datamain">
      <!-- 弹窗对话框  -->
      <el-dialog
        :title="title==='NEW'?'添加角色':'编辑角色'"
        :visible.sync="dialogFormVisible"
        :before-close="handleClose"
      >
        <el-form :model="dialogform" ref="ruleForm" :rules="rules">
          <el-form-item label="角色名称" :label-width="formLabelWidth" prop="title">
            <el-input v-model.trim="dialogform.title" autocomplete="off" clearable></el-input>
          </el-form-item>
          <el-form-item label="关联权限" :label-width="formLabelWidth">
            <el-tree
              :data="selectData"
              show-checkbox
              default-expand-all
              node-key="id"
              ref="tree"
              highlight-current
              :props="defaultProps"
              class="treeStyle"
            ></el-tree>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click.native.prevent="cancel">取 消</el-button>
          <el-button type="primary" @click.native.prevent="confirm">确 定</el-button>
        </div>
      </el-dialog>

      <!-- 表格渲染 -->
      <el-table ref="tableForm" :data="tableData" style="width: 100%" border>
        <el-table-column type="selection" width="55"></el-table-column>
        <el-table-column type="index" label="ID" fixed="left"></el-table-column>
        <el-table-column label="角色名称" prop="title" fixed="left" width="150px"></el-table-column>
        <el-table-column label="关联权限" prop="permission">
          <template v-slot="perscope">
            <div>
              {{ perscope.row.permission | filterper }}
            </div>
          </template>
        </el-table-column>
        <el-table-column align="left" label="编辑" fixed="right" width="150px">
          <template v-slot="scope">
            <el-button size="mini" @click.native.prevent="handleEdit(scope.$index, scope.row)">Edit</el-button>
            <el-button
              size="mini"
              type="danger"
              @click.native.prevent="handleDelete(scope.$index, scope.row)"
            >Delete</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <!-- 分页及按钮 -->
    <div class="datafooter">
      <div class="btnStyle">
        <el-button type="primary" icon="el-icon-share" @click.native.prevent="selectAllData">全选</el-button>
        <el-button
          type="primary"
          icon="el-icon-delete"
          class="btnDelStyle"
          @click.native.prevent="delAllData"
        >全删</el-button>
      </div>
      <div class="paginationStyle">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="pagin.currentPage"
          :page-sizes="pagin.pagesizes"
          :page-size="pagin.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="pagin.total"
        ></el-pagination>
      </div>
    </div>
  </div>
</template>
<script>
// 弹窗对话框初始值
const defaultDialogForm = {
  id: "",
  title: "",
};

// 分页初始值
const defaultPagin = {
  currentPage: 1,
  pagesizes: [10, 20, 50],
  pagesize: 10,
  total: 0
};

export default {
  data() {
    return {
      // -----------添加及搜索-----------
      search: "",
      // ----------- 弹窗对话框-----------
      dialogFormVisible: false,
      title: "NEW",
      dialogform: Object.assign({}, defaultDialogForm),
      formLabelWidth: "100px",
      selectData: [],
      defaultProps:{
      children: 'children',
      label: 'title'
      },
      // ----------- 表格渲染-----------
      tableData: [],
      // ----------- 分页-----------
      pagin: Object.assign({}, defaultPagin),
      // ----------- 验证-----------
      rules: {
        title: [
          { required: true, message: "请输入菜单名称", trigger: "blur" },
          { min: 3, max: 32, message: "长度在 3 到 32 个字符", trigger: "blur" }
        ],
      }
    };
  },
  methods: {
    // -----------添加及搜索-----------
    // 添加目标
    addData() {
      this.title = "NEW";
      this.dialogform = Object.assign({}, defaultDialogForm);
      this.$nextTick(()=>{
        this.$refs['tree'].setCheckedKeys([])
      })
      this.dialogFormVisible = true;
    },
    // 搜索框没有值时触发
    serchPutChange(val) {
      let newVal = val.toString().trim();
      if (newVal == "") {
        this.getAllInfo();
      } else {
        return false;
      }
    },
    // 按下搜索时触发
    btnSearch() {
      this.getAllInfo();
    },
    // ----------- 弹窗对话框-----------
    //点击对话框外面事件
    handleClose(done) {
      this.dialogFormVisible = false;
      this.dialogform = Object.assign({}, defaultDialogForm);
      this.$refs["ruleForm"].resetFields();
      this.$refs.tree.setCheckedKeys([]);
      done();
    },
    // 确定送出
    confirm() {
      this.$refs["ruleForm"].validate(val => {
        if (val) {
          this.dialogFormVisible = false;
          this.dialogform.permission = this.$refs['tree'].getCheckedKeys();
          let menuData = JSON.stringify(this.dialogform);
          if (this.title === "EDIT") {
            this.$store
              .dispatch("roles/PutRole", menuData)
              .then(response => {
                this.getAllInfo();
              });
          } else {
            this.$store
              .dispatch("roles/AddRole", menuData)
              .then(response => {
                this.getAllInfo();
              });
          }
        }
      });
    },
    // 取消对话框
    cancel() {
      this.dialogFormVisible = false;
      this.dialogform = Object.assign({}, defaultDialogForm);
      this.$refs["ruleForm"].resetFields();
      this.$refs.tree.setCheckedKeys([]);
    },

    // ----------- 表格渲染-----------
    // 获取所有目标
    getAllInfo() {
      // this.pagin = Object.assign({},defaultPagin)
      if (
        (this.pagin.currentPage - 1) * this.pagin.pagesize ==
          this.pagin.total &&
        this.pagin.currentPage - 1 > 0
      ) {
        this.pagin.currentPage -= 1;
      }
      let dataInfo = {
        currentpage:
          this.pagin.currentPage === null ? 1 : this.pagin.currentPage,
        pagesize: this.pagin.pagesize
      };
      if (!!this.search.toString().trim()) {
        dataInfo["search"] = this.search.toString().trim();
        // 有搜索需要恢复初始值，否则会显示错误
        dataInfo["currentpage"] = 1;
        this.pagin.currentPage = 1;
      }
      this.$store.dispatch("roles/GetRole", dataInfo).then(response => {
        this.tableData = response.data;
        this.pagin.total = response.total;
      });
      this.$store.dispatch("roles/GetRolePermission").then(response => {
        const { data } = response;
        this.selectData = data;
      });
    },
    // 编辑
    handleEdit(index, row) {
      this.dialogform = Object.assign({}, defaultDialogForm);
      this.title = "EDIT";
      this.dialogFormVisible = true;
      this.dialogform.id = row.id;
      this.dialogform.title = row.title;
      let arr1 = []
      for(let n in row.permission){
        arr1.push(row.permission[n].id)
      }
      this.$nextTick(()=>{
        this.$refs.tree.setCheckedKeys(arr1)
      })
    },
    // 删除
    handleDelete(index, row) {
      let arr1 = [];
      arr1.push(row.id);
      this.$store
        .dispatch("roles/DelRole", JSON.stringify({ id: arr1 }))
        .then(response => {
          this.pagin.total -= 1;
          this.getAllInfo();
        });
    },
    // ----------- 分页-----------
    // 全选
    selectAllData() {
      this.$refs.tableForm.toggleAllSelection();
    },
    // 全删
    delAllData() {
      let arr1 = [];
      const temp = this.$refs.tableForm.selection;
      for (let t in temp) {
        arr1.push(temp[t].id);
        this.pagin.total -= 1;
      }
      this.$store
        .dispatch("roles/DelRole", JSON.stringify({ id: arr1 }))
        .then(response => {
          this.getAllInfo();
        });
    },
    // 当前一页显示数量
    handleSizeChange(val) {
      let isR = val * this.pagin.currentPage > this.pagin.total;
      if (isR) {
        this.pagin.pagesize = val;
        this.handleCurrentChange(1);
        return false;
      }
      this.pagin.pagesize = val;
      this.getAllInfo();
    },
    // 当前页码
    handleCurrentChange(val) {
      this.pagin.currentPage = val;
      this.getAllInfo();
    }
  },
  created() {
    this.getAllInfo();
  },
  filters:{
    "filterper":function(val){
      let arr1 = []
      for(let n in val){
        arr1.push(val[n].title)
      }
      return arr1.join(",")
    }
  }
};
</script>
<style scoped>

.treeStyle {
  margin-top:7px;
}

.dataheader {
  margin: 10px 0px 0px 20px;
}

.datamain {
  margin-left: 22px;
}

.el-select {
  display: block;
}
.el-button {
  margin-left: 0px;
}
.secondheader {
  margin-left: 10px;
}
.msgheader {
  display: inline-block;
  margin-left: 30px;
}
.msgheader span:nth-child(2) {
  margin-left: 10px;
  color: red;
}
.msgheader span:last-child {
  margin-left: 10px;
}
.btnStyle {
  display: inline-block;
  margin-top: 20px;
  margin-left: 30px;
  vertical-align: middle;
}

.paginationStyle {
  margin-top: 20px;
  margin-left: 30px;
  display: inline-block;
  vertical-align: middle;
}

.btnDelStyle {
  background-color: #f56c6c;
  border-color: #f56c6c;
}

</style>