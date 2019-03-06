<template>
  <div>
    <div style="width: 80%;margin-top: 25px;margin-left: 10%">
      <Table :columns="columns" :data="status"></Table>
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
import expandRow from "./table-expand.vue";
export default {
  data() {
    return {
      total: null,
      index: null,
      year: new Date().getFullYear(),
      status: [],
      columns: [
        {
          type: "expand",
          width: 50,
          render: (h, params) => {
            return h(expandRow, {
              props: {
                row: params.row
              }
            });
          }
        },
        {
          title: "姓名",
          key: "xingming"
        },
        {
          title: "状态",
          key: "status"
        },
        {
          title: "学年",
          key: "year"
        },
        {
          title: "操作",
          key: "action",
          width: 300,
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
                "下载报表"
              ),
              h(
                "Button",
                {
                  props: {
                    type: "error",
                    size: "small"
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.auditing(params.index, "通过");
                    }
                  }
                },
                "通过审核"
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
                      this.auditing(params.index, "不通过");
                    }
                  }
                },
                "不通过审核"
              )
            ]);
          }
        }
      ]
    };
  },
  mounted() {
    this.findStatusByPage(1);
    this.axios
      .get("/auditing/findStatusByPageBySort/count", {
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
    auditing(index, status) {
      if (this.year != this.status[index].year) {
        this.$Message.error("非当前年份暂不允许审核");
      } else {
        this.axios
          .get("/auditing/status", {
            params: {
              ID: this.status[index].id,
              status: status
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
    },
    download(index) {
      {
        this.axios
          .get("/declare/download", {
            params: {
              gonghao: this.status[index].gonghao,
              xuekeName: this.status[index].xuekeName,
              zhicheng: this.status[index].zhicheng
            },
            responseType: "blob"
          })
          .then(response => {
            const url = window.URL.createObjectURL(new Blob([response.data]));
            const link = document.createElement("a");
            link.href = url;
            link.setAttribute(
              "download",
              this.status[index].gonghao +
                "-" +
                this.status[index].xuekeName +
                "-" +
                this.status[index].zhicheng +
                ".xlsx"
            );
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
      }
    },
    onChange(event) {
      this.findStatusByPage(event);
    },
    findStatusByPage(page) {
      this.axios
        .get("/auditing/findStatusByPageBySort", {
          params: { page: page }
        })
        .then(
          function(res) {
            console.log(res.data);
            this.status = res.data;

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
