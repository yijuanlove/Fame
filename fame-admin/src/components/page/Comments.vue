<template>
  <div>
    <el-table :data="commentDatas" border style="width: 100%">
      <el-table-column prop="id" label="id" width="60"></el-table-column>
      <el-table-column
        prop="name"
        label="称呼"
        show-overflow-tooltip
      ></el-table-column>
      <el-table-column
        prop="content"
        label="内容"
        show-overflow-tooltip
      ></el-table-column>
      <el-table-column
        prop="email"
        label="邮箱"
        show-overflow-tooltip
      ></el-table-column>
      <el-table-column
        prop="created"
        label="评论日期"
        show-overflow-tooltip
      ></el-table-column>
      <el-table-column label="操作" width="150">
        <template slot-scope="scope">
          <el-button size="small" @click="handleDetail(scope.row.id)"
            >详情
          </el-button>
          <el-button
            size="small"
            type="danger"
            @click="handleDelete(scope.row.id)"
            >删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog
      :visible.sync="detailVisible"
      :modal="false"
      :fullscreen="isMobile"
      :width="dialogWidth"
      center
      append-to-body
      custom-class="comment-dialog"
    >
      <el-row :gutter="10">
        <el-col :xs="24" :sm="12" :md="3">所属文章:</el-col>
        <el-col :xs="24" :sm="12" :md="9">{{ comment.title }}</el-col>
        <el-col :xs="24" :sm="12" :md="3">称呼:</el-col>
        <el-col :xs="24" :sm="12" :md="9">{{ comment.name }}</el-col>
      </el-row>
      <el-row :gutter="10">
        <el-col :xs="24" :sm="12" :md="3">邮箱:</el-col>
        <el-col :xs="24" :sm="12" :md="9">{{ comment.email }}</el-col>
        <el-col :xs="24" :sm="12" :md="3">网址:</el-col>
        <el-col :xs="24" :sm="12" :md="9">{{ comment.website }}</el-col>
      </el-row>
      <el-row :gutter="10" class="comment-row-detail">
        <el-col :span="24">
          <div class="markdown-body comment-replay" v-show="hasReplay">
            <div>
              <span class="comment-replay-name">{{ comment.replayName }}</span>
            </div>
            <div v-html="comment.replay"></div>
          </div>
          <div v-html="comment.content" class="markdown-body"></div>
        </el-col>
      </el-row>
      <el-row :gutter="10">
        <el-col :xs="24" :sm="12" :md="3">赞:</el-col>
        <el-col :xs="24" :sm="12" :md="9">{{ comment.agree }}</el-col>
        <el-col :xs="24" :sm="12" :md="3">踩:</el-col>
        <el-col :xs="24" :sm="12" :md="9">{{ comment.disagree }}</el-col>
      </el-row>
      <el-row :gutter="10">
        <el-col :xs="24" :sm="12" :md="3">ip:</el-col>
        <el-col :xs="24" :sm="12" :md="9">{{ comment.ip }}</el-col>
        <el-col :xs="24" :sm="12" :md="3">agent:</el-col>
        <el-col :xs="24" :sm="12" :md="9">{{ comment.agent }}</el-col>
      </el-row>
    </el-dialog>
    <div class="admin-page">
      <el-pagination
        layout="total,prev, pager, next"
        @current-change="init"
        :current-page.sync="currentPage"
        :page-size="pageSize"
        :total="total"
      >
      </el-pagination>
    </div>
  </div>
</template>

<script>
export default {
  data: function() {
    return {
      commentDatas: [],
      comment: {},
      total: 0,
      pageSize: 10,
      currentPage: 1,
      detailVisible: false,
      hasReplay: false,
      isMobile: false,
      dialogWidth: "60%"
    };
  },
  methods: {
    init() {
      this.$api.auth.getComments(this.currentPage).then(data => {
        this.commentDatas = data.data.list;
        this.total = data.data.total;
        this.pageSize = data.data.pageSize;
        for (let comment of this.commentDatas) {
          comment.created = this.$dayjs(comment.created).format(
            "YYYY-MM-DD HH:mm"
          );
        }
      });
    },
    initDetail(data) {
      this.comment = data;
      if (data.article) {
        this.comment.title = data.article.title;
      }
      if (data.pComment) {
        this.hasReplay = true;
        this.comment.replayName = data.pComment.name;
        this.comment.replay = data.pComment.content;
      } else {
        this.hasReplay = false;
      }
    },
    handleDetail(id) {
      this.$api.auth.getCommentDetail(id).then(data => {
        this.initDetail(data.data);
        this.detailVisible = true;
        if (document.body.clientWidth < 768) {
          this.isMobile = true;
          this.dialogWidth = "100%";
        } else {
          this.isMobile = false;
          this.dialogWidth = "60%";
        }
      });
    },
    handleDelete(id) {
      this.$confirm("此操作将永久删除该评论,是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "danger"
      })
        .then(() => {
          this.deleteComment(id);
        })
        .catch(() => {});
    },
    deleteComment(id) {
      this.$api.auth.deleteComment(id).then(() => {
        this.$message({
          type: "success",
          message: "删除成功!"
        });
        this.init();
      });
    }
  },
  mounted() {
    this.currentPage = Number(this.$route.query.page) || 1;
    this.init();
  }
};
</script>

<style scoped>
.el-table {
  border: 1px solid #e6ebf5;
}

.admin-page {
  margin-top: 30px;
  text-align: center;
}

.comment-dialog .comment-row-detail {
  width: 90%;
  margin: 15px auto !important;
  padding: 10px;
  border: 1px solid #eee;
  border-radius: 4px;
}

.comment-dialog .comment-row-detail .el-col {
  text-align: left !important;
}

.comment-dialog .comment-row-detail .comment-replay {
  padding: 10px;
  margin-bottom: 5px;
  border: 1px solid #eee;
  border-radius: 4px;
}

.comment-replay .comment-replay-name {
  text-decoration: underline;
  font-size: 16px;
}

@media screen and (min-width: 768px) {
  .comment-dialog .el-row .el-col:nth-child(odd) {
    text-align: right;
  }
}
</style>
