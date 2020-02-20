<template>
  <div class="container" v-if="questionDetail">
    <h3>{{ questionDetail.caption }}</h3>

    <div class="content">{{ questionDetail.content }}</div>

    <div class="other_message">
      <div>作者：{{ questionDetail.author_name }}</div>

      <div class="">种类：{{ questionDetail.category_name }}</div>

      <div>创建时间：{{ questionDetail.create_time }}</div>

      <Button type="primary" @click="modal2 = true">查看评论</Button>

      <Button type="primary" @click="showFormaPage('answer')">回答问题</Button>
    </div>

    <hr />

    <div class="answer_list">
      <div class="answer_count">答案数量：{{ answerCount }}</div>
      <div
        class="answer_item"
        style="background:#eee;padding: 20px"
        v-for="answer of answerList"
        :key="answer.answer_id"
      >
        <Card :bordered="false">
          <div slot="title">
            <div
              style="display:flex;align-items:center;justify-content:space-between;"
            >
              <div>{{ answer.author_name }}</div>

              <div>{{ answer.create_time }}</div>
            </div>
          </div>
          <div>{{ answer.content }}</div>
          <div class="btns">
            <Button type="primary" style="margin-right:100px">查看评论</Button>

            <Button type="primary" @click="showFormaPage('comment')"
              >添加评论</Button
            >
          </div>
        </Card>
      </div>
    </div>

    <Modal
      v-model="modal1"
      :title="addComment ? '请输入你的评论' : '请输入你的答案'"
      @on-ok="ok"
      @on-cancel="cancel"
      okText="提交答案"
      cancelText="取消回答"
    >
      <Form :model="formItem" ref="answer" :label-width="80">
        <FormItem :label="addComment ? 'your comment' : 'your answer'">
          <Input
            v-model="formItem.value"
            type="textarea"
            :autosize="{ minRows: 2, maxRows: 5 }"
            placeholder="Enter something..."
          ></Input>
        </FormItem>
      </Form>
    </Modal>

    <Modal v-model="modal2" fullscreen title="Fullscreen Modal">
      <div>This is a fullscreen modal</div>
    </Modal>
  </div>
</template>

<script>
export default {
  props: ["question_id"],
  data() {
    return {
      questionDetail: null,
      answerList: [],
      modal1: false,
      formItem: {
        value: ""
      },
      answerCount: 0,
      modal2: false,
      addComment: false
    };
  },
  methods: {
    async getQuestionDetail() {
      const { data } = await this.$http.get(
        `/api/question/detail?question_id=${this.question_id}`
      );
      if (data.code == 0) {
        this.$Message.success("请求成功");
        this.questionDetail = data.data;
      } else {
        this.$Message.error("请求失败");
      }
    },
    async getAnswers() {
      const { data } = await this.$http.get("/api/answer/list", {
        params: { question_id: this.question_id }
      });
      if (data.code == 0) {
        this.answerList = data.data.answer_list;
        this.answerCount = data.data.total_count;
      } else {
        this.$Message.error("请求失败");
      }
    },
    async answerQuestion() {},
    async ok() {
      if (this.formItem.value) {
        const formData = {
          question_id: this.question_id,
          content: this.formItem.value
        };

        if (this.addComment) {
          const { data } = await this.$http.post(
            "/api/comment/post_comment",
            formData
          );
          if (data.code === 0) {
            this.$Message.success("评论成功");
          }
        } else {
          const { data } = await this.$http.post(
            "/api/answer/commit",
            formData
          );
          if (data.code === 0) {
            this.getAnswers();
          }
        }

        this.formItem.value = "";
      } else {
        this.$Message.error("答案不能为空");
      }
    },
    cancel() {},
    showFormaPage(type) {
      if (type === "answer") {
        this.addComment = false;
      } else {
        // 添加的是评论
        this.addComment = true;
      }
      this.modal1 = true;
    }
    // async getComments() {

    // }
  },
  created() {
    this.getQuestionDetail();
    this.getAnswers();
  }
};
</script>

<style lang="scss" scoped>
.container {
  padding: 15px;
  background: white;
}

.content {
  margin: 20px 0;
}
.answer_list {
  padding: 40px 0;
}
.other_message {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 30px;
}

.answer_count {
  font-weight: bold;
  margin: 0 0 30px 30px;
  font-size: 2em;
  color: red;
}

.btns {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  padding: 30rpx 0;
}
</style>
