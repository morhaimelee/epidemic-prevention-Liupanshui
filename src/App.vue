<template>
  <div id="app">
    <div id="all-window">
      <div id="nav">
        <div class="location">
          <i></i>六盘水市
        </div>
        <div class="calendar">
          <div class="time">{{nowTime}}</div>
          <div class="dd-date">
            <div class="week">{{nowWeek}}</div>
            <div class="date">{{nowDate}}</div>
          </div>
        </div>
      </div>
      <!-- 头部结束 -->
      <div id="mid_three">
        <div class="mid-box" id="mid_1">
          <div class="mid-text">累计发热人数</div>
          <div class="mid-num-box">{{dataList.grandTotal.heat_person}}</div>
        </div>
        <div class="mid-box" id="mid_2">
          <div class="mid-text">累计重点患者</div>
          <div class="mid-num-box">{{dataList.grandTotal.sickKey_person}}</div>
        </div>
        <div class="mid-box" id="mid_3">
          <div class="mid-text">累计疑似人数</div>
          <div class="mid-num-box">{{dataList.grandTotal.suspect_person}}</div>
        </div>
      </div>
      <!-- 人数展示结束 -->
      <div class="title-text">
        <i></i>六盘水数据
      </div>
      <div id="deadline">截止{{dataList.deadline.deadline_date}} {{dataList.deadline.deadline_time}} 全市数据统计</div>
      <div class="guiyang_data">
        <ol class="case-text">
          <li>
            确诊病例
            <span>较昨日人数：{{dataList.openData.openCompare[0]}}人</span>
          </li>
          <li>
            治愈案例
            <span>较昨日人数：{{dataList.openData.openCompare[1]}}人</span>
          </li>
          <li>
            死亡案例
            <span>较昨日人数：{{dataList.openData.openCompare[2]}}人</span>
          </li>
        </ol>
        <ul class="case-bar">
          <li>
            <i v-bind:style="{width: widthAbs(dataList.openData.openPatients[0])}"></i>
            <span>{{dataList.openData.openPatients[0]}}</span>
            <s>人</s>
          </li>
          <li>
            <i v-bind:style="{width: widthAbs(dataList.openData.openPatients[1])}"></i>
            <span>{{dataList.openData.openPatients[1]}}</span>
            <s>人</s>
          </li>
          <li>
            <i v-bind:style="{width: widthAbs(dataList.openData.openPatients[2])}"></i>
            <span>{{dataList.openData.openPatients[2]}}</span>
            <s>人</s>
          </li>
        </ul>
      </div>
      <!-- 六盘水公开数据结束 -->
      <div class="title-text" id="drug">
        <i></i>药品，物资消耗折线图
      </div>
      <div class="durg-data">
        <div id="myCharts" ref="myCharts"></div>
      </div>
      <!-- 药物结束 -->
      <div class="title-text" id="epidemic_warning">
        疫情预警管理
        <i></i>
      </div>
      <div class="epidemic-data">
        <div class="sample">
          <div>接受样本数量</div>
          <div>{{dataList.sampleData.accept}}</div>
          <div>检测样本数量</div>
          <div>{{dataList.sampleData.detection}}</div>
        </div>
        <div class="res-i positive">
          阳性人数
          <i>{{dataList.sampleData.positive}}</i>
        </div>
        <div class="res-i pos-2">
          待出结果
          <i class="i-2">{{dataList.sampleData.stay_out}}</i>
        </div>
        <div class="res-i negative">
          阴性人数
          <i>{{dataList.sampleData.negtive}}</i>
        </div>
        <div class="res-i neg-2">
          其他流感
          <i class="i-2">{{dataList.sampleData.others}}</i>
        </div>
      </div>
      <!-- 样本结束 -->
      <div class="point-patient">
        <div class="patient-title">最新重点患者</div>
        <div class="patient-list">
          <ul>
            <li v-for="item in dataList.keyPatient">
              <span>{{item.name}}</span>
              <span class="sex">{{item.sex}}</span>
              <span class="age">{{item.age}}</span>
              <span class="phone-num">{{item.phone_num}}</span>
              <span class="date-sicken">{{item.confirmed_date}}</span>
            </li>
          </ul>
        </div>
      </div>
      <!-- 重点患者结束 -->
      <div class="title-text" id="day_of_person">
        <i></i>每日疫情人数统计
      </div>
      <div class="statistics-allDay">
        <div id="myCharts2" ref="myCharts2"></div>
      </div>
      <!-- 底部每日统计结束 -->
      <div class="map-screen" :id="id"></div>
    </div>

    <!-- <router-view/> -->
  </div>
</template>

<script>
import echarts from "echarts";
import JSON from "./assets/520200";
import axios from "axios";
document.title = "六盘水疫情防控指挥调度平台"

export default {
  name: "liupanshuiyiqing",
  props: {},
  data() {
    return {
      openWidth1: "77%",
      myCharts: "",
      dataList: {
        grandTotal: {},
        openData: {
          openPatients: [],
          openCompare: []
        },
        sampleData: {},
        keyPatient: [],
        medicinal: {
          medicine_date: [],
          medicine_dataNum: [],
          supplies_dataNum: []
        },
        deadline: {
          deadline_date: "",
          deadline_time: ""
        }
      },
      nowTime: "", //当前时间
      nowWeek: "", //当前星期
      nowDate: "", //当前日期
      id: "echarts_" + new Date().getTime() + Math.floor(Math.random() * 1000),
      echartObj: null,
      option: {
        title: {
          // 设置标题
          text: "",
          top: "3%",
          left: "5%",
          textStyle: {
            fontSize: 18,
            fontWeight: 300,
            color: "#b6d7ff"
          }
        },
        tooltip: {
          formatter: params => {
            return (
              `<div class="map_tooltip">` +
              params.name +
              "</br>重点患者&nbsp;" +
              params.value +
              `</div>`
            );
            console.log(this.params.value);
          },
          textStyle: { color: "#fff", fontSize: "16" }
        },
        legend: {
          orient: "vertical",
          top: "9%",
          left: "5%",
          icon: "circle",
          data: [],
          selectedMode: "single",
          selected: {},
          itemWidth: 12,
          itemHeight: 12,
          itemGap: 30,
          inactiveColor: "#b6d7ff",
          textStyle: {
            color: "#ec808d",
            fontSize: 14,
            fontWeight: 300,
            padding: [0, 0, 0, 15]
          }
        },
        visualMap: {
          // 设置地图范围值显示的颜色
          min: 0,
          max: 20,
          right: '130',
          bottom: '70',
          text: ['20','0'],
          // show: false,
          // splitNumber: 5,
          inRange: {
            color: [
              "rgb(288,191,123)",
              "rgb(11,255,254)",
              "rgb(9,168,255)"
            ].reverse()
          },
          textStyle: {
            color: "#fff"
          }
        },
        geo: {
          // 设置地图的显示信息
          map: "六盘水", // 注意  哪个区域的就显示哪个区域的名称
          label: {
            normal: {
              // 设置字体相关信息
              show: true,
              color: "#fff",
              fontSize: "16"
            },
            emphasis: {
              // 设置鼠标移上去hover效果
              show: true,
              color: "#000"
            }
          },
          roam: false,
          itemStyle: {
            // 设置地图块的相关显示信息
            normal: {
              areaColor: "rgb(9,192,255)",
              borderColor: "#6367ad",
              borderWidth: 1
            },
            emphasis: {
              areaColor: "#3093d8" // hover效果
            }
          },
          left: "5%",
          right: "5%",
          top: "5%",
          bottom: "5%"
        },
        series: [
          {
            // 地图块的相关信息
            name: "疫情监控",
            type: "map",
            geoIndex: 0, // 不可缺少，否则无tooltip 指示效果
            zoom: 1,
            label: {
              normal: {
                show: true,
                textStyle: {
                  color: "#fff",
                  fontSize: "16"
                }
              },
              emphasis: {
                show: true,
                textStyle: { color: "#fff", fontSize: "16" }
              }
            },
            roam: false,
            data: []
          }
        ]
      }
      //option结束
    };
  },
  watch: {
    //观察option的变化
    options1: {
      handler(newVal, oldVal) {
        console.log(newVal, oldVal);
        if (this.myCharts) {
          if (newVal) {
            this.myCharts.setOption(newVal);
          } else {
            this.myCharts.setOption(oldVal);
          }
        } else {
          this.initChart1();
        }
      },
      deep: true //对象内部属性的监听，关键。
    },
    options2: {
      handler(newVal, oldVal) {
        console.log(newVal, oldVal);
        if (this.myCharts2) {
          if (newVal) {
            this.myCharts2.setOption(newVal);
          } else {
            this.myCharts2.setOption(oldVal);
          }
        } else {
          this.initChart2();
        }
      },
      deep: true //对象内部属性的监听，关键。
    }
  },
  methods: {
    widthAbs(event) {
      event = Math.abs(event / 10 * 100) + "%"
      return event
    },
    getData() {
      axios.get("/js/data.json").then(
        response => {
          this.dataList = response.data;
          console.log(response.data)
          // this.options1.xAxis[0].data = esponse.data.medicinal.medicine_date
          // this.options1.series[0].data = esponse.data.medicinal.medicine_dataNum
        },
        response => {
          console.log("error");
        }
      );
    },
    // 组件的方法
    getTime: function() {
      let myDate = new Date();
      let _this = this;
      let yy = new Date().getFullYear();
      let mm = new Date().getMonth() + 1;
      let dd = new Date().getDate();
      let hh = new Date().getHours();
      let mf =
        new Date().getMinutes() < 10
          ? "0" + new Date().getMinutes()
          : new Date().getMinutes();
      let ss =
        new Date().getSeconds() < 10
          ? "0" + new Date().getSeconds()
          : new Date().getSeconds();
      let wk = myDate.getDay();
      _this.nowTime = hh + ":" + mf;
      let weeks = [
        "星期日",
        "星期一",
        "星期二",
        "星期三",
        "星期四",
        "星期五",
        "星期六"
      ];
      let week = weeks[wk];
      _this.nowWeek = week;
      _this.nowDate = mm + "-" + dd;
    },
    currentTime() {
      setInterval(this.getTime, 500);
    },
    //echarts地图方法
    getOptions() {
      this.$set(this.option);
      return this.option;
    },
    runEchartsmap() {
      this.echartObj = echarts.init(document.getElementById(this.id));
      echarts.registerMap("六盘水", JSON);
      let that = this
      axios.get("/js/data.json").then(
        response => {
          // console.log(response.data);
          this.option.series[0].data = response.data.map_data_person;
          // options1.series[0].data = response.data.medicinal.medicine_dataNum;
          // options1.series[1].data = response.data.medicinal.supplies_dataNum;
          that.echartObj.setOption(this.getOptions(), true);
        },
        response => {
          console.log("error");
        }
      );
      // this.echartObj.setOption(this.getOptions(), true);
    },
    initChart1() {
      const myCharts = this.$echarts.init(this.$refs.myCharts);
      let options1 = {
        grid: {
          left: "5%",
          right: "10%",
          top: "20%",
          bottom: "15%",
          containLabel: true
        },
        tooltip: {
          show: true,
          trigger: "item"
        },
        legend: {
          show: true,
          x: "center",
          y: "35",
          icon: "stack",
          itemWidth: 22,
          itemHeight: 10,
          textStyle: {
            color: "#E4FAFF",
            fontSize: "18"
          },
          data: ["药品", "物资"]
        },
        xAxis: [
          {
            type: "category",
            boundaryGap: false,
            axisLabel: {
              color: "#30eee9"
            },
            axisLine: {
              show: true,
              lineStyle: {
                color: "#397cbc"
              }
            },
            axisTick: {
              show: false
            },
            splitLine: {
              show: true,
              lineStyle: {
                color: "#195384"
              }
            },
            data: []
          }
        ],
        yAxis: [
          {
            type: "value",
            name: "药品",
            min: 0,
            max: 1000,
            axisLabel: {
              formatter: "{value}",
              textStyle: {
                color: "#2ad1d2"
              }
            },
            axisLine: {
              lineStyle: {
                color: "#27b4c2"
              }
            },
            axisTick: {
              show: false
            },
            splitLine: {
              show: true,
              lineStyle: {
                color: "#11366e"
              }
            }
          },
          {
            type: "value",
            name: "物资",
            min: 0,
            max: 1000,
            axisLabel: {
              formatter: "{value} 件",
              textStyle: {
                color: "#186afe"
              }
            },
            axisLine: {
              lineStyle: {
                color: "#186afe"
              }
            },
            axisTick: {
              show: false
            },
            splitLine: {
              show: true,
              lineStyle: {
                color: "#11366e"
              }
            }
          }
        ],
        series: [
          {
            name: "药品",
            type: "line",
            stack: "总量",
            symbol: "circle",
            symbolSize: 8,
            itemStyle: {
              normal: {
                color: "#0092f6",
                lineStyle: {
                  color: "#0092f6",
                  width: 1
                },
                areaStyle: {
                  color: "rgb(12,52,105)"
                }
              }
            },
            markPoint: {
              itemStyle: {
                normal: {
                  color: "red"
                }
              }
            },
            data: []
          },
          {
            name: "物资",
            type: "line",
            stack: "总量",
            symbol: "circle",
            symbolSize: 8,

            itemStyle: {
              normal: {
                color: "#00d4c7",
                lineStyle: {
                  color: "#00d4c7",
                  width: 1
                },
                areaStyle: {
                  color: "rgb(9,66,94)"
                }
              }
            },
            data: []
          }
        ]
      };

      axios.get("/js/data.json").then(
        response => {
          this.dataList = response.data;
          // console.log(options1);
          options1.xAxis[0].data = response.data.medicinal.medicine_date;
          options1.series[0].data = response.data.medicinal.medicine_dataNum;
          options1.series[1].data = response.data.medicinal.supplies_dataNum;
          myCharts.setOption(options1, true);
        },
        response => {
          console.log("error");
        }
      );
    },
    initChart2() {
      const myCharts2 = this.$echarts.init(this.$refs.myCharts2);
      // 左下折线图结束
      let options2 = {
        tooltip: {
          trigger: "axis",
          axisPointer: {
            type: "shadow"
          }
        },
        legend: {
          data: ["发热人数", "疑似人数", "重点患者"],
          align: "left",
          right: 50,
          top: 10,
          textStyle: {
            color: "#fff",
            fontSize: "18"
          },
          itemWidth: 22,
          itemHeight: 10,
          itemGap: 35
        },
        grid: {
          left: "3%",
          right: "4%",
          bottom: "3%",
          containLabel: true
        },
        xAxis: [
          {
            type: "category",
            name: "日期",
            data: [
              "02-01",
              "02-02",
              "02-03",
              "02-04",
              "02-05",
              "02-06",
              "02-07"
            ],
            axisLine: {
              show: true,
              lineStyle: {
                color: "#E4FAFF",
                width: 1,
                type: "solid"
              }
            },
            axisTick: {
              show: false
            },
            axisLabel: {
              show: true,
              textStyle: {
                color: "#E4FAFF",
                fontSize: "16"
              }
            }
          }
        ],
        yAxis: [
          {
            type: "value",
            name: "人数",
            axisLabel: {
              formatter: "{value} "
            },
            axisTick: {
              show: false
            },
            axisLine: {
              show: false,
              lineStyle: {
                color: "#E4FAFF",
                width: 1,
                type: "solid"
              }
            },
            axisLabel: {
              show: true,
              textStyle: {
                color: "#E4FAFF",
                fontSize: "16"
              }
            },
            splitLine: {
              lineStyle: {
                color: "#063374"
              }
            }
          }
        ],
        series: [
          {
            name: "发热人数",
            type: "bar",
            data: [],
            barWidth: 20, //柱子宽度
            barGap: 0.5, //柱子之间间距
            itemStyle: {
              normal: {
                color: "rgb(9,168,255)"
              }
            }
          },
          {
            name: "疑似人数",
            type: "bar",
            data: [],
            barWidth: 20,
            barGap: 0.5,
            itemStyle: {
              normal: {
                color: "rgb(7,107,253)"
              }
            }
          },
          {
            name: "重点患者",
            type: "bar",
            data: [],
            barWidth: 20,
            barGap: 0.5,
            itemStyle: {
              normal: {
                color: "rgb(10,255,254)"
              }
            }
          }
        ]
      };
      axios.get("/js/data.json").then(
        response => {
          this.dataList = response.data;
          // console.log(options2);
          options2.series[0].data = response.data.dailyStatistics.fever_person;
          options2.series[1].data =
            response.data.dailyStatistics.suspect_person;
          options2.series[2].data =
            response.data.dailyStatistics.key_sick_person;
          myCharts2.setOption(options2, true);
        },
        response => {
          console.log("error");
        }
      );
      // 底部echarts结束
    }
  },
  computed: {
    // computed擅长处理的场景：一个数据受多个数据影响
  },
  beforeCreate: function() {
    // 在实例初始化之后，数据观测(data observer) 和 event/watcher 事件配置之前被调用。
  },
  created: function() {
    this.currentTime();
    this.getData();

    // 实例已经创建完成之后被调用。在这一步，实例已完成以下的配置：数据观测(data observer)，属性和方法的运算， watch/event 事件回调。然而，挂载阶段还没开始，$el 属性目前不可见。
  },
  beforeMount: function() {
    // 在挂载开始之前被调用：相关的 render 函数首次被调用。
  },
  mounted: function() {
    // this.open_width1 = widthAbs(dataList.openData.openPatients[0] / 10 * 100 + '%')
    this.initChart1();
    this.initChart2();
    this.runEchartsmap();
    // 编译好的HTML挂载到页面完成后执行的事件钩子
    // el 被新创建的 vm.$el 替换，并挂载到实例上去之后调用该钩子。
    // 此钩子函数中一般会做一些ajax请求获取数据进行数据初始化
    // console.log(JSON.parse(this.dataList))

    console.log("Home done");
  },
  beforeUpdate: function() {
    // 数据更新时调用，发生在虚拟 DOM 重新渲染和打补丁之前。 你可以在这个钩子中进一步地更改状态，这不会触发附加的重渲染过程。
  }
};
</script>

<style lang="stylus">
@font-face
  font-family 'uniDreamLED'
  src url('./assets/UnidreamLED.woff2') format('woff2'), url('./assets/UnidreamLED.woff') format('woff')
::-webkit-scrollbar
  display none
#app
  font-family 'Avenir', Helvetica, Arial, sans-serif
  -webkit-font-smoothing antialiased
  -moz-osx-font-smoothing grayscale
  color #D8F4FB
  #all-window
    width 1920px
    height 1080px
    background url('./imgs/background.jpg') center center no-repeat
    #nav
      width 1920px
      height 100px
      background url('./imgs/top.png') center center no-repeat
    .location
      position absolute
      top 30px
      left 30px
      font-size 20px
      color #fff
      i
        display inline-block
        width 17px
        height 25px
        margin-right 12px
        background url('./imgs/local.png') center center no-repeat
        vertical-align bottom
    .calendar
      .time
        position absolute
        top 37px
        left 1743px
        width 88px
        font-size 30px
      .dd-date
        width 45px
        height 30px
        position absolute
        top 35px
        left 1849px
        font-size 14px
        .date
          text-align center
    #mid_three
      text-align center
      #mid_1
        position absolute
        top 122px
        left 565px
      #mid_2
        position absolute
        top 122px
        left 822px
      #mid_3
        position absolute
        top 122px
        left 1079px
      .mid-box
        width 200px
        line-height 62px
        .mid-text
          font-size 18px
          line-height 18px
        .mid-num-box
          width 170px
          height 60px
          padding-right 30px
          padding-top 2px
          color #00CCFF
          font-size 32px
          font-family 'uniDreamLED'
          text-align right
          background url('./imgs/tu.png') center center no-repeat
    .title-text
      color #E4FAFF
      font-size 24px
      width 200px
      margin-left 20px
      line-height 24px
      i
        display inline-block
        width 5px
        height 34px
        margin-right 13px
        background #179DFB
        vertical-align bottom
    .guiyang_data
      width 524px
      height 324px
      background url('./imgs/1.png') center center no-repeat
      position absolute
      top 164px
      left 20px
      .case-text
        position absolute
        top 30px
        left 20px
        font-size 24px
        li
          margin-bottom 60px
          span
            font-size 14px
            margin-left 78px
          span:nth-child(1)
            margin-left 50px
      .case-bar
        position absolute
        top 75px
        left 20px
        color #05CDFF
        line-height 24px
        li
          width 404px
          height 24px
          background rgb(7, 59, 101)
          border-radius 8px
          margin-bottom 68px
          position relative
          i
            display inline-block
            width 240px
            height 24px
            background #00D6FA
            border-radius 8px
          span
            display inline-block
            margin 2px 0 0 16px
            font-size 24px
            line-height 24px
            vertical-align top
            margin-right 10px
          s
            height 14px
            position absolute
            top 4px
            display inline-block
            font-size 14px
            text-decoration none
    #drug
      width 300px
      position absolute
      top 512px
    .durg-data
      width 526px
      height 505px
      background url('./imgs/2.png') center center no-repeat
      position absolute
      top 560px
      left 20px
      #myCharts
        width 500px
        height 500px
        margin-top 30px
    #epidemic_warning
      position absolute
      left 1718px
      top 108px
      i
        margin 0 0 0 10px
    .epidemic-data
      width 561px
      height 298px
      background url('./imgs/3.png') center center no-repeat
      position absolute
      top 164px
      left 1331px
      font-size 24px
      .sample
        position relative
        :nth-child(1)
          position absolute
          top 24px
          left 74px
        :nth-child(3)
          position absolute
          top 24px
          left 347px
        :nth-child(2)
          position absolute
          top 86px
          left 108px
          color #09FFFC
          font-size 48px
        :nth-child(4)
          position absolute
          top 86px
          left 381px
          color #09FFFC
          font-size 48px
      .res-i
        i
          color #09FFFC
          font-size 30px
          line-height 30px
          vertical-align bottom
          margin-left 25px
      .positive
        position absolute
        top 180px
        left 76px
      .pos-2
        position absolute
        top 180px
        left 345px
      .negative
        position absolute
        top 232px
        left 76px
      .neg-2
        position absolute
        top 232px
        left 345px
    .point-patient
      width 561px
      height 306px
      background url('./imgs/4.png') center center no-repeat
      position absolute
      top 472px
      left 1331px
      .patient-title
        position absolute
        top 17px
        left 73px
        font-size 24px
        color #09FFFC
      .patient-list
        position absolute
        top 68px
        left 74px
        font-size 18px
        color #FAFFFF
        overflow hidden
        ul
          width 480px
          height 230px
          overflow-y auto
          li
            margin-bottom 20px
            position relative
            .sex
              position absolute
              top 0
              left 87px
            .age
              position absolute
              top 0
              left 136px
            .phone-num
              position absolute
              top 0
              left 188px
            .date-sicken
              position absolute
              left 341px
    #day_of_person
      width 280px
      position absolute
      top 755px
      left 552px
    .statistics-allDay
      width 1321px
      height 264px
      background url('./imgs/5.png') center center no-repeat
      background-size contain
      position absolute
      top 798px
      left 569px
      #myCharts2
        width 1320px
        height 260px
    .map-screen
      width 720px
      height 640px
      position absolute
      top 180px
      left 620px
  .map_tooltip
    width 100px
    height 50px
    text-align center
  #deadline
    position absolute
    top 115px
    left 250px
</style>
