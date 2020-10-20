<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-col>
        <el-button class="addRoleBtn" type="primary">添加角色</el-button>
      </el-col>
      <el-table :data="rolesList" stripe border>
        <el-table-column type="expand" width="50">
          <template slot-scope="scope">
            <el-row :class="['bdbottom', i1===0?'bdtop':'' ,'vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag>{{item1.authName}}</el-tag><i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">

                <el-row :class="[i2===0?'':'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                  <!-- 渲染二级权限 -->
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag><i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 渲染三级权限 -->
                  <el-col :span="18">
                    <el-tag closable @close="removeRightById(scope.row,item3.id)" v-for="item3 in item2.children" :key="item3.id" type="warning">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#" width="50">
        </el-table-column>
        <el-table-column prop="roleName" label="角色名称">
        </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述">
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button icon="el-icon-edit" size='mini' type="primary">编辑</el-button>
            <el-button icon="el-icon-delete" size='mini' type="warning">删除</el-button>
            <el-button @click="shouSetRightDialog(scope.row)" icon="el-icon-setting" size='mini' type="danger">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 分配权限对话框 -->
    <el-dialog title="提示" :visible.sync="setRightDialogVisible" width="50%" @close="resetDefaultKeys">
      <!-- 树形控件 -->
      <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" :default-expand-all="true" :default-checked-keys="defaultKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights()">确 定</el-button>
      </span>
    </el-dialog>
  </div>

</template>

<script>
export default {
  data() {
    return {
      rolesList: [],
      rightsList: [],
      // 树形控件属性绑定对象
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认选中的节点id值
      defaultKeys: [],
      // 控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      roleId: ''
    };
  },
  created() {
    this.getRolesList();
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles');
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色信息失败');
      }
      this.rolesList = res.data;
      console.log(res);
    },
    // 根据id删除对应的权限
    async removeRightById(role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err);
      console.log(confirmResult);
      if (confirmResult !== 'confirm') return this.$message.error('您已取消该操作');
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`);
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败');
      }
      this.$message.success('删除权限成功');
      role.children = res.data;
    },
    async shouSetRightDialog(role) {
      this.roleId = role.id;
      const { data: res } = await this.$http.get('rights/tree');
      if (res.meta.status !== 200) return this.$message.error('获取权限列表失败');
      this.rightsList = res.data;
      this.setRightDialogVisible = true;
      this.getLeafKeys(role, this.defaultKeys);
    },
    // 通过递归的形式，获取角色下所有三级权限的id，并保存到defkeys数组中
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id);
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr);
      });
    },
    resetDefaultKeys() {
      this.defaultKeys = [];
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [this.$refs.treeRef.getCheckedKeys(), this.$refs.treeRef.getHalfCheckedKeys()];
      const idStr = keys.join(',');
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, {
        rids: idStr
      });
      if (res.meta.status !== 200) return this.$message.error('分配权限失败');
      this.$message.success('分配权限成功！');
      this.getRolesList();
      this.setRightDialogVisible = false;
    }
  }
};
</script>

<style lang="less" scoped>
.addRoleBtn {
  margin-bottom: 15px;
}
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
