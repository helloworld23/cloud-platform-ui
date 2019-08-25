<template>
<div class="app-container calendar-list-container">
  <div class="filter-container">
    <el-input @keyup.enter.native="handleFilter" style="width: 100px;"  placeholder="租户名称" v-model="listQuery.name"> </el-input>
    <el-input @keyup.enter.native="handleFilter" style="width: 100px;"  placeholder="节点IP" v-model="listQuery.name"> </el-input>
    <el-input @keyup.enter.native="handleFilter" style="width: 100px;"  placeholder="端口" v-model="listQuery.name"> </el-input>

    <el-select v-model="status" placeholder="服务状态">
    <el-option 
      v-for="item in optionsStatus"
      :key="item.value"
      :label="item.label"
      :value="item.value">
    </el-option>
  </el-select>
  <el-date-picker 
      v-model="timeRange"
      type="daterange"
      range-separator="至"
      start-placeholder="开始日期"
      end-placeholder="结束日期">
    </el-date-picker>
    <el-button  type="primary" v-waves icon="search" @click="handleFilter">搜索</el-button>
    <el-button  v-if="userManager_btn_add"  style="margin-left: 10px;" @click="handleCreate" type="primary" icon="edit">添加</el-button>
  </div>
  <el-table :key='tableKey' :data="list" v-loading.body="listLoading" border fit highlight-current-row style="width: 100%">
    <el-table-column width="150"  fixed align="center" label="租户名称"> <template scope="scope">
        <span>{{scope.row.name}}</span>
      </template> </el-table-column>
    <el-table-column width="150" align="center" label="手机"> <template scope="scope">
            <span>{{scope.row.contact}}</span>
          </template> </el-table-column>
    <el-table-column width="200" align="center" label="邮箱"> <template scope="scope">
            <span>{{scope.row.email}}</span>
          </template> </el-table-column>
    <el-table-column width="200" align="center" label="注册日期"> <template scope="scope">
            <span>{{scope.row.registrationDate}}</span>
          </template> </el-table-column>
    <el-table-column width="110" align="center" label="服务状态"> <template scope="scope">
       <div v-if="scope.row.node">
          <span>{{(scope.row.node.flag&&scope.row.node.flag==1)?'启用':'禁用'}}</span>
       </div>
         <div v-else><span>禁用</span></div>
        </template> </el-table-column>
    <el-table-column width="200" align="center" label="节点IP"> <template scope="scope">
      <div v-if="scope.row.node">
            <span>{{scope.row.node.ip}}</span>
            </div>
          </template> </el-table-column>
    <el-table-column width="100" align="center" label="端口"> <template scope="scope">
      <div v-if="scope.row.node">
      <span>{{scope.row.node.remotePort}}</span>
      </div>
    </template> </el-table-column>
    <el-table-column align="center"  fixed="right" label="操作" width="150"> <template scope="scope">
        <el-button v-if="userManager_btn_edit" size="small" type="success" @click="handleUpdate(scope.row)">编辑
        </el-button>
        <el-button v-if="userManager_btn_del" size="small" type="danger" @click="handleDelete(scope.row)">删除
        </el-button>
      </template> </el-table-column>
  </el-table>
  <div v-show="!listLoading" class="pagination-container">
    <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="listQuery.page" :page-sizes="[10,20,30, 50]" :page-size="listQuery.limit" layout="total, sizes, prev, pager, next, jumper" :total="total"> </el-pagination>
  </div>
  <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
    <el-form :model="form" :rules="rules" ref="form" label-width="100px">
      <el-form-item label="姓名" prop="name">
        <el-input v-model="form.name" placeholder="请输入姓名"></el-input>
      </el-form-item>
      <el-form-item label="账户" prop="username">
        <el-input v-if="dialogStatus == 'create'" v-model="form.username" placeholder="请输入账户"></el-input>
        <el-input v-else v-model="form.username" placeholder="请输入账户" readonly></el-input>
      </el-form-item>
      <el-form-item v-if="dialogStatus == 'create'" label="密码" placeholder="请输入密码" prop="password">
        <el-input type="password" v-model="form.password"></el-input>
      </el-form-item>
      <el-form-item label="性别">
        <el-select class="filter-item" v-model="form.sex" placeholder="请选择">
          <el-option v-for="item in  sexOptions" :key="item" :label="item" :value="item"> </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="描述">
        <el-input type="textarea" :autosize="{ minRows: 3, maxRows: 5}" placeholder="请输入内容" v-model="form.description"> </el-input>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="cancel('form')">取 消</el-button>
      <el-button v-if="dialogStatus=='create'" type="primary" @click="create('form')">确 定</el-button>
      <el-button v-else type="primary" @click="update('form')">确 定</el-button>
    </div>
  </el-dialog>
</div>
</template>

<script>
import {
  page
} from 'api/tenant/user/index';
import { mapGetters } from 'vuex';
export default {
  name: 'user',
  data() {
    return {
      status: "",
      timeRange: "",
      optionsStatus:[{
          value: '1',
          label: '启用'
        }, {
          value: '0',
          label: '禁用'
        }],
      form: {
        username: undefined,
        name: undefined,
        sex: '男',
        password: undefined,
        description: undefined
      },
      rules: {
        name: [
          {
            required: true,
            message: '请输入用户',
            trigger: 'blur'
          },
          {
            min: 3,
            max: 20,
            message: '长度在 3 到 20 个字符',
            trigger: 'blur'
          }
        ],
        username: [
          {
            required: true,
            message: '请输入账户',
            trigger: 'blur'
          },
          {
            min: 3,
            max: 20,
            message: '长度在 3 到 20 个字符',
            trigger: 'blur'
          }
        ],
        password: [
          {
            required: true,
            message: '请输入密码',
            trigger: 'blur'
          },
          {
            min: 5,
            max: 20,
            message: '长度在 5 到 20 个字符',
            trigger: 'blur'
          }
        ]
      },
      list: [],
      total: null,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 10,
        name: undefined
      },
      sexOptions: ['男', '女'],
      dialogFormVisible: false,
      dialogStatus: '',
      userManager_btn_edit: false,
      userManager_btn_del: false,
      userManager_btn_add: false,
      textMap: {
        update: '编辑',
        create: '创建'
      },
      tableKey: 0
    }
  },
  created() {
    this.getList();
    this.userManager_btn_edit = this.elements['userManager:btn_edit'];
    this.userManager_btn_del = this.elements['userManager:btn_del'];
    this.userManager_btn_add = this.elements['userManager:btn_add'];
  },
  computed: {
    ...mapGetters([
      'elements'
    ])
  },
  methods: {
    getList() {
      this.listLoading = true;
      page(this.listQuery)
        .then(response => {
          this.list = response.data.rows;
          this.total = response.data.total;
          this.listLoading = false;
        })
    },
    handleFilter() {
      this.getList();
    },
    handleSizeChange(val) {
      this.listQuery.limit = val;
      this.getList();
    },
    handleCurrentChange(val) {
      this.listQuery.page = val;
      this.getList();
    },
    handleCreate() {
      this.resetTemp();
      this.dialogStatus = 'create';
      this.dialogFormVisible = true;
    },
    handleUpdate(row) {
      getObj(row.id)
        .then(response => {
          this.form = response.data;
          this.dialogFormVisible = true;
          this.dialogStatus = 'update';
        });
    },
    handleDelete(row) {
      this.$confirm('此操作将永久删除, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          delObj(row.id)
            .then(() => {
              this.$notify({
                title: '成功',
                message: '删除成功',
                type: 'success',
                duration: 2000
              });
              const index = this.list.indexOf(row);
              this.list.splice(index, 1);
            });
        });
    },
    create(formName) {
      const set = this.$refs;
      set[formName].validate(valid => {
        if (valid) {
          addObj(this.form)
            .then(() => {
              this.dialogFormVisible = false;
              this.getList();
              this.$notify({
                title: '成功',
                message: '创建成功',
                type: 'success',
                duration: 2000
              });
            })
        } else {
          return false;
        }
      });
    },
    cancel(formName) {
      this.dialogFormVisible = false;
      this.$refs[formName].resetFields();
    },
    update(formName) {
      const set = this.$refs;
      set[formName].validate(valid => {
        if (valid) {
          this.dialogFormVisible = false;
          this.form.password = undefined;
          putObj(this.form.id, this.form).then(() => {
            this.dialogFormVisible = false;
            this.getList();
            this.$notify({
              title: '成功',
              message: '创建成功',
              type: 'success',
              duration: 2000
            });
          });
        } else {
          return false;
        }
      });
    },
    resetTemp() {
      this.form = {
        username: undefined,
        name: undefined,
        sex: '男',
        password: undefined,
        description: undefined
      };
    }
  }
}
</script>
