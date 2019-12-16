<template>
  <el-container>
    <el-header>
    </el-header>
    <el-container>
      <!-- <el-aside width="200px">
      </el-aside> -->
      <el-container>
        <el-main>
          <el-table
            :data="tableData"
            stripe
            style="width: 100%">
            <el-table-column
              prop="id"
              label="页面ID">
            </el-table-column>
            <el-table-column
              prop="name"
              label="活动名称"/>
            <el-table-column
              prop="templateId"
              label="模板ID"/>
            <el-table-column
              label="状态">
              <template slot-scope="scope">
                {{scope.row.status == 1 ? "在线" : "待上线"}}
              </template>
            </el-table-column>
            <el-table-column
              width="300"
              label="操作">
              <template slot-scope="scope">
                <el-button @click="handleClick(scope.row)" type="text" size="small">查看</el-button>
                <el-button @click="publish(scope.row)" type="text" size="small">发布</el-button>
                <el-button @click="showRemoveDialog(scope.row)" type="text" size="small">删除</el-button>
                <el-button @click="showUploadDialog(scope.row)" type="text" size="small">更新模板</el-button>
                <el-button @click="showCopyDialog(scope.row)" type="text" size="small">复制模板</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-main>
      </el-container>
    </el-container>
    <el-dialog title="更新模板" :visible.sync="dialogFormVisible">
      <el-form :model="form">
        <el-form-item label="活动名称">
          <el-input v-model="form.name" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="活动id">
          <el-input v-model="form.id" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="模板文件">
          <el-button size="small"
                     @click="onChooseFile">
            <i class="el-icon-upload el-icon--right"></i>上传模板
          </el-button>
          <span class="template-file-name">{{uploadFileName}}</span>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="upload">确 定</el-button>
      </div>
    </el-dialog>
    <form id="fileForm" enctype="multipart/form-data">
      <input style="display:none;"
             ref="tplPageFileInput"
             type="file" name="file"
             @change="getFile"/>
    </form>
    <!--  复制活动页弹窗  -->
    <el-dialog
      title="输入活动名称"
      :visible.sync="copyDialogFormVisible"
      width="30%"
      center>
      <el-input v-model="pageName"></el-input>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="sureCopyPage">确 定</el-button>
      </span>
    </el-dialog>
  </el-container>

</template>

<script>
  import {APIS} from 'comp/util/constants';
  import fetch from 'comp/util/fetch';

  export default {
    components: {},
    data() {
      return {
        tableData: [],
        fromPage: {},
        url: "",
        pageName: "",
        form: {
          name: ""
        },
        uploadFileName: '',
        dialogFormVisible: false,
        copyDialogFormVisible: false,
        // removeDialogVisible: false,
        dataSchema: {},
      };
    },
    methods: {
      handleClick(item) {
        const {id} = item;
        this.$router.push(`pipeline?pageId=${id}`);
      },
      onChooseFile() {
        this.$refs.tplPageFileInput.click();
      },
      // 展示remove dialog
      showRemoveDialog(row) {
        console.log(row, "row");
         this.$alert('确定删除模板？', '删除模板', {
          confirmButtonText: '确定',
          callback: action => {
            if(action == 'confirm') {
              this.removePage(row);
            }
          }
        });
      },
      // 展示上传弹窗
      showUploadDialog(item) {
        console.log(item);
        this.form = Object.assign(this.form, item);
        this.dialogFormVisible = true;
      },
      // 展示复制当前模板弹窗
      showCopyDialog(item) {
        this.fromPage = item;
        this.copyDialogFormVisible = true;
      },
      // 确定拷贝新活动
      async sureCopyPage() {
        // 检测name
        if(this.pageName === '') {
          return this.$message.error('活动名称不能为空');
        }
        // 请求复制模板
        fetch(`${APIS.ROOT}/page/copypage`, {
          method: 'POST',
          body: {
            pageId: this.fromPage.id,
            pageName: this.pageName,
            templateId: this.fromPage.templateId,
          },
        }).then(() => {
          this.$message({
            message: '复制成功.',
            type: 'success',
          });
          this.getPageList();
        }).catch((res) => {
          this.$message({
            message: res.errMsg || '拷贝失败, 请检查网络.',
            type: 'warning',
          });
        }).finally(()=> {
          this.copyDialogFormVisible = false;
        });


      },
      getFile() {
        const file = event.target.files[0];
        const fileName = file.name;
        this.uploadFileName = fileName;

        if (!/zip$/.test(fileName)) {
          this.$refs.tplPageFileInput.value = null;
          return this.$message({
            message: '请选择模板压缩包(.zip).',
            type: 'warning',
          });
        }

        // 暂不支持上传超过 10M 的模板
        if (file.size > 1024 * 1024 * 10) {
          return this.$message({
            message: '暂不支持上传超过 10M 的模板.',
            type: 'warning',
          });
        }
        this.form.file = file;
      },
      // 更新模板
      upload() {
        const {id, file} = this.form;
        const formData = new FormData();
        formData.append('pageId', id);
        formData.append('file', file)
        fetch(`${APIS.ROOT}/page/upload`, {
          method: 'POST',
          headers: {
            'Content-Type': 'multipart/form-data',
          },
          body: formData,
        }).then(() => {
          this.$refs.tplPageFileInput.value = null;
          this.$message({
            message: '上传成功.',
            type: 'success',
          });
        }).catch((err) => {
          console.log(err);
          this.$message({
            message: '上传失败, 请检查网络.',
            type: 'warning',
          });
          this.$refs.tplPageFileInput.value = null;
        });
      },
      async publish(item) {
        const {id} = item;
        const ret = await fetch(`${APIS.PIPLINE}/publish`, {
          method: 'POST',
          body: {
            pageId: id + ""
          },
        });
        // console.log(ret);
        this.url = ret.pageId;
        this.$message(ret.pageId);
      },
      // remove page from mongodb
      async removePage(item) {
        const {id} = item;
        console.log("id", id);
        const ret = await fetch(`${APIS.PIPLINE}/remove`, {
          method: 'POST',
          body: {
            pageId: id + ""
          },
        });
        // console.log(ret);
        this.$message(`删除${ret.pageId}成功！`);
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
