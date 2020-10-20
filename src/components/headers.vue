<template>
  <div class="headers">
    <div class="head-cen">
      <div class="heade-left">
        <ul>
          <li>角色选择(暂时不可用)</li>
          <li>设置(暂时不可用)</li>
          <li @click="changePas">修改密码</li>
          <li @click="openStrategy">攻略(新手请点击这个)</li>
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
        :styles="{ top: '100px' }"
      >
        <p>
          目前装备只开放了0-50级,地图也只开放了0-50级,技能目前只在秦郡城和灵霄山脉(强力单体和群攻)掉落
        </p>
        <p>每升一级加3个属性点,装备品质分为,黑色,黄色,绿色,蓝色,紫色,红色</p>
        <p>如果有任何bug 或者有任何意见 可以加群反馈,qq群号: 808647555</p>
        <p>
          铜钱作用买商店物品(目前商店未开放) 金币用来升级技能 和强化高星装备
        </p>
        <p>装备强化,和技能升级,主动技能脱下目前是没有效果的</p>
        <br />
        <p>测试版: v0.1(2020-10-19)</p>
        <p>1: 修复了主动技能不能脱下的问题</p>
        <p>2: 修复了背包超出,底部留白的问题</p>
        <p>3: 修复了手机端出现白边的问题</p>
        <p>4: 修改了人物攻击为橙色,怪物攻击为蓝色</p>
        <p>5: 战斗间隙时间增加2秒 可以方便看掉落物品</p>
        <br>  
        <p>测试版: v0.2(2020-10-20)</p>
        <p>1: 增加了修改密码</p>
        <p>2: 增加了技能升级,技能升级是用金币的,金币目前打副本不会掉落</p>
        <p>3: 装备防御力显示的问题,之前装备显示上没加防御力,实际上是有防御力的,显示问题已经修复</p>
        <p>4: 下一步会做兑换码功能,可以进群免费拿兑换码,兑换码可以领取金币,然后就是强化装备(最高强化10次 每次在原基础属性上加15%-20%的属性,前5级用铜币 后5级用金币)</p>
      </Modal>

      <!-- 修改密码弹出框 -->
      <Modal
        width="620"
        title="攻略"
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
      originalPas: "", // 原密码
      newPas: "", // 新密码
      confirmPas: "" // 确认密码
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
        .then(res => {
          this.$Message.success(res.data.msg);
        })
        .catch(err => {
          this.$Message.warning("修改密码失败,请联系管理员");
        });
    }
  }
};
</script>

<style scoped>
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
</style>
