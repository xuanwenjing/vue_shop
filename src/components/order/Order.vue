<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col :span='8'>
          <el-input placeholder="请输入fsdfd容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 订单列表数据 -->
      <el-table :data='orderList' stripe border>
        <el-table-column type="index">
        </el-table-column>
        <el-table-column label="订单编号" prop="order_number">
        </el-table-column>
        <el-table-column label="订单价格" prop="order_price">
        </el-table-column>
        <el-table-column label="是否付款" prop="pay_status">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status==='1'">已付款</el-tag>
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send">
        </el-table-column>
        <el-table-column label="下单时间">
          <template slot-scope="scope">
            {{scope.row.create_time | dataFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="">
            <el-button icon="el-icon-edit" @click="showBox" type="primary" size="mini"></el-button>
            <el-button icon="el-icon-location" type="success" size="mini" @click="showProgressBox"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[5, 10, 20, 40]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>

    <!-- 修改对话框 -->
    <el-dialog title="修改地址对话框" :visible.sync="editAddressDialogVisible" width="50%" @close='addressDialogClosed'>
      <el-form :model="addressForm" :rules="addressFormRules" ref="addressFormRef" label-width="100px">
        <el-form-item label="省市区/县" prop="address">
          <el-cascader v-model="addressForm.address" :options="cityData" :props="addressProps"></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="detailAddress">
          <el-input v-model="addressForm.detailAddress"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editAddressDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editAddressDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 物流进度对话框 -->
    <el-dialog title="物流进度" :visible.sync="pregressVisible" width="50%">
      <el-timeline :reverse="reverse">
        <el-timeline-item v-for="(activity, index) in activities" :key="index" :timestamp="activity.timestamp">
          {{activity.content}}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>

  </div>
</template>

<script>
import cityData from './citydata.js';
export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      orderList: [],
      editAddressDialogVisible: false,
      addressFormRules: {
        address: [{ required: true, message: '请选择地址', trigger: 'blur' }],
        detailAddress: [{ required: true, message: '请输入详细地址', trigger: 'blur' }]
      },
      addressForm: {
        address: '',
        detailAddress: ''
      },
      cityData,
      addressProps: {
        expandTrigger: 'hover',
        children: 'children',
        value: 'value',
        label: 'label'
      },
      pregressVisible: false,
      reverse: true,
      activities: [
        {
          content: '活动按期开始',
          timestamp: '2018-04-15'
        },
        {
          content: '通过审核',
          timestamp: '2018-04-13'
        },
        {
          content: '创建成功',
          timestamp: '2018-04-11'
        }
      ]
    };
  },
  created() {
    this.getOrderList();
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get('orders', { params: this.queryInfo });
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表失败');
      }
      this.total = res.data.total;
      this.orderList = res.data.goods;
    },
    handleSizeChange(newPageSize) {
      this.queryInfo.pagesize = newPageSize;
      this.getOrderList();
    },
    handleCurrentChange(newPageNum) {
      this.queryInfo.pagenum = newPageNum;
      this.getOrderList();
    },
    showBox() {
      this.editAddressDialogVisible = true;
    },
    addressDialogClosed() {
      this.$refs.addressFormRef.resetFields();
    },
    async showProgressBox() {
      // const { data: res } = await this.$http.get('/kuaidi/804909574412544580');
      // if (res.meta.status !== 200) {
      //   return this.$message.error('获取物流信息失败');
      // }
      // this.$message.success('获取物流信息成功');
      this.pregressVisible = true;
      // console.log(res.data);
    }
  }
};
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
