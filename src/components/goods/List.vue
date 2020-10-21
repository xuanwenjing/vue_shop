<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddPage"> 添加商品</el-button>
        </el-col>
      </el-row>
      <!-- table 表格区 -->
      <el-table :data='goodsList' border stripe>
        <el-table-column type='index' label='#'></el-table-column>
        <el-table-column label='商品名称' prop="goods_name"></el-table-column>
        <el-table-column label='商品价格（元）' width="105px" prop="goods_price"></el-table-column>
        <el-table-column label='商品重量' width="95px" prop="goods_weight"></el-table-column>
        <el-table-column label='商品创建时间' width="140px" prop="add_time">
          <template slot-scope="scope">
            {{scope.row.add_time | dataFormat}}
          </template>
        </el-table-column>
        <el-table-column label='操作' width="130px">
          <template slot-scope="">
            <el-button icon="el-icon-edit" type="primary" size='mini'></el-button>
            <el-button icon="el-icon-delete" type="danger" size='mini'></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[5, 10, 15, 20]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total" background>
      </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      goodsList: []
    };
  },
  created() {
    this.getGoodsList();
  },
  methods: {
    async getGoodsList() {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo });
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败');
      }
      this.$message.success('获取商品列表成功');
      this.goodsList = res.data.goods;
      this.total = res.data.total;
      console.log(res.data);
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getGoodsList();
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getGoodsList();
    },
    goAddPage() {
      this.$router.push('/goods/add');
    }
  }
};
</script>
<style lang="less" scoped>
</style>
