<template>
  <div class="table">
    <div class="crumbs">
      <el-breadcrumb>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
      <div class="search-box">
        <div class="handle-box">
          <el-button v-if="getAlcsObj.YHXZ" type="primary" icon="el-icon-plus" @click="addUsers">新增用户</el-button>
        </div>
        <el-form ref="search" :model="userSearch" class="demo-form-inline" :inline="true">
          <el-form-item label="状态：">
            <el-select class="el-select-width" v-model="userSearch.status" @change="search" clearable>
              <el-option v-for="item in status" :key="item.num" :value="item.value" :label="item.label">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="角色：">
            <el-select class="el-select-width" v-model="userSearch.roleId" clearable @change="search">
              <el-option v-for="item in allRoles" :key="item.value" :value="item.value" :label="item.label">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="用户名称：">
            <el-input class="el-input-width" v-model.trim="userSearch.username" @change="search" placeholder="输入用户名称查询" clearable maxlength="30"></el-input>
          </el-form-item>
          <el-form-item label="真实姓名：">
            <el-input class="el-input-width" v-model.trim="userSearch.name" @change="search" placeholder="输入用户真实姓名查询" clearable maxlength="30"></el-input>
          </el-form-item>
          <el-form-item label="工号：">
            <el-input class="el-input-width" v-model.trim="userSearch.code" @change="search" placeholder="输入工号查询" clearable maxlength="30"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="search">搜索</el-button>
          </el-form-item>
        </el-form>
      </div>
      <el-table :data="UserList" stripe style="width: 100%;" ref="multipleTable" height="550" fit :row-class-name="tableRowStatusName">
        <el-table-column prop="index" label="序号" width="60">
        </el-table-column>
        <el-table-column prop="username" label="用户名称">
        </el-table-column>
        <el-table-column prop="Status" label="状态" width="60">
        </el-table-column>
        <el-table-column prop="roleName" label="角色">
        </el-table-column>
        <el-table-column prop="name" label="真实姓名">
        </el-table-column>
        <el-table-column prop="code" label="工号">
        </el-table-column>
        <el-table-column prop="phone" label="电话">
        </el-table-column>
        <el-table-column prop="email" label="邮箱">
        </el-table-column>
        <el-table-column prop="updatedAt" label="更新时间">
        </el-table-column>
        <el-table-column label="操作" width="150">
          <template slot-scope="scope">
            <el-button v-if="getAlcsObj.YHXG" type="text" size="small" @click="handleEdit(scope.row, scope.$index)">修改</el-button>
            <el-button v-if="scope.row.isButtonShow && getAlcsObj.YHSZZT" type="text" @click="handleEffective(scope.row, scope.$index)" size="small">置为有效</el-button>
            <el-button v-if="!scope.row.isButtonShow && getAlcsObj.YHSZZT" type="text" @click="handleInvalid(scope.row, scope.$index)" size="small">置为无效</el-button>
            <el-button v-if="scope.row.isButtonShow && getAlcsObj.YHSC" size="small" type="text" @click="handleDelete(scope.row, scope.$index)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <ve-pagination :cur_page="cur_page" :pageSize="pageSize" :count="count" @handleCurrentChange="handleCurrentChange" @handleSizeChange="handleSizeChange" @gofist="gofist" @goLast="goLast"></ve-pagination>
    </div>
    <!-- 编辑弹出框 -->
    <el-dialog title="编辑" :visible.sync="editVisible" width="30%" :before-close="editDiaClose" :close-on-click-modal="false">
      <el-form ref="editUser" :model="editUser" label-width="100px" :rules="UserRule" label-position="rigth">
        <!-- <el-form-item label="用户名称:" prop="username">
          <el-input v-model.trim="editUser.username"></el-input>
        </el-form-item> -->
        <el-form-item label="真实姓名:" prop="name">
          <el-input v-model="editUser.name" maxlength="30"></el-input>
        </el-form-item>
        <el-form-item label="密码:">
          <el-input v-model="editUser.password" maxlength="30"></el-input>
        </el-form-item>
        <el-form-item label="工号:" prop="code">
          <el-input v-model="editUser.code" maxlength="30" clearable></el-input>
        </el-form-item>
        <el-form-item label="用户角色:" prop="roles">
          <el-select v-model="editUser.roles" multiple collapse-tags>
            <el-option v-for="item in editUserRoles" :key="item.value" :value="item.value" :label="item.label">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="电话:" prop="phone">
          <el-input v-model.trim="editUser.phone" maxlength="11"></el-input>
        </el-form-item>
        <el-form-item label="邮箱:" prop="email">
          <el-input v-model.trim="editUser.email" maxlength="30"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="saveEdit">确 定</el-button>
        <el-button @click="editVisible=false">取 消</el-button>
      </div>
    </el-dialog>

    <!-- 新增用户 -->
    <el-dialog title="新增用户" :visible.sync="addVisible" width="30%" :before-close="addDiaClose" :close-on-click-modal="false">
      <el-form ref="addusers" :model="addUser" label-width="100px" :rules="UserRule" label-position="rigth" class="demo-ruleForm">
        <el-form-item label="用户名称:" prop="username">
          <el-input v-model.trim="addUser.username" maxlength="30" clearable></el-input>
        </el-form-item>
        <el-form-item label="真实姓名:" prop="name">
          <el-input v-model.trim="addUser.name" maxlength="30" clearable></el-input>
        </el-form-item>
        <el-form-item label="工号:" prop="code">
          <el-input v-model.trim="addUser.code" maxlength="30" clearable></el-input>
        </el-form-item>
        <el-form-item label="用户角色:" prop="roles">
          <el-select class="inputs" v-model="addUser.roles" placeholder="请选择角色" multiple collapse-tags>
            <el-option v-for="item in userRoles" :key="item.value" :label="item.label" :value="item.value">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="密码:" prop="password">
          <el-input v-model.trim="addUser.password" maxlength="30"></el-input>
        </el-form-item>
        <el-form-item label="电话:" prop="phone">
          <el-input v-model.number="addUser.phone" maxlength="11" clearable></el-input>
        </el-form-item>
        <el-form-item label="邮箱:" prop="email">
          <el-input v-model.trim="addUser.email" :clearable="true" maxlength="30"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="saveAdd('addUser')">确 定</el-button>
        <el-button @click="cancelAddUser">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
  import { mapActions, mapGetters } from 'vuex';
  import { getNowFormatDate } from '../../utils';
  import { checkPhone, checkUsername } from '../../utils/rules';
  export default {
    name: 'user',
    data() {
      return {
        count: 0,
        UserList: [],
        addVisible: false,
        addUser: {
          roles: [],
          username: '',
          password: '',
          phone: '',
          email: '',
          remark: '',
          workNumber: '',
          name: '',
          code: ''
        },
        UserRule: {
          username: [
            { validator: checkUsername, trigger: 'blur' },
            { required: true, trigger: 'blur', message: '用户名不能为空' }],
          name: [
            { validator: checkUsername, trigger: 'blur' },
            { required: true, message: '真实姓名不能为空', trigger: 'blur' }],
          code: [{ validator: checkUsername, trigger: 'blur' }],
          roles: [{ required: true, trigger: 'blur', message: '用户角色不能为空' }],
          password: [
            { max: 30, trigger: 'blur' },
            { required: true, trigger: 'blur', message: '密码不可为空' }],
          phone: [
            { validator: checkPhone, trigger: 'blur' }
          ],
          email: [
            { message: '请输入邮箱地址', trigger: 'blur' },
            { type: 'email', message: '请按正确的邮箱格式输入', trigger: 'blur' }
          ]
        },
        newAddUser: {
          roles: [],
          username: '',
          password: '',
          phone: '',
          email: '',
          remark: ''
        },
        //状态
        status: [
          {
            value: 1,
            label: '有效'
          },
          {
            value: 0,
            label: '无效'
          }
        ],
        editVisible: false,
        editUser: {},
        options: [],
        allRoles: [],
        userRoles: [],
        editUserRoles: [],
        idx: -1,
        delIndex: Number,
        delVisible: false,
        RoleID: Number,
        cur_page: 1,
        pageSize: 10,
        multipleSelection: [],
        userSearch: {
          status: '',
          username: '',
          name: '',
          code: ''
        }
      }
    },
    created() {
      this.getUsers();
      this.getUserRoles();
      this.getRoles();
    },
    computed: {
      ...mapGetters(['getAlcs', 'getAlcsObj'])
    },
    methods: {
      ...mapActions(['ajax']),
      // 分页导航
      handleCurrentChange(val) {
        this.cur_page = val;
        this.getUsers();
      },
      handleSizeChange(val) {
        this.pageSize = val;
        this.getUsers();
      },
      gofist() {
        this.cur_page = 1;
        this.getUsers();
      },
      goLast() {
        this.cur_page = Math.ceil(this.count / this.pageSize);
        this.getUsers();
      },
      getUsers() {
        let d = this.cur_page; //当前页
        let m = this.pageSize; //每页显示条数
        let count = (d - 1) * m + 1 === 0 ? 1 : (d - 1) * m + 1;
        this.ajax({
          name: 'getUsers',
          data: {
            pageNum: this.cur_page,
            pageSize: this.pageSize,
            ...this.userSearch
          }
        }).then(res => {

          res.rows.forEach((item, index) => {
            switch(item.status) {
              case 0:
                item.Status = '无效';
                item.isButtonShow = true;
                break;
              case 1:
                item.Status = '有效';
                item.isButtonShow = false;
                break;
              case 2:
                item.Status = '删除'
                break;
              default:
                break;
            }
            let str = '';
            item.roles.forEach((item, index) => {
              str += item.name + ','
            });
            item.roleName = str.substr(0, str.length - 1);
            item.index = count++;
          });
          this.UserList = res.rows;
          this.count = res.count; //分页

        });
      },
      getUserRoles() {
        this.ajax({
          name: 'getUserRoles'
        }).then(res => {
          res.forEach(item => {
            let obj = {}
            obj.value = item.id;
            obj.label = item.name;
            this.userRoles.push(obj);
          });
        })
      },
      getRoles() {
        this.ajax({
          name: 'getAllRoles'
        }).then(res => {
          res.rows.forEach(item => {
            let obj = {}
            obj.value = item.id;
            obj.label = item.name;
            this.allRoles.push(obj);
          });
        })
      },
      addDiaClose(done) {
        this.$refs.addusers.resetFields();
        done();
      },
      //新增用户
      addUsers() {
        this.addVisible = true;
      },
      saveAdd(ruleName) {
        this.$refs.addusers.validate((valid) => {
          if(valid) {
            this.ajax({
              name: 'addUser',
              data: this.addUser
            }).then(res => {
              this.getUsers();
              this.addVisible = false;
              this.$message.success('新增用户成功');
            });
          } else {
            return false;
          }
        });

      },
      //取消 
      cancelAddUser() {
        this.$refs.addusers.resetFields();
        this.addVisible = false;
      },
      editDiaClose(done) {
        this.$refs.editUser.resetFields();
        done();
      },
      //编辑
      handleEdit(row, index) {
        this.editVisible = true;
        this.$nextTick(() => {
          let currentRoles = row.roles.map(r => ({ value: r.id, label: r.name })).concat(this.userRoles);
          this.editUserRoles = [];
          currentRoles.forEach(item => {
            if(this.editUserRoles.findIndex(f => f.value === item.value) === -1) {
              this.editUserRoles.push(item);
            }
          });
          this.editUser = {
            name: row.name,
            password: null,
            remark: row.remark,
            phone: row.phone,
            code: row.code,
            status: row.status,
            email: row.email,
            roles: row.roles.map(r => r.id)
          }
          this.idx = row.id;
        });
      },
      // 保存编辑
      saveEdit() {
        this.$refs.editUser.validate((valid) => {
          if(valid) {
            this.editVisible = false;
            this.ajax({
              name: 'editUser',
              data: this.editUser,
              id: this.idx
            }).then(res => {
              this.getUsers();
              this.$message.success('修改成功');
            });
          }
        });
      },
      getRoleName(roldId) {
        switch(roldId) {
          case 1:
            this.editUser.roleName = '超级管理员'
            break;
          case 2:
            this.editUser.roleName = '管理员'
            break;
          case 3:
            this.editUser.roleName = '用户'
          default:
            break;
        }
      },
      //删除
      handleDelete(row, index) {
        this.delIndex = index;
        this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          type: 'warning'
        }).then(() => {
          this.ajax({
            name: 'deleteUser',
            id: row.id
          }).then(res => {
            this.getUsers();
          });
        })
      },

      //搜索
      search() {
        this.getUsers();
        this.gofist();
      },
      //置为有效
      handleEffective(row, index) {
        this.ajax({
          name: 'setUserStatus',
          data: { id: row.id, status: 1 }
        }).then(res => {
          this.$message.success('操作成功');
          this.getUsers();
        });
      },
      //置为无效
      handleInvalid(row, index) {
        this.ajax({
          name: 'setUserStatus',
          data: { id: row.id, status: 0 }
        }).then(res => {
          this.$message.success('操作成功');
          this.getUsers();
        });
      },
      // tableRowStatusName 根据有效无效修改 row 样式
      tableRowStatusName({ row, rowIndex }) {
        if(row.status == 0) {
          return 'invalid-row'
        } else {
          return ''
        }
      }
      // delAll() {
      //   const length = this.multipleSelection.length;
      //   let str = '';
      //   this.del_list = this.del_list.concat(this.multipleSelection);
      //   for(let i = 0; i < length; i++) {
      //     str += this.multipleSelection[i].name + ' ';
      //   }
      //   this.$message.error('删除了' + str);
      //   this.multipleSelection = [];
      // },
      // handleSelectionChange(val) {
      //   this.multipleSelection = val;
      // }
    }
  }

</script>

<style scoped>
  .handle-select {
    width: 120px;
  }

  .handle-input {
    width: 300px;
    display: inline-block;
  }
  .del-dialog-cnt {
    font-size: 16px;
    text-align: center;
  }

  .inputs {
    width: 100%;
  }
</style>
