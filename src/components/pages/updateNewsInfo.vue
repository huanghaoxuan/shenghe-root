<template>
  <div align="center" style="margin-top: 100px">
    <Card style="width:550px">
      <Form
        ref="formDate"
        :model="formDate"
        :rules="ruleValidate"
        :label-width="80"
      >
        <FormItem label="标题：" prop="title">
          <Input
            prefix="ios-contact"
            placeholder="请输入标题"
            style="width: 400px"
            clearable
            v-model="formDate.title"
          />
        </FormItem>
        <FormItem label="内容：" prop="content">
          <Input
            v-model="formDate.content"
            type="textarea"
            :autosize="{ minRows: 2 }"
            style="width: 400px"
            clearable
            placeholder="请输入公告内容"
        /></FormItem>
      </Form>

      <Button
        type="primary"
        :disabled="ButtonStatus"
        @click="handleSubmit('formDate')"
        >提交</Button
      >
      <Button @click="handleReset('formDate')" style="margin-left: 50px"
        >重置为修改前内容</Button
      >
      <Button @click="handleReset2()" style="margin-left: 50px">清空</Button>
      <Button type="warning" to="/news" style="margin-left: 50px"
        >返回列表</Button
      >
    </Card>
  </div>
</template>
<script>
export default {
  data() {
    return {
      ButtonStatus: false,
      formDate: {
        ID: this.$store.state.newsInfo.id,
        gonghao: this.$store.state.gonghao,
        title: this.$store.state.newsInfo.title,
        content: this.$store.state.newsInfo.content
      },
      ruleValidate: {
        content: [
          {
            required: true,
            message: "请输入公告内容！",
            trigger: "blur"
          }
        ],
        title: [
          {
            required: true,
            message: "请输入文件标题！",
            trigger: "blur"
          }
        ]
      }
    };
  },
  methods: {
    handleSubmit(name) {
      this.$refs[name].validate(valid => {
        if (valid) {
          console.log(this.formDate);
          this.ButtonStatus = true;
          this.saveNewsInfoAjax(this.formDate);
          this.$Message.success("Success!");
        }
      });
    },
    handleReset(name) {
      this.$refs[name].resetFields();
    },
    handleReset2() {
      this.formDate.title = null;
      this.formDate.content = null;
    },
    saveNewsInfoAjax(data) {
      this.axios
        .post(
          "/newsInfo/update",
          this.qs.stringify({
            ID: data.ID,
            gonghao: data.gonghao,
            content: data.content,
            title: data.title
          }),
          {
            headers: {
              "Content-Type": "application/x-www-form-urlencoded"
            }
          }
        )
        .then(
          function(res) {
            console.log(res.data);
            this.$router.push("/news");
            this.$router.go(0);
            //控制台打印请求成功时返回的数据
            //bind(this)可以不用
          }.bind(this)
        )
        .catch(
          function(err) {
            if (err.response) {
              console.log(err.response);
              //控制台打印错误返回的内容
            }
            //bind(this)可以不用
          }.bind(this)
        );
    }
  }
};
</script>
