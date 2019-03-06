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
      </Form>
      <Upload
        ref="upload"
        type="drag"
        :data="formDate"
        action="/api/fileInfo/upload"
        :before-upload="handleUpload"
        :on-success="handleSuccess"
      >
        <div style="padding: 20px 0">
          <Icon type="ios-cloud-upload" size="52" style="color: #3399ff"></Icon>
          <p>点击或拖到此处进行上传</p>
        </div>
      </Upload>
      <div v-if="file !== null">{{ file.name }}</div>
      <Button
        type="primary"
        :disabled="ButtonStatus"
        @click="handleSubmit('formDate')"
        >提交</Button
      >
      <Button @click="handleReset('formDate')" style="margin-left: 100px"
        >清空</Button
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
        gonghao: this.$store.state.gonghao,
        title: null
      },
      ruleValidate: {
        title: [
          {
            required: true,
            message: "请输入文件标题！",
            trigger: "blur"
          }
        ]
      },
      file: null
    };
  },
  activated: function() {
    this.getCase();
  },
  methods: {
    handleUpload(file) {
      this.file = file;
      return false;
    },
    handleSubmit(name) {
      this.$refs[name].validate(valid => {
        if (valid) {
          console.log(this.formDate);
          this.upload();

          //this.$Message.success("Success!");
        }
      });
    },
    handleReset(name) {
      this.$refs[name].resetFields();
      this.file = null;
    },
    upload() {
      this.ButtonStatus = true;
      this.$refs.upload.post(this.file);
    },
    handleSuccess() {
      this.$Message.success("Success!");
      this.$router.push("/download");
      this.$router.go(0);
    }
  }
};
</script>
