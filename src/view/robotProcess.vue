<template>
  <div class="wraps">
    <!-- tab切换 -->
    <div class="tabBox">
      <Card>
        <div class="tabList">
          <p
            :class="cur == index ? 'cur' : ''"
            v-for="(item, index) in tabList"
            :key="index"
            @click="changeCur(index)"
          >
          <template v-if="hasPerm(item.perm)">
            {{ item.type }}
          </template>
          </p>
        </div>
      </Card>
    </div>
    <!-- 内容区 -->
    <div class="conBox">
      <!-- 控制台 -->
      <div class="conOne" v-show="cur == 0">
        <!-- 左边内容 -->
        <div class="leftCon">
          <Card
            :class="item.status == 2 ? 'curCard' : ''"
            v-for="(item, index) in dataList"
            :key="index"
          >
            <p slot="title">{{ item.label }}</p>
            <Poptip :content="item.describe" placement="right" slot="extra">
              <a>描述</a>
            </Poptip>
            <div class="imgCon" @click="showRightInfo(item, index)">
              <img src="@/assets/images/trians.png" class="trians" />
              <div class="nameText" v-if="item.status == 1">
                {{ item.name }}<span style="color: #1991dd">（未启动）</span>
              </div>
              <div class="nameText" v-if="item.status == 2">
                {{ item.name }}<span style="color: #f7b500">（进行中）</span>
              </div>
              <div class="nameText" v-if="item.status == 3">
                {{ item.name }}<span style="color: #f7b500">（暂停）</span>
              </div>
              <div class="nameText" v-if="item.status == 4">
                {{ item.name }}<span style="color: #1991dd">（结束）</span>
              </div>
              <div class="nameText" v-if="item.status == 5">
                {{ item.name }}<span style="color: #999">（异常停止）</span>
              </div>
            </div>
          </Card>
        </div>
        <!-- 右边图表 -->
        <div class="rigthCon">
          <div class="topCon">
            <Card style="width: 100%; display:block;" class="topConTitle">
              <div slot="title" style="display: flex; flex-direction: row;flex-wrap: wrap;">
                <p style="flex:1 0;">Hi,{{ rightData.name }}为您服务</p>
                <div v-if="hasPerm('control_search')"
                  style="
                    display: flex;
                    flex-direction: row;
                    justify-content: flex-end;
                  "
                >
                  <Form
                    :inline="true"
                    :label-width="80"
                    class="demo-form-inline"
                    ref="formInline"
                  >
                    <FormItem label="审核日期:" prop="checkBeginDate">
                      <div class="numCount" >
                        <Date-picker
                          placeholder="选择日期"
                          type="date"
                          style="width: 150px"
                          :options="disabledDate1"
                          :value="checkBeginDate"
                          format="yyyy-MM-dd"
                          @on-change="
                            handleDatepicker($event, 'checkBeginDate')
                          "
                        >
                        </Date-picker>
                        <span style="margin: 0 5px">—</span>
                        <Date-picker
                          placeholder="选择日期"
                          type="date"
                          style="width: 150px"
                          :options="disabledDate2"
                          :value="checkEndDate"
                          format="yyyy-MM-dd"
                          @on-change="handleDatepicker($event, 'checkEndDate')"
                        >
                        </Date-picker>
                      </div>
                    </FormItem>

                    <FormItem :label-width="0">
                      <el-button
                        type="primary"
                        @click="checkyear()"
                        size="small"
                      >
                        本年
                      </el-button>

                      <el-button
                        type="primary"
                        @click="checkmonth()"
                        size="small"
                        >本月</el-button
                      >

                      <el-button
                        type="primary"
                        @click="checktoday()"
                        size="small"
                        >本日</el-button
                      >
                    </FormItem>
                    <FormItem :label-width="0">
                      <el-button
                        type="primary"
                        @click="getCheckdate()"
                        icon="el-icon-search"
                        size="small"
                        >查询</el-button
                      >
                      <el-button
                        @click="handleReset()"
                        style="margin-left: 15px"
                        icon="el-icon-refresh-left"
                        size="small"
                        >重置</el-button
                      >
                    </FormItem>
                  </Form>
                </div>
                <div class="titBtn" v-if="hasPerm('control_operate')">
                  <el-button
                    type="primary"
                    size="small"
                    style="margin-right: 10px"
                    v-if="rightData.status == 1"
                    @click="handleChange(2, rightData.sceneId)"
                    >启动</el-button
                  >
                  <el-button
                    type="warning"
                    size="small"
                    style="margin-right: 10px"
                    v-if="rightData.status == 2"
                    @click="handleChange(3, rightData.sceneId)"
                    >暂停</el-button
                  >
                  <el-button
                    type="success"
                    size="small"
                    style="margin-right: 10px"
                    v-if="rightData.status == 3"
                    @click="handleChange(2, rightData.sceneId)"
                    >继续</el-button
                  >
                  <el-button
                    type="info"
                    size="small"
                    style="margin-right: 10px"
                    v-if="rightData.status == 3"
                    @click="handleChange(4, rightData.sceneId)"
                    >结束</el-button
                  >
                  <el-button
                    type="info"
                    size="small"
                    style="margin-right: 10px"
                    v-if="rightData.status == 5"
                    >需人工处理</el-button
                  >
                  <el-button
                    type="primary"
                    size="small"
                    style="margin-right: 10px"
                    v-if="rightData.status == 4"
                    @click="handleReboot(rightData.sceneId)"
                    >重启</el-button
                  >
                </div>
              </div>
              <!--slot="extra"
                  justify-content: space-between; -->
              <div class="chartOne">
                <div id="countChart" ref="countChart" style="width: 100%; height: 260px"></div>
              </div>
            </Card>
          </div>
          <div class="downCon">
            <Card style="width: 49%">
              <div id="myChartTwo" ref="myChartTwo" style="width: 100%; height: 500px"></div>
            </Card>
            <Card style="width: 49%">
              <div id="myChartThr" ref="myChartThr" style="width: 100%; height: 500px"></div>
            </Card>
          </div>
        </div>
      </div>
      <!-- 信息管理 -->
      <div class="conTwo" v-if="cur == 1">
        <InfoManage></InfoManage>
      </div>
      <!-- 操作日志 -->
      <div class="conThr" v-if="cur == 2">
        <OperationLog></OperationLog>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "@/libs/api.request";
import * as echarts from "echarts";
import { localSave, localRead } from "@/libs/util";
import { on, off } from '@/libs/tools';
import {
  homelist, // 列表
  changeStatus, // 改变状态
  reboot,
} from "@/api/user";
import InfoManage from "./components/InfoManage";
import OperationLog from "./components/OperationLog";
export default {
  components: {
    InfoManage,
    OperationLog,
  },
  data() {
    return {
      countChart: null,
      myChartTwo: null,
      myChartThr: null,
      secneName: "",
      id: "",
      dataList: [],
      sort: [],
      rightData: {},
      leftData: localRead("leftPie") ? JSON.parse(localRead("leftPie")) : [],
      rightItem: localRead("rightPie") ? JSON.parse(localRead("rightPie")) : [],
      chartsDate: localRead("countChart")
        ? JSON.parse(localRead("countChart"))
        : {},
      robotId: null,
      cur: 0,
      cardIdx: 0,
      timer: null, // 定时器
      tabList: [
        {
          type: "控制台",
          perm: 'control_search'
        },
        {
          type: "信息管理",
          perm: 'info_search'
        },
        {
          type: "操作日志",
          perm: 'log_search'
        },
      ],
      checkBeginDate: "",
      checkEndDate: "",
      disabledDate1: {
        disabledDate(date) {
          return date && date.valueOf() > new Date();
        },
      },
      disabledDate2: {
        disabledDate(date) {
          return date && date.valueOf() > new Date();
        },
      },
    };
  },
  computed:{
    topCollapse(){
      return this.$store.state.app.mainSideBarCollapse;
    }
  },
  watch: {
    topCollapse(v, oldv){
      const _this = this;
      setTimeout(() => {
        _this.resize();
      }, 300);
    }
  },
  beforeDestroy() {
    clearInterval(this.timer);
    this.timer = null;
  },
  mounted() {
    this.getCharts();
    this.query();
    this.timer = setInterval(() => {
      this.query();
    }, 1800000);
  },
  beforeDestroy () {
    off(window, 'resize', this.resize)
  },
  methods: {
    resize() {
      this.countChart.resize()
      this.myChartTwo.resize()
      this.myChartThr.resize()
    },
    handleDatepicker(dateValue, dataKey) {
      this.$set(this, dataKey, dateValue);
      this.status = "";
      const bb = new Date();
      bb.setFullYear(bb.getFullYear() - 1);
      let lastyear = bb;
      let today = new Date();

      let begindate = new Date(this.checkBeginDate);
      begindate.setDate(begindate.getDate()-1);
      let enddate = new Date(this.checkEndDate);
      this.disabledDate1 = {
        disabledDate(date) {
          return (
            (date && date.valueOf() < lastyear) ||
            (date && date.valueOf() > enddate)
          );
        },
      };
      this.disabledDate2 = {
        disabledDate(date) {
          return (
            (date && date.valueOf() < begindate) ||
            (date && date.valueOf() > today)
          );
        },
      };
    },
    getCheckdate() {
      const _this = this;
      _this.getChartsData(_this.robotId);
      _this.getData([], _this.robotId);
    },
    handleReset(){
      this.checkBeginDate = "";
      this.checkEndDate = "";
    },
    checkyear() {
      var now_year = new Date().getFullYear();
      let begindate = new Date(now_year, 0, 1);
      this.checkBeginDate=this.formatDate(begindate);
      this.checkEndDate=this.formatDate(new Date());
    },
    checkmonth() {
      var date=new Date();
      date.setDate(1);
      this.checkBeginDate=this.formatDate(date);

      this.checkEndDate=this.formatDate(new Date());
    },
    checktoday() {
      var now = new Date()

      this.checkBeginDate=this.formatDate(now);
      this.checkEndDate=this.formatDate(now);
    },
    formatDate(date) {
      var d = new Date(date),
        month = "" + (d.getMonth() + 1),
        day = "" + d.getDate(),
        year = d.getFullYear();

      if (month.length < 2) month = "0" + month;
      if (day.length < 2) day = "0" + day;

      return [year, month, day].join("-");
    },
    showRightInfo(item, index) {
      this.cardIdx = index;
      this.rightData = item;
      this.robotId = item.id;
      this.getData(item, item.id);
      this.getChartsData(item.id);
    },
    getChartTwo() {
      this.myChartTwo = echarts.init(this.$refs.myChartTwo);
      this.myChartTwo.clear();
      this.myChartTwo.setOption({
        grid: {
          top: 0,
          bottom: 0,
          left: 0,
          right: 0,
          width: "auto",
          height: "auto",
        },
        title: {
          text: "统计结果",
          left: "left",
        },
        tooltip: {
          trigger: "item",
        },
        color: ["#70B822", "#1991DD", "#F7B500"],
        legend: {
          icon: "circle",
          orient: "vertical",
          bottom: "bottom",
          right: "right",
        },
        series: [
          {
            name: "统计结果",
            type: "pie",
            radius: "50%",
            data: [...this.leftData],
            emphasis: {
              itemStyle: {
                shadowBlur: 10,
                shadowOffsetX: 0,
                shadowColor: "rgba(0, 0, 0, 0.5)",
              },
            },
            label: {
              normal: {
                show: true,
                // position: "inner", //标签的位置
                textStyle: {
                  fontWeight: 300,
                  fontSize: 16, //文字的字体大小
                },
                formatter: "{d}%",
                // color: "#ffffff",
              },
            },
          },
        ],
      });
      this.myChartTwo.resize();
      on(window, 'resize', this.resize)
    },
    getChartThr() {
      this.myChartThr = echarts.init(this.$refs.myChartThr);
      this.myChartThr.clear();
      this.myChartThr.setOption({
        grid: {
          top: 0,
          bottom: 0,
          left: 0,
          right: 0,
          width: "auto",
          height: "auto",
        },
        title: {
          text: "人机协同统计",
          left: "left",
        },
        tooltip: {
          trigger: "item",
        },
        color: ["#70B822", "#9C97D3", "#1991DD", "#F7B500"],
        legend: {
          icon: "circle",
          orient: "vertical",
          bottom: "bottom",
          right: "right",
          selectedMode: false,
        },
        series: [
          {
            name: "人机协同统计",
            type: "pie",
            radius: ["35%", "50%"],
            avoidLabelOverlap: false,
            label: {
              show: true,
              position: "outer",
              formatter: "{d}%\n\n",
              borderWidth: 20,
              borderRadius: 4,
              padding: [50, -50],
            },
            data: [...this.rightItem],
            labelLine: {
              show: true,
              length: 30,
              length2: 50,
            },
          },
        ],
      });
      this.myChartThr.resize();
      on(window, 'resize', this.resize)
    },
    changeCur(index) {
      this.cur = index;
    },
    query() {
      const _this = this;
      homelist({ secneName: _this.secneName, id: _this.id }).then((res) => {
        if (res.data.code == 0) {
          _this.dataList = res.data.data;
          _this.robotId = res.data.data[0].id;
          _this.rightData = res.data.data[0];
          _this.getChartsData(res.data.data[0].id);
          _this.getData(res.data.data[0], res.data.data[0].id);
        }
      });
    },
    /**
     * 单据量统计 - 右 > 顶部百分比
     * 结果统计 - 右 > 下左:饼图
     */
    getChartsData(id) {
      let _this = this;
      axios
        .request({
          method: "post",
          url: `/api/bill/robot`,
          data: {
            robotId: id,
            checkBeginDate: _this.checkBeginDate,
            checkEndDate: _this.checkEndDate,
          },
        })
        .then((resp) => {
          let data = resp.data;
          if (data.code === 20000) {
            // _this.chartsDate = data.data;
            const sum = Object.values(data.data).reduce(
              (acc, val) => acc + val
            );
            const percent = {};
            Object.keys(data.data).map((key) => {
              percent[`${key}P`] = ((data.data[key] / sum) * 100).toFixed(2);
            });
            percent.completedP = (
              ((data.data.success + data.data.fail) / sum) *
              100
            ).toFixed(2);
            _this.chartsDate = Object.assign(data.data, percent);
            localSave("countChart", JSON.stringify(_this.chartsDate));
            _this.renderCountChart();
            _this.getLeftData(data.data);
          } else {
            _this.$Notice.warning({
              title: "温馨提示",
              desc: data.message,
            });
          }
        })
        .catch((err) => {
          console.log(err);
        });
    },
    renderCountChart() {
      const _this = this;
      this.countChart = echarts.init(this.$refs.countChart);
      this.countChart.clear();
      this.countChart.setOption({
        grid: {
          top: 0,
          bottom: 0,
          left: 0,
          right: 0,
        },
        title: {
          text: "单据量统计",
          left: "left",
        },
        tooltip: {
          trigger: "item",
        },
        color: ["#70B822", "#1991DD", "#F7B500"],
        legend: {
          icon: "circle",
          orient: "vertical",
          bottom: "bottom",
          right: "right",
          selectedMode: false,
          borderRadius: 50,
        },
        xAxis: {
          show: false,
          type: "value",
          boundaryGap: [0, 0],
        },
        yAxis: [
          {
            type: "category",
            data: [""],
            axisLine: { show: false },
            axisTick: [
              {
                show: false,
              },
            ],
          },
        ],
        series: [
          {
            name: "",
            type: "bar",
            name: `审核完成单据（${
              _this.chartsDate.success + _this.chartsDate.fail
            }单）`,
            stack: "总量",
            // barMaxWidth: 20,
            barWidth: 35,
            label: {
              show: _this.chartsDate.success + _this.chartsDate.fail > 0,
              position:
                Number(_this.chartsDate.completedP) > 5 ? "inside" : "top",
              offset: [20, 0],
              formatter: "{c}%",
            },
            itemStyle: {
              barBorderRadius: [0, 0, 0, 0],
            },
            data: [_this.chartsDate.completedP],
            tooltip: {
              formatter: "{a}<br>{c}%",
            },
          },
          {
            name: "",
            type: "bar",
            name: `审核超时单据（${_this.chartsDate.timeout}单）`,
            stack: "总量",
            // barMaxWidth: 20,
            barWidth: 35,
            label: {
              show: _this.chartsDate.timeout > 0,
              position:
                Number(_this.chartsDate.timeoutP) > 5 ? "inside" : "top",
              offset: [-15, 0],
              formatter: "{c}%",
            },
            itemStyle: {
              barBorderRadius: [0, 0, 0, 0],
            },
            data: [_this.chartsDate.timeoutP],
            tooltip: {
              formatter: "{a}<br>{c}%",
            },
          },
          {
            name: "",
            type: "bar",
            name: `审核未完成单据（${_this.chartsDate.checking}单）`,
            stack: "总量",
            // barMaxWidth: 20,
            barWidth: 35,
            label: {
              show: _this.chartsDate.checking > 0,
              position:
                Number(_this.chartsDate.checkingP) > 5 ? "inside" : "bottom",
              offset: [-15, 0],
              formatter: "{c}%",
            },
            itemStyle: {
              barBorderRadius: [0, 0, 0, 0],
            },
            data: [_this.chartsDate.checkingP],
            tooltip: {
              formatter: "{a}<br>{c}%",
            },
          },
        ],
      });
      this.countChart.resize();
      on(window, 'resize', this.resize)
    },
    getLeftData(data) {
      let _this = this;
      _this.leftData = [];
      let obj1 = {};
      obj1.value = data.success;
      obj1.name = "通过";
      let obj2 = {};
      obj2.value = data.fail;
      obj2.name = "不通过";
      let obj6 = {};
      obj6.value = data.timeout;
      obj6.name = "超时";
      _this.leftData.push(obj1, obj2, obj6);
      localSave("leftPie", JSON.stringify(_this.leftData));
      _this.getChartTwo();
    },
    /**
     * 人机协同统计 - 右 > 下右:圆环
     */
    getData(data, id) {
      const request = {
        robotId: id,
        checkBeginDate: this.checkBeginDate,
        checkEndDate: this.checkEndDate,
      };
      let _this = this;
      axios
        .request({
          method: "post",
          url: `/api/bill/billstatuscount`,
          data: request,
        })
        .then((resp) => {
          let data = resp.data;
          if (data.code === 20000) {
            const parseData = [
              { name: "通过", value: data.data.adopt },
              { name: "驳回", value: data.data.reject },
              { name: "转人工", value: data.data.artificial },
              { name: "超时", value: data.data.timeout },
            ];
            _this.rightItem = parseData;
            localSave("rightPie", JSON.stringify(parseData));
            _this.getChartThr();
          } else {
            _this.$Notice.warning({
              title: "温馨提示",
              desc: data.message,
            });
          }
        })
        .catch((err) => {
          console.log(err);
        });
    },
    handleChange(status, sceneId) {
      let _this = this;
      changeStatus({ status, sceneId }).then((res) => {
        if (res.data.code == 0) {
          _this.$message({
            message: res.data.msg,
            type: "success",
            duration: 1500,
          });
          _this.query();
        } else {
          _this.$notify({
            type: "warning",
            title: "失败",
            message: res.data.msg,
            duration: 1500,
          });
        }
      });
    },
    handleReboot(sceneId) {
      let _this = this;
      reboot({ sceneId }).then((res) => {
        if (res.data.code == 0) {
          _this.$message({
            message: res.data.msg,
            type: "success",
            duration: 1500,
          });
          _this.query();
        } else {
          _this.$notify({
            type: "warning",
            message: res.data.msg,
            duration: 1500,
          });
        }
      });
    },
    getCharts() {
      this.getChartTwo();
      this.getChartThr();
    },
  },
};
</script>
<style rel="stylesheet/scss" lang="less" scoped>
.tabBox {
  width: 100%;
  height: 66px;
  margin-bottom: 20px;
}
.tabList {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  border-bottom: 1px solid #999;
  p {
    height: 30px;
    margin-right: 20px;
    cursor: pointer;
  }
}
.cur {
  color: #1991dd;
  border-bottom: 2px solid #1991dd;
}
.curCard {
  box-shadow: 0 0 0 2px rgba(6, 116, 243, 0.856);
}
.logProcess {
  width: 100%;
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  display: none;
}
.conBox {
  width: 100%;
  height: 100%;
}
.conOne {
  width: 100%;
  display: flex;
  justify-content: space-between;
  .leftCon {
    width: 18%;
    cursor: pointer;
  }
  .rigthCon {
    width: 81%;
    display: flex;
    flex-direction: column;
    .topCon {
      width: 100%;
    }
  }
}
/deep/.ivu-card-head {
  display: flex;
}
/deep/.topConTitle .ivu-card-head {
    display: block;
}
.titBtn {
  // display: flex;
}
.downCon {
  width: 100%;
  display: flex;
  justify-content: space-between;
}
.chartOne {
  width: 100%;
  display: flex;
  flex-direction: column;
  color: #333;
  font-weight: 500;
  > p {
    width: 100%;
    text-align: left;
  }
  .lineBox {
    width: 100%;
    height: 35px;
    margin: 22px 0 63px 0;
    box-sizing: border-box;
    display: flex;
    li {
      list-style: none;
      height: 100%;
      text-align: center;
      line-height: 35px;
      color: #fff;
      font-size: 24px;
    }
    li:nth-child(1) {
      background: #1991dd;
    }
    li:nth-child(2) {
      background: #f7b500;
    }
    li:nth-child(3) {
      background: #70b822;
    }
  }
  .colorBox {
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: flex-end;
    .itemCor {
      display: flex;
      align-items: center;
      margin: 0 10px;
      .corBlock {
        width: 14px;
        height: 14px;
        border-radius: 50%;
        margin-right: 10px;
      }
    }
    .itemCor:nth-child(1) .corBlock {
      background: #1991dd;
    }
    .itemCor:nth-child(2) .corBlock {
      background: #f7b500;
    }
    .itemCor:nth-child(3) .corBlock {
      background: #70b822;
    }
  }
}

.imgCon {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  color: #333;
  .trians {
    width: 63px;
    height: 76px;
    margin-bottom: 20px;
  }
}
.ivu-card {
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
}
.top {
  width: 100%;
  display: flex;
  .trian {
    width: 20px;
    height: 24px;
  }
  span {
    color: #333;
    font-weight: 600;
    margin-left: 10px;
  }
}
.statusBox {
  display: flex;
  align-items: center;
  margin: 15px 0;
  .status,
  .service {
    color: #333;
    font-weight: 600;
    .result {
      color: #999;
    }
  }
  .service {
    margin-left: 8%;
  }
}
.situation {
  > p {
    width: 100%;
    color: #333;
    font-weight: 600;
  }
  .list {
    display: flex;
    align-items: center;
    .left {
      width: 65%;
      .itemCon {
        display: flex;
        align-items: center;
        line-height: 2.3;
        > p {
          width: 20%;
        }
        .el-progress {
          flex: 1;
        }
        /deep/ .el-progress-bar {
          width: 95%;
        }
      }
    }
    .right {
      flex: 1;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      > p {
        margin-bottom: 10px;
      }
    }
  }
}
// /deep/.ivu-input {
//   width: 150px;
// }
// /deep/.ivu-form-item{
//   margin-bottom: 1px;
// }
</style>
