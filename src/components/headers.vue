<template>
  <div class="headers">
    <div class="head-cen">
      <div class="heade-left">
        <ul>
          <li>角色选择(暂时不可用)</li>
          <li>设置(暂时不可用)</li>
          <li @click="changePas">修改密码</li>
          <li @click="openStrategy">攻略(新手请点击这个)</li>
          <li @click="games">小游戏</li>
        </ul>
      </div>
      <div class="heade-right">
        <ul>
          <li @click="out">退出登录</li>
        </ul>
      </div>

      <!-- 攻略弹出框 -->
      <Modal
        width="620"
        title="攻略"
        v-model="strategy"
        @on-ok="this.strategy = false"
        :styles="{ top: '40px' }"
      >
        <p>
          目前装备只开放了0-50级,地图也只开放了0-50级,技能目前只在秦郡城和灵霄山脉(强力单体和群攻)掉落
        </p>
        <p>每升一级加3个属性点,装备品质分为,黑色,黄色,绿色,蓝色,紫色,红色</p>
        <p>如果有任何bug 或者有任何意见 可以加群反馈,qq群号: 808647555</p>
        <p>
          铜钱作用买商店物品(目前商店未开放) 金币用来升级技能 和强化高星装备
        </p>
        <p>装备强化目前是没有效果的</p>
        <br />
        <p>地图掉落说明</p>
        <div class="mapLoots">
          <p v-for="(mapLoots, index) in mapLootsList" :key="index">
            {{ mapLoots.mapName }}:
            <!-- {{item.itemsVoList[0] }} -->
            <template v-if="mapLoots.itemsVoList">
              <span v-for="(loot, idx) in mapLoots.itemsVoList" :key="idx">
                <template v-if="loot">
                  <span :style="{ color: loot.color }"
                    >{{ loot.itemName }}
                  </span>
                </template>
              </span>
            </template>
          </p>
        </div>

        <br />
        <p>测试版: v0.1(2020-10-19)</p>
        <p>1: 修复了主动技能不能脱下的问题</p>
        <p>2: 修复了背包超出,底部留白的问题</p>
        <p>3: 修复了手机端出现白边的问题</p>
        <p>4: 修改了人物攻击为橙色,怪物攻击为蓝色</p>
        <p>5: 战斗间隙时间增加2秒 可以方便看掉落物品</p>
        <br />
        <p>测试版: v0.2(2020-10-20)</p>
        <p>1: 增加了修改密码,增加了技能升级</p>
        <p>
          2:
          装备防御力显示的问题,之前装备显示上没加防御力,实际上是有防御力的,显示问题已经修复
        </p>
        <p>
          3:
          下一步会做兑换码功能,可以进群免费拿兑换码,兑换码可以领取金币,然后就是强化装备(最高强化10次
          每次在原基础属性上加15%-20%的属性,前5级用铜币 后5级用金币)
        </p>
        <br />
        <p>测试版: v0.3(2020-10-21)</p>
        <p>1: 怪物暴击率调整普通怪物20%个别特殊的20%及以上</p>
        <p>2: 怪物暴击伤害200%下降到150%</p>
        <p>3: 加强了腰带,戒指,项链,现在精神和法力值加的更多了</p>
        <p>4: 增加了怪物前缀,会随机让怪物属性提高或者降低</p>
        <p>5: 增加了一个地图掉落表</p>
        <p>6: 修改了切换地图</p>
        <p>7: 全部怪物生命值下降10%</p>
        <p>8: 增加一键兑换金币</p>
        <br />
        <p>测试版: v0.4(2020-10-22)</p>
        <p>1: 修复了经验值问题</p>
        <p>2: 新加一个小游戏,别踩白块儿</p>
        <p>测试版: v0.5(2020-10-23)</p>
        <p>1: 增加了道具使用(洗点，背包扩充)</p>
      </Modal>

      <!-- 修改密码弹出框 -->
      <Modal
        width="620"
        title="修改密码"
        v-model="pasEject"
        @on-ok="confirmPassword()"
        :styles="{ top: '100px' }"
      >
        <div class="pasEject">
          <Input v-model="originalPas" placeholder="请输入原密码" />
          <Input v-model="newPas" placeholder="请输入新密码" />
          <Input v-model="confirmPas" placeholder="确认密码" />
        </div>
      </Modal>

      <!-- 小游戏 -->
      <Modal
        width="620"
        title="别踩白块儿"
        v-model="gamesEject"
        :styles="{ top: '40px' }"
      >
        <div class="box">
          <div id="cont">
            <div id="go">
              <span>点击开始</span>
            </div>
            <div id="main"></div>
          </div>
          <div id="count"></div>
        </div>
      </Modal>
    </div>
  </div>
</template>

<script>
export default {
  name: "headers",
  data() {
    return {
      strategy: false, // 攻略弹出框
      pasEject: false, // 修改密码弹出框
      gamesEject: false, // 小游戏弹出框
      originalPas: "", // 原密码
      newPas: "", // 新密码
      confirmPas: "", // 确认密码
      mapLootsList: undefined, // 地图掉落
    };
  },
  methods: {
    // 退出登录
    out() {
      this.$router.push({ name: "Login" });
    },

    // 打开攻略
    openStrategy() {
      this.strategy = true;

      if (this.mapLootsList) return;

      this.$http
        .post("/gameChara/mapDropItems")
        .then((res) => {
          // this.$Message.success(res.data.msg);
          console.log(res.data.data);
          this.mapLootsList = res.data.data;
        })
        .catch((err) => {
          this.$Message.warning("获取地图掉落失败");
        });
    },

    // 打开修改密码
    changePas() {
      this.originalPas = "";
      this.newPas = "";
      this.confirmPas = "";
      this.pasEject = true;
    },

    // 修改密码弹出框的确认
    confirmPassword() {
      this.$http
        .post(
          "/gamepassport/updatePwd?userId=" +
            this.getCookie("userId") +
            "&originalPwd=" +
            this.originalPas +
            "&newPwd=" +
            this.newPas +
            "&confirmPwd=" +
            this.confirmPas
        )
        .then((res) => {
          this.$Message.success(res.data.msg);
        })
        .catch((err) => {
          this.$Message.warning("修改密码失败,请联系管理员");
        });
    },

    // 小游戏
    games() {
      this.gamesEject = true;
    },
  },
  mounted() {
    //获取内容区里面得#main和#go，以及获取计数区。
    var main = document.getElementById("main");
    var go = document.getElementById("go");
    var count = document.getElementById("count");
    //设置四种颜色
    var cols = ["#1AAB8A", "#E15650", "#121B39", "#80A84E"];
    //动态创建div
    function CDiv(classname) {
      //创建div节点，为一行
      var Div = document.createElement("div");
      //生成随机数，Math.floor()是四舍五入的作用，产生的数永远不会大于4，产生的随机数表示那个有颜色的那一个
      var index = Math.floor(Math.random() * 4);
      //设置class
      Div.className = classname;

      //在一行里面动态添加四个div，一行里面的四块
      for (var i = 0; i < 4; i++) {
        var iDiv = document.createElement("div");
        Div.appendChild(iDiv);
      }

      console.log(Div);
      //判断#main里面是否有元素
      if (main.children.length == 0) {
        main.appendChild(Div);
      } else {
        //如果有元素，则在该元素之前插入
        main.insertBefore(Div, main.children[0]);
      }
      //随机的设置四个div块的背景颜色
      Div.children[index].style.backgroundColor = cols[index];
      console.log(cols[index]);
      Div.children[index].className = "i";
    }
    function move(obj) {
      //默认速度与计分
      var speed = 5,
        num = 0;
      //定义一个定时器
      obj.timer = setInterval(function () {
        //速度
        var step = parseInt(getComputedStyle(obj, null)["top"]) + speed;
        obj.style.top = step + "px";
        if (parseInt(getComputedStyle(obj, null)["top"]) >= 0) {
          CDiv("row");
          obj.style.top = -150 + "px";
        }
        if (obj.children.length == 6) {
          for (var i = 0; i < 4; i++) {
            if (
              obj.children[obj.children.length - 1].children[i].className == "i"
            ) {
              //游戏结束
              obj.style.top = "-150px";
              count.innerHTML = "游戏结束,最高得分: " + num;
              //关闭定时器
              clearInterval(obj.timer);
              //显示开始游戏
              go.children[0].innerHTML = "游戏结束";
              go.style.display = "block";
            }
          }
          obj.removeChild(obj.children[obj.children.length - 1]);
        }
        //点击与计分
        obj.onmousedown = function (event) {
          //点击的不是白盒子
          // 兼容IE
          event = event || window.event;
          if (
            (event.target ? event.target : event.srcElement).className == "i"
          ) {
            //点击后的盒子颜色
            (event.target
              ? event.target
              : event.srcElement
            ).style.backgroundColor = "#bbb";
            //清除盒子标记
            (event.target ? event.target : event.srcElement).className = "";
            //计分
            num++;
            //显示得分
            count.innerHTML = "当前得分: " + num;
          } else {
            //游戏结束
            obj.style.top = 0;
            count.innerHTML = "游戏结束,最高得分: " + num;
            //关闭定时器
            clearInterval(obj.timer);
            //显示开始游戏
            go.children[0].innerHTML = "游戏结束";
            go.style.display = "block";
          }
          //盒子加速
          if (num % 10 == 0) {
            speed++;
          }
        };
        //松开触发停止
        obj.onmouseup = function (event) {};
      }, 20);
    }
    go.children[0].onclick = function () {
      if (main.children.length) {
        //暴力清除main里面所有盒子
        main.innerHTML = "";
      }
      //清空计分
      count.innerHTML = "游戏开始";
      //隐藏开始盒子
      this.parentNode.style.display = "none";
      move(main);
    };
  },
};
</script>
<style>
.headers {
  height: 60px;
  line-height: 60px;
  min-width: 1140px;
  color: #fff;
  /* backdrop-filter: blur(20px); */
  background: rgba(95, 95, 95, 0.3);
  box-shadow: 3px 3px 6px 3px rgba(0, 0, 0, 0.3);
  margin-bottom: 10px;
}

.head-cen {
  width: 1140px;
  margin: 0 auto;
  overflow: hidden;
}
.heade-left {
  float: left;
}
.heade-left ul,
.heade-right ul {
  overflow: hidden;
}
.heade-left ul li,
.heade-right ul li {
  float: left;
  height: 60px;
  padding: 0 10px;
  cursor: pointer;
}
.heade-left ul li:hover,
.heade-right ul li:hover {
  background: #666;
  transition: all 0.6s;
}
.heade-right {
  float: right;
}

.pasEject /deep/ .ivu-input-wrapper {
  margin-bottom: 20px;
}
.mapLoots p {
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
  line-height: 2;
}

/* 小游戏 */
.box {
  margin: 50px auto 0 auto;
  width: 400px;
  height: auto;
  border: solid 1px #222;
}

/* 内容区域 */
#cont {
  width: 400px;
  height: 600px;
  position: relative;
  overflow: hidden;
}

/* 内容区域里面的go区域 */
#go {
  width: 100%;
  height: 600px;
  position: absolute;
  top: 0;
  font: 700 60px "微软雅黑";
  text-align: center;
  z-index: 99;
}
#go span {
  cursor: pointer;
  background-color: #fff;
  border-bottom: solid 1px #222;
}

/* 内容区域里面的游戏区域 ，注意这里定位向上移了一行的距离*/
#main {
  width: 400px;
  height: 600px;
  position: relative;
  top: -150px;
}
/* 设置每一行的高度 */
.row {
  width: 400px;
  height: 150px;
}
/* 设置行里面的每一个格子 */
.row div {
  width: 99px;
  height: 149px !important;
  border: solid 1px #222;
  float: left;
  border-top-width: 0;
  border-left-width: 0;
  cursor: pointer;
}
/* 计数区域 */
#count {
  border-top: solid 1px #222;
  width: 400px;
  height: 50px;
  font: 700 36px/50px "微软雅黑";
  text-align: center;
}
</style>
