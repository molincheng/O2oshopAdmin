<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button @click="addDialogVisible=true" type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom',i1 === 0 ? 'bdtop' : '', 'venter']"
              v-for="(item1,i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 熏染一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二三级权限 -->
              <el-col :span="19">
                <!-- 通过for循环嵌套二级权限 -->
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop', 'venter']"
                  v-for="(item2,i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row,item2.id)"
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="(item3,i3) in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row,item3.id)"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showRolesEditDialog(scope.row.id)"
            >编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRolesById(scope.row.id)">删除</el-button>
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限的对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="showSetRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <!-- 树形控件 -->
      <el-tree
        :data="rightslist"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="showSetRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 添加角色对话框 -->
    <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <el-form :model="addFrom" :rules="addFromRules" ref="addFromRef" label-width="100px">
        <el-form-item label="角色名称" prop="rolename">
          <el-input v-model="addFrom.rolename"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="addFrom.roledesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoles">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改角色对话框 -->
    <el-dialog title="修改角色" :visible.sync="editRolesDialogVisible" width="50%" @close="editRolesDialogClose">
      <el-form :model="editForm" :rules="editRolesFormRules" ref="editRolesFormRef" label-width="100px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRolesDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRolesInfo()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 所有角色列表数据
      rolelist: [],
      //控制分配权限对话框的显示与隐藏
      showSetRightDialogVisible: false,
      // 所有权限的数组
      rightslist: [],
      // 树形控件的绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选择的节点Id值数组
      defKeys: [],
      // 当前即将分配权限的角色id
      roleId: '',
      // 控制添加角色对话框的显示与隐藏
      addDialogVisible: false,
      // 添加角色的表单数据
      addFrom: { rolename: '', roledesc: '' },
      // 添加表单的验证规则对象
      addFromRules: {
        rolename: [
          { required: true, message: '请输入角色名称！', trigger: 'blur' },
          {
            min: 4,
            max: 15,
            message: '角色名称长度在3~25个字符之间！',
            trigger: 'blur'
          }
        ]
      },
      // 查询到的角色用户信息
      editForm: {},
      // 控制修改角色对话框
      editRolesDialogVisible: false,
      editRolesFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称！', trigger: 'blur' },
          {
            min: 2,
            max: 15,
            message: '角色名称长度在2~25个字符之间！',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
      this.rolelist = res.data
      console.log(this.rolelist)
    },
    // 根据ID删除相应的权限
    async removeRightById(role, rightId) {
      // 弹框提示用户是否删除
      const confirmResult = await this.$confirm(
        '此操作将永久删除该文件, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除！')
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      if (res.meta.status !== 200) return this.$message.error('删除权限失败')
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败！')
      }
      //  获取到的权限数据保存到data中
      this.rightslist = res.data
      console.log(this.rightslist)
      // 递归获取三级节点的id
      this.getLeafKeys(role, this.defKeys)

      this.showSetRightDialogVisible = true
    },
    // 通过递归的形式，获取角色下所有三级权限的id，并保存到defKeys数组中
    getLeafKeys(node, arr) {
      if (!node.children) {
        // 如果当前node节点不包含children属性，则是三级节点
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        this.$refs.treeRef.getCheckedKeys(),
        this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败!')
      }
      this.$message.success('分配权限成功!')
      this.getRolesList()
      this.showSetRightDialogVisible = false
    },
    // 监听添加角色对话框的关闭事件
    addDialogClosed() {
      this.$refs.addFromRef.resetFields()
    },
    // 点击按钮，添加角色
    addRoles() {
      this.$refs.addFromRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', {
          roleName: this.addFrom.rolename,
          roleDesc: this.addFrom.roledesc
        })
        if (res.meta.status !== 201) {
          this.$message.error('角色添加失败！')
        }
        this.$message.success('添加角色成功！')
        this.addDialogVisible = false
        this.getRolesList()
      })
    },
    // 显示修改角色对话框
    async showRolesEditDialog(id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询修改用户信息失败！')
      }
      this.editForm = res.data
      console.log(this.editForm)
      this.editRolesDialogVisible = true
    },
    // 修改角色信息并提交
    editRolesInfo() {
      this.$refs.editRolesFormRef.validate(async valid => {
       if (!valid) return
      //  发起修改用户的数据请求
      const {data:res} = await this.$http.put('roles/'+this.editForm.roleId,{roleName:this.editForm.roleName,roleDesc:this.editForm.roleDesc})
      if (res.meta.status !== 200) {
        return this.$message.error('更新角色信息失败！')
      }
      // 关闭对话框
      this.editRolesDialogVisible = false
      // 刷新数据列表
      this.getRolesList()
      // 提示修改角色成功
      this.$message.success('更新角色信息成功!')
      })
    },
    // 监听修改角色对话框关闭事件
    editRolesDialogClose(){
      this.$refs.editRolesFormRef.resetFields()
    },
    // 根据id删除对应的角色信息
    async removeRolesById(id){
      // 弹框询问用户是否删除数据
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 如果用户确认删除，则返回值为字符串 confirm
      // 如果用户取消了删除，则返回值为z字符串 cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除角色！')
      }
     const {data:res} = await this.$http.delete('roles/'+id)
     if (res.meta.status !== 200) {
       return this.$message.error('删除角色失败!')
     }
     this.$message.success('删除角色成功!')
     this.getRolesList()
    }
  }
}
</script>
<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.venter {
  display: flex;
  align-items: center;
}
</style>