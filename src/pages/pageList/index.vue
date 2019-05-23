<template>
<el-container>
  <el-header>

  </el-header>
  <el-container>
    <el-aside width="200px">
      
    </el-aside>
    <el-container>
      <el-main>
         <el-table
            :data="tableData"
            stripe
            style="width: 100%">
            <el-table-column
              prop="id"
              label="ID"
              width="180">
            </el-table-column>
            <el-table-column
              prop="pageId"
              label="页面ID">
            </el-table-column>
            <el-table-column
              label="操作">
               <template slot-scope="scope">
                <el-button @click="handleClick(scope.row)" type="text" size="small">查看</el-button>
                 <el-button @click="publish(scope.row)" type="text" size="small">发布</el-button>
              </template>
            </el-table-column>
          </el-table>
      </el-main>
      <el-footer>
        published-url: {{url}}
      </el-footer>
    </el-container>
  </el-container>
</el-container>

</template>

<script>
import { APIS } from 'comp/util/constants';
import fetch from 'comp/util/fetch';
export default {
  components: {
  },
  data() {
    return {
      tableData: [],
      url: "",
      dataSchema: {
     
      },
    };
  },
  methods: {
    handleClick(item) {
      const {pageId} = item;
      this.$router.push(`pipeline?pageId=${pageId}`);
    },
    async publish(item) {
      const {pageId} = item;
      const ret = await fetch(`${APIS.PIPLINE}/publish`, {
        method: 'POST',
        body: {
          pageId
        },
      });
      // console.log(ret);
      this.url = ret.pageId;
      this.$message(ret.pageId);
    },
    async getPageList() {
      const ret = await fetch(`${APIS.PIPLINE}/pageList`, {
        method: 'POST',
        body: {
          templateId: this.templateId,
        },
      });
      console.log(ret.list);
      this.tableData = ret.list;
    }
  },
  mounted() {
    this.getPageList()
  }
};
</script>

<style lang="less" scoped>

</style>
