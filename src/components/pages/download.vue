<template>
  <div>
    <Button type="primary" style="margin-top: 25px" to="/uploadFileInfo"
      >上传文件</Button
    >
    <div style="width: 80%;margin-top: 25px;margin-left: 10%">
      <Table stripe :columns="columns" :data="fileInfo"></Table>
      <Modal
        title="请确认是否进行删除"
        v-model="removeModal"
        class-name="vertical-center-modal"
        @on-ok="remove"
      >
        <p>
          删除后将
          <font color="#ed4014">无法恢复</font>
          ！
        </p>
        <p>删除后将无法查看该条信息！</p>
      </Modal>
      <div style="margin: 10px;overflow: hidden">
        <div style="float: right;">
          <Page
            :total="this.total"
            :page-size="10"
            @on-change="onChange"
            show-elevator
          ></Page>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      index: null,
      removeModal: false,
      showFileInfo: [],
      total: null,
      fileInfo: [],
      columns: [
        {
          title: "题目",
          key: "title"
        },
        {
          title: "上传时间",
          key: "time",
          width: 200
        },
        {
          title: "操作",
          key: "action",
          width: 150,
          align: "center",
          render: (h, params) => {
            return h("div", [
              h(
                "Button",
                {
                  props: {
                    type: "primary",
                    size: "small"
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.download(params.index);
                    }
                  }
                },
                "下载"
              ),
              h(
                "Button",
                {
                  props: {
                    type: "error",
                    size: "small"
                  },
                  on: {
                    click: () => {
                      this.index = params.index;
                      this.removeModal = true;
                    }
                  }
                },
                "删除"
              )
            ]);
          }
        }
      ]
    };
  },
  mounted() {
    this.findFileInfoListByPage(1);
    this.axios
      .get("/fileInfo/count", {
        params: {}
      })
      .then(
        function(res) {
          console.log(res.data);
          this.total = res.data;
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
  },
  methods: {
    onChange(event) {
      this.findFileInfoListByPage(event);
    },
    findFileInfoListByPage(page) {
      this.axios
        .get("/fileInfo/downloadList", {
          params: { page: page }
        })
        .then(
          function(res) {
            console.log(res.data);
            this.fileInfo = res.data;
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
    },
    download(index) {
      const msg = this.$Message.loading({
        content: "正在连接服务器",
        duration: 0
      });
      setTimeout(msg, 3000);
      this.axios
        .get("/fileInfo/download", {
          params: {
            ID: this.fileInfo[index].id
          },
          responseType: "blob"
        })
        .then(response => {
          const url = window.URL.createObjectURL(new Blob([response.data]));
          const link = document.createElement("a");
          link.href = url;
          link.setAttribute("download", this.fileInfo[index].fileInfoName);
          document.body.appendChild(link);
          link.click();
        })
        .catch(
          function(err) {
            if (err.response) {
              console.log(err.response);
              //控制台打印错误返回的内容
            }
            //bind(this)可以不用
          }.bind(this)
        );
    },
    remove() {
      if (this.$store.state.gonghao != this.fileInfo[this.index].gonghao) {
        this.$Message.error("非当前当前用户上传暂不允许删除");
      } else {
        this.axios
          .get("/fileInfo/remove", {
            params: {
              ID: this.fileInfo[this.index].id
            }
          })
          .then(
            function(res) {
              console.log(res.data);
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
  }
};
</script>
