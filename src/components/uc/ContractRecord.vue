<style scoped>
.ivu-table td, .ivu-table th{
  height: 35px!important;
}
</style>
<template>
  <div class="nav-rights">
    <div class="nav-right">
      <div class="bill_flow_box">
        <div class="rightarea-con">
          <div class="form-group">
            <span>
              {{$t('uc.finance.record.start_end')}} ：
            </span>
            <DatePicker v-model="rangeDate" @on-change="changedate" format="yyyy-MM-dd" type="daterange" style="width: 200px;margin-right:30px;" @on-clear="clear"></DatePicker>
            <!--<DatePicker v-model="startDate" type="date"></DatePicker>-->
            <!--<span>-->
            <!--{{$t('uc.finance.record.to')}}-->
            <!--</span>-->
            <!--<DatePicker v-model="endDate" type="date"></DatePicker>-->
            <span>{{$t('uc.finance.record.symbol')}} ：</span>
            <Select v-model="coinType" style="width:100px;margin-right:30px;" @on-change="getAddrList" clearable :placeholder="$t('common.pleaseselect')">
              <Option v-for="item in coinList" :value="item.unit" :key="item.unit">{{ item.unit }}</Option>
            </Select>
            <span>
              {{$t('uc.finance.record.operatetype')}} ：
            </span>
            <Select v-model="recordValue" clearable style="width:200px" @on-change="getType" :placeholder="$t('common.pleaseselect')">
              <Option v-for="item in recordType" :value="item.value" :key="item.value">{{ item.label }}</Option>
            </Select>
            <Button type="warning" @click="queryOrder" style="padding: 6px 30px;margin-left:10px;background-color:#f0a70a;border-color:#f0a70a">{{$t('uc.finance.record.search')}}</Button>
          </div>
          <div class="order-table">
            <Table :no-data-text="$t('common.nodata')" :columns="tableColumnsRecord" :data="tableRecord" :disabled-hover="true" :loading="loading"></Table>
            <div style="margin: 10px;overflow: hidden">
              <div style="float: right;">
                <Page :total="total" :pageSize="pageSize" show-total :current="page" @on-change="changePage" id="record_pages"></Page>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  components: {},
  data() {
    return {
      loading: false,
      ordKeyword: "",
      rangeDate: "",
      startTime: "",
      endTime: "",
      recordValue: "",
      recordType: [
        {
          value: 0,
          label: this.$t("uc.finance.contractrecord.transferin")
        },
        {
          value: 1,
          label: this.$t("uc.finance.contractrecord.transferout")
        },
        {
          value: 2,
          label: this.$t("uc.finance.contractrecord.settlement")
        },
        {
          value: 3,
          label: this.$t("uc.finance.contractrecord.forcedping")
        }
      ],
      coinList: [],
      coinType: "",
      pageSize: 10,
      page: 1,
      total: 0,
      tableRecord: []
    };
  },
  created: function() {
    this.getList(this.page);
    this.getAddrList();
  },
  methods: {
    changedate() {
      if (this.rangeDate[0]) {
        this.startTime = this.dateform(this.rangeDate[0]);
        this.endTime = this.dateform(this.rangeDate[1]);
      }
    },
    changePage(pageindex) {
      this.page=pageindex;
      this.getList(this.page);
    },
    queryOrder() {
      if (this.rangeDate.length == 0) {
        this.$Message.error("uc.finance.contractrecord.datetip");
        return;
      } else {
        try {
          this.page=1;
          this.getList(this.page);
        } catch (ex) {
          this.$Message.error("uc.finance.contractrecord.datetip");
          return;
        }
      }
    },
    getAddrList() {
      //获取地址
      this.$http.post(this.host + "/uc/withdraw/support/coin/info").then(response => {
          var resp = response.body;
          if (resp.code == 0 && resp.data.length > 0) {
            this.coinList = resp.data;
            if (this.coinType) {
              this.coinType = this.coinType;
            }
          } else {
            this.$Message.error(resp.message);
          }
        });
    },
    getType() {
      // console.log(this.recordValue);
    },
    dateform(time) {
      var date = new Date(time);
      var y = date.getFullYear();
      var m = date.getMonth() + 1;
      m = m < 10 ? "0" + m : m;
      var d = date.getDate();
      d = d < 10 ? "0" + d : d;
      var h = date.getHours();
      h = h < 10 ? "0" + h : h;
      var minute = date.getMinutes();
      var second = date.getSeconds();
      minute = minute < 10 ? "0" + minute : minute;
      second = second < 10 ? "0" + second : second;
      return y + "-" + m + "-" + d + " " + h + ":" + minute + ":" + second;
    },
    getList(pageNo) {
      //获取tableWithdraw
      let memberId = 0;
      !this.$store.getters.isLogin && this.$router.push("/login");
      this.$store.getters.isLogin && (memberId = this.$store.getters.member.id);
      let startTime = "";
      let endTime = "";
      let symbol = "";
      let type = "";
      this.startTime && (startTime = this.startTime);
      this.endTime && (endTime = this.endTime);
      this.coinType && (symbol = this.coinType);
      if(this.recordValue == 0 || this.recordValue){
        type = this.recordValue;
      }
      // this.recordValue!="" || this.recordValue!=undefined && (type = this.recordValue);
      this.loading = true;
      let params = {
        pageNo: pageNo,
        pageSize: this.pageSize,
        startTime,
        endTime,
        memberId,
        symbol,
        type
      };
      this.$http.post(this.host + "/uc/asset/contract-transaction/all", params).then(response => {
          var resp = response.body;
          if (resp.code == 0) {
            this.loading = false;
            if (resp.data) {
              let trueData = resp.data;
              this.total = trueData.totalElements;
              this.tableRecord = trueData.content;
            }
          } else {
            this.$Message.error(resp.message);
          }
          this.loading = false;
        });
    },
    clear() {
      this.startTime = "";
      this.endTime = "";
    }
  },
  computed: {
    tableColumnsRecord() {
      let columns = [];
      var that = this;
      columns.push({
        title: this.$t("uc.finance.record.chargetime"),
        align: "center",
        width: 160,
        key:"createTime"
      });
      columns.push({
        title: this.$t("uc.finance.record.type"),
        render: function(h, params) {
          let str = "";
          let type = params.row.type;
          if (type == "IN") {
            str = that.$t("uc.finance.contractrecord.transferin");
          } else if (type == "OUT") {
            str = that.$t("uc.finance.contractrecord.transferout");
          } else if (type == "CONUT") {
            str = that.$t("uc.finance.contractrecord.settlement");
          }else {
            str = "uc.finance.contractrecord.forcedping";
          }
          return h("div", str, "");
        }
      });
      columns.push({
        title: this.$t("uc.finance.record.symbol"),
        align: "center",
        key:"symbol"
        // render: (h, param) => {
        //   return h("div", {}, param.row._source.symbol);
        // }
      });
      columns.push({
        // title: this.$t("uc.finance.record.num"),
        title: this.$t("uc.finance.record.num"), //到账数量
        align: "center",
        render(h, params) {
          return h(
            "span",
            {
              attrs: {
                title: params.row.amount
              }
            },
            that.toFloor(params.row.amount || "0")
          );
        }
      });
      // columns.push({
      //   title: this.$t("uc.finance.record.shouldfee"), //"应付手续费"
      //   align: "center",
      //   render(h, params) {
      //     return h(
      //       "span",
      //       {
      //         attrs: {
      //           title: params.row.fee
      //         }
      //       },
      //       that.toFloor(params.row.fee || "0")
      //     );
      //   }
      // });
      // columns.push({
      //   title: this.$t("uc.finance.record.discountfee"), //"抵扣手续费"
      //   align: "center",
      //   render(h, params) {
      //     return h(
      //       "span",
      //       {
      //         attrs: {
      //           title: params.row.discountFee
      //         }
      //       },
      //       that.toFloor(params.row.discountFee || "0")
      //     );
      //   }
      // });
      // columns.push({
      //   title: this.$t("uc.finance.record.realfee"), //"实际手续费"
      //   align: "center",
      //   render(h, params) {
      //     return h(
      //       "span",
      //       {
      //         attrs: {
      //           title: params.row.realFee
      //         }
      //       },
      //       that.toFloor(params.row.realFee || "0")
      //     );
      //   }
      // });
      columns.push({
        title: this.$t("uc.finance.record.status"),
        // key: "status",
        align: "center",
        render: (h, params) => {
          return h("div", that.$t("uc.finance.record.finish"), "");
        }
      });
      return columns;
    }
  }
};
</script>
<style scoped lang="scss">
.nav-rights {
  .nav-right {
    height: auto;
    overflow: hidden;
    padding: 0 15px;
    .bill_flow_box .rightarea-con {
      .form-group {
        margin-bottom: 20px;
        text-align: left;
      }
    }
  }
}
</style>
