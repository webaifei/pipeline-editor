<template>
  <el-container>
    <el-header height="80">
      
    </el-header>
    <el-main>
      <div class="template-container">
        <div class="template-list">
          <div class="template-item template-item--add">
            <el-button type="primary" @click="addTemplate">新增模板</el-button>
            <router-link :to="{ name: 'pagelist'}">历史编辑页面</router-link>
          </div>
          <div class="template-item"
            v-for="(item) in templateList"
            :key="item.id">
            <div class="template-item__thumbnail">
              <img class="template-item__img" :src="getThumbnailUrl(item.thumbnail)" />
            </div>
            <div class="template-item__name">{{item.name}}</div>
            <div class="template-item__button-group">
                <el-button type="primary" size="small" plain @click="useTemplate(item)">使用</el-button>
                <el-button type="warning" size="small" plain @click="editTemplate(item)">修改</el-button>
                <el-button type="danger" size="small" plain @click="deleteTemplate(item)">删除</el-button>
            </div>
          </div>
        </div>
      </div>
    </el-main>
    <el-dialog
      title="输入活动名称"
      :visible.sync="centerDialogVisible"
      width="30%"
      center>
        <el-input v-model="name"></el-input>
        <span slot="footer" class="dialog-footer">
          <el-button type="primary" @click="sureUseTemplate">确 定</el-button>
        </span>
    </el-dialog>
  </el-container>
</template>

<script>

import { APIS } from 'comp/util/constants';
import fetch from 'comp/util/fetch';
import Topbar from 'comp/topbar';

export default {
  name: 'template-list',
  components: {
    topbar: Topbar,
  },
  data() {
    return {
      centerDialogVisible: false,
      name: "",
      templateId: "",
      templateList: [],
    };
  },
  methods: {
    async getTemplates() {
      const ret = await fetch(`${APIS.TEMPLATE}`, {
      });
      return ret;
    },
    async getTemplateId() {
      const ret = await fetch(`${APIS.ROOT}/templateid`, {
      });
      return ret;
    },
    useTemplate(template) {
      this.centerDialogVisible = true;
      this.templateId = template.id;
    },
    sureUseTemplate() {
      if(this.name === '') {
        return this.$message.error('活动名称不能为空');
      }
      this.$router.push({
        path: `/pipeline?templateId=${this.templateId}&name=${this.name}`,
      });
    },
    async addTemplate() {
      const templateId = await this.getTemplateId();
      this.$router.push({
        path: '/template/add',
        query: {
          templateId,
        },
      });
    },
    editTemplate(template) {
      const templateId = template.id;
      this.$router.push({
        path: '/template/edit',
        query: {
          templateId,
        },
      });
    },
    deleteTemplate(template) {
      this.$confirm('此操作将永久删除模板, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }).then(() => {
        // 删除模板
        fetch(`${APIS.TEMPLATE}/${template.id}`, {
          method: 'DELETE',
        }).then(() => {
          this.$message({
            type: 'success',
            message: '删除模板成功.',
          });
        }).then(async () => {
          // 更新模板
          this.templateList = await this.getTemplates();
        }).catch(() => {
          this.$message({
            type: 'warning',
            message: '删除失败, 请检查网络.',
          });
        });
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除',
        });
      });
    },
    getThumbnailUrl(url) {
      if (/^https?:\/\//.test(url)) {
        return url;
      }
      return `http://res.pipeline/${url}`;
    },
  },
  async mounted() {
    this.templateList = await this.getTemplates();
  },
};
</script>

<style lang="less" scoped>
.page-content {
  height: 100%;
}

.template-list {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: center;
  margin: 10px 50px;
}

.template-item {
  display: flex;
  flex-direction: column;
  padding: 10px 20px 30px;

  &__thumbnail {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 196px;
    height: 196px;
    border: 1px solid#66666640;
  }

  &__img {
    max-width: 100%;
    max-height: 100%;
  }

  &__name {
    text-align: center;
    font-weight: bold;
  }

  &__button-group {
    display: flex;
    justify-content: center;
    margin: 2px 0 0;
  }

  &__button {
    background-color: #557e2a;
    color: #FFFFFF;
    border-radius: 3px;
    padding: 1px 8px;
    margin: 0 10px;
    cursor: pointer;
  }
}

.template-item--add {
  justify-content: center;
}
</style>
