<template>
  <div class="headers">
    <div class="head-cen">
      <div class="heade-left">
        <ul>
          <li @click="openOrganize">组队</li>
          <li @click="openSetEject">设置</li>
          <li @click="changePas">修改密码</li>
          <li @click="openStrategy">更新日志</li>
          <li @click="openExchange">兑换码</li>
          <li @click="PlanEject = true">排行榜</li>
          <li @click="RackingEject = true">在线玩家</li>
        </ul>
      </div>
      <div class="heade-right">
        <ul>
          <li @click="out">退出登录</li>
        </ul>
      </div>

      <!-- 组队 -->
      <Modal
        width="720"
        :title="organizeList.armyName ? organizeList.armyName : '组队'"
        v-model="organizeEject"
        :styles="{ top: '100px' }"
        class="organizeEject"
      >
        <div
          v-for="(item, idx) in organizeList.charaInfoVoList"
          :key="idx"
          class="organizeList"
        >
          <div v-if="item.avatar" style="position: relative; float: left">
            <img
              :src="item.avatar"
              alt="老弟,没图片找群主要啊"
              class="organizeList_face"
            />
            <span @click="kickOut(item)" v-if="item.status == 0">
              <Icon type="md-close" />
            </span>
          </div>
          <div
            v-else
            style="
              text-align: center;
              font-size: 60px;
              cursor: pointer;
              position: relative;
              float: left;
            "
          >
            <Icon type="ios-body" class="organizeList_face" />

            <span @click="kickOut(item)" v-if="item.status == 0">
              <Icon type="md-close" />
            </span>
          </div>
          <p class="organizeList_name" v-if="item.charaName">
            {{ item.charaName }} {{ item.charaLevel }} 级
          </p>
        </div>
        <Select v-model="mapName">
          <!-- ({{ item.minLv }}) -->
          <Option
            v-for="item in mapList"
            :value="item.mapName"
            :name="item.mapId"
            :key="item.mapId"
          >
            {{ item.mapName }}{{ item.minLv }}-{{ item.maxLv }}
          </Option>
        </Select>
        <Button
          @click="CombatRecord()"
          :loading="recordStatus"
          style="margin-top: 10px"
        >
          战斗记录
        </Button>
        <template v-if="combatRecordProcess">
          <p
            v-for="(item, idx) in combatRecordProcess"
            :key="item.reHealth"
            style="font-size: 18px; margin: 5px 0"
          >
            第{{ ++idx }}场战斗
            <span style="color: red">
              {{ combatRecordProcess.length == idx ? "失败" : "胜利" }}
            </span>
            剩余 {{ item.reHealth }}血量 剩余 {{ item.reMana }}真气 获取
            {{ item.dropExp }} 经验 物品: {{ item.gameItemsList1 || "空" }}
          </p>
        </template>
        <div slot="footer">
          <Button
            type="primary"
            size="small"
            @click="join"
            v-if="teamStatus == 0"
          >
            队伍列表
          </Button>
          <Button
            type="primary"
            size="small"
            @click="establish"
            v-if="teamStatus == 0"
          >
            创建队伍
          </Button>
          <Button
            type="primary"
            size="small"
            @click="quitTheTeam"
            v-if="teamStatus == 1"
          >
            退出队伍
          </Button>
        </div>
      </Modal>

      <!-- 队伍名字弹出框 -->
      <Modal
        width="320"
        title="队伍名字"
        v-model="armyEject"
        @on-ok="changeEstablish()"
        :styles="{ top: '100px' }"
      >
        <div class="pasEject">
          <Input v-model="armyName" placeholder="请输入队伍名字" />
        </div>
      </Modal>

      <!-- 队伍列表弹出框 -->
      <Modal
        width="520"
        title="队伍列表"
        v-model="armyListEject"
        :styles="{ top: '100px' }"
      >
        <div
          v-for="(item, idx) in armyList"
          :key="idx"
          style="margin-bottom: 5px"
        >
          队伍名:
          <span style="color: red">{{ item.armyName }}</span>
          &nbsp;&nbsp;&nbsp;&nbsp; 队长名字:
          <span style="color: #0090ff">{{ item.captainName }}</span>
          &nbsp;&nbsp;&nbsp;&nbsp; 剩余位置:
          <span style="color: rgb(232 0 255)">{{ item.armyNum }}</span>
          &nbsp;&nbsp;&nbsp;&nbsp;
          <Button type="primary" size="small" @click="JoinTheTeam(item)">
            加入队伍
          </Button>
        </div>
      </Modal>

      <!-- 设置 -->
      <Modal
        width="750"
        title="设置"
        v-model="setEject"
        @on-ok="changeSet()"
        :styles="{ top: '100px' }"
      >
        <Form ref="formInline" :model="gamePackageBo" :label-width="80" inline>
          <FormItem label="自动出售">
            <i-switch
              v-model="gamePackageBo.isOpen"
              :true-value="1"
              :false-value="0"
            />
          </FormItem>
          <FormItem label="最低等级">
            <Input type="text" v-model="gamePackageBo.minLevel"></Input>
          </FormItem>
          <FormItem label="最高等级">
            <Input type="text" v-model="gamePackageBo.maxLevel"></Input>
          </FormItem>
          <FormItem label="品质列表">
            <CheckboxGroup v-model="gamePackageBo.quality">
              <Checkbox label="1">黑色</Checkbox>
              <Checkbox label="2">黄色</Checkbox>
              <Checkbox label="3">绿色</Checkbox>
              <Checkbox label="4">蓝色</Checkbox>
              <Checkbox label="5">紫色</Checkbox>
              <Checkbox label="6">红色</Checkbox>
            </CheckboxGroup>
          </FormItem>
        </Form>
      </Modal>

      <!-- 攻略弹出框 -->
      <Modal
        width="620"
        title="更新日志"
        v-model="strategy"
        @on-ok="this.strategy = false"
        :styles="{ top: '40px' }"
      >
        <p>进群可以免费领取兑换码</p>
        <p class="importantFont">
          角色50级之后可以带3个被动 前期35级前尽量别强化装备
          用来升级技能或者后面用
        </p>
        <br />
        <p class="importantFont">
          每升一级加3个属性点,装备品质分为,黑色,黄色,绿色,蓝色,紫色,红色
        </p>
        <br />

        <p class="importantFont">
          如果有任何bug 或者有任何意见 反馈qq群:
          <span class="importantFont">808647555</span>
        </p>
        <br />

        <p class="importantFont">地图掉落说明</p>
        <p class="importantFont">
          普通图只掉落除红色以下与地图同等级的装备，挑战掉落紫色以上同等级装备。地狱掉落技能
        </p>
        <br />
        <!-- <p>地图掉落说明</p>
        <div class="mapLoots">
          <p v-for="(mapLoots, index) in mapLootsList" :key="index">
            {{ mapLoots.mapName }}:
            <template v-if="mapLoots.itemsVoList">
              <span v-for="(loot, idx) in mapLoots.itemsVoList" :key="idx">
                <template v-if="loot">
                  <span :style="{ color: distinguishColor(loot.color) }"
                    >{{ loot.itemName }}
                  </span>
                </template>
              </span>
            </template>
          </p>
        </div>
        <br /> -->
        <p>测试版: v0.11.3(2020-12-14)</p>
        <p class="importantFont">1: 预计12月14号删档</p>
        <p class="importantFont">2: 这中间都在准备地图 优化 调整</p>
        <br />
        <p>测试版: v0.11.2(2020-12-1)</p>
        <p class="importantFont">1: 新增批量合成</p>
        <p class="importantFont">2: 新增批量使用</p>
        <br />
        <p>测试版: v0.11.1(2020-11-27)</p>
        <p class="importantFont">1: 新增批量购买</p>
        <p class="importantFont">2: 新增合成</p>
        <br />
        <p>测试版: v0.11.0(2020-11-25)</p>
        <p class="importantFont">1: 新增境界(可以大幅度提升属性)</p>
        <br />
        <p>测试版: v0.10.9(2020-11-24)</p>
        <p class="importantFont">
          1: 装备新增：55-60级装备。（装备已加入55-60级地图）
        </p>
        <p class="importantFont">
          2: 装备强化：30-60级装备紫以下强化材料少量削减。红色强化提升提高。
        </p>
        <p>
          3: 60级装备大礼包：201124000079MXKT2H4H.
          强化材料大礼包：201124000079Y7BY4F14
        </p>
        <br />
        <p>测试版: v0.10.8(2020-11-16)</p>
        <p class="importantFont">1: 组队副本来了</p>
        <p class="importantFont">1: 组队副本必须满3个人</p>
        <br />
        <p>测试版: v0.10.7(2020-11-13)</p>
        <p class="importantFont">1: 组队先上线了..(战斗还差点)</p>
        <p>2: 战斗调整,buff可叠加</p>
        <br />
        <p>测试版: v0.10.6(2020-11-11)</p>
        <p class="importantFont">
          1: 新增编号(以后群里有问题就报编号,组队也需要编号)
        </p>
        <p class="importantFont">2: 更新上传头像(点击图片就可以)</p>
        <p class="importantFont">
          3: 新增自动战斗 (修改了 已角色id记录最近的地图
          不会出现换小号就不行的情况)
        </p>
        <p class="importantFont">
          4: 增强持续伤害buff 从每回合单次伤害 改为怪物攻击次数
        </p>
        <p class="importantFont">5: 反击增强，无视防御可暴击</p>
        <br />
        <p>测试版: v0.10.5(2020-11-10)</p>
        <p class="importantFont">1: 新增装备对比...</p>
        <br />
        <p>测试版: v0.10.4(2020-11-09)</p>
        <p class="importantFont">1: 用户体验优化</p>
        <p>2: 排行榜和在线玩家移动到导航条</p>
        <br />
        <p>测试版: v0.10.3(2020-11-07)</p>
        <p>1: 每一点灵力成的真气值削弱</p>
        <p>2: 灵巧加的攻击力增强</p>
        <p>3: 实装命中，闪避，暴击伤害</p>
        <p class="importantFont">4: 装备随机属性浮动</p>
        <p class="importantFont">5: 装备等级限制</p>
        <br />
        <p>测试版: v0.10.2(2020-11-05)</p>
        <p class="importantFont">1: 金币兑换关闭 通过商店去兑换</p>
        <p>2: 装备等级限制，功能道具等级使用限制，商店新增好运袋</p>
        <br />
        <p>测试版: v0.10.1(2020-11-04)</p>
        <p>1: 新增4个技能</p>
        <br />
        <p>测试版: v0.9(2020-11-01)</p>
        <p class="importantFont">1: 开放了强化</p>
        <br />
        <p>测试版: v0.8(2020-10-30)</p>
        <p class="importantFont">1: 自动出售</p>
        <p>2: 锁定</p>
        <br />
        <p>测试版: v0.7(2020-10-27)</p>
        <p>1: 添加等级排行榜</p>
        <p>2: 添加在线挂机玩家列表</p>
        <p>3: 添加金币排行榜和铜币排行榜</p>
        <p>
          4: 现有1-50级装备属性数据小幅度调整。
          新增：1-60级法系武器装备，40-60级物理武器装备。
          新增：30-50级戒指，30-50级腰带。
          新增：法系主动技能：落雷，雷击，火球，大火球，极寒风暴，寥落九天-天惩。
          新增：法系被动技能：冥思。
          新增：物理主动技能：连斩，疾风斩，真气爆发。
          新增：物理被动技能：好战。 新增：50-60级地图，新增50-60怪物。
          ：1-40级地图，怪物，装备爆率小幅度调整。
          ：煌龙炎月斩，九转凤鸣杀技能爆出低地点由20级地图灵霄山脉变更至50级新地图“云
          梦泽”。
        </p>
        <br />
        <p>测试版: v0.6(2020-10-26)</p>
        <p>1: 修复了挂机可能会结束的问题</p>
        <p>2: 界面大修改</p>
        <p>3: 修复全体技能在第二回合后无法攻击全部敌人</p>
        <p>4: 补上灵力体系攻击(需要补灵力装备进行测试)</p>
        <br />
        <p>测试版: v0.5(2020-10-23)</p>
        <p>1: 增加了道具使用(洗点，背包扩充)</p>
        <p>2: 增加了兑换码</p>
        <br />
        <p>测试版: v0.4(2020-10-22)</p>
        <p>1: 修复了经验值问题</p>
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
        <p>测试版: v0.1(2020-10-19)</p>
        <p>1: 修复了主动技能不能脱下的问题</p>
        <p>2: 修复了背包超出,底部留白的问题</p>
        <p>3: 修复了手机端出现白边的问题</p>
        <p>4: 修改了人物攻击为橙色,怪物攻击为蓝色</p>
        <p>5: 战斗间隙时间增加2秒 可以方便看掉落物品</p>
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

      <!-- 兑换码 -->
      <Modal
        width="620"
        title="兑换码"
        v-model="exchangeEject"
        @on-ok="changeExchange()"
        :styles="{ top: '100px' }"
      >
        <Input v-model="exchange" placeholder="请输入兑换码" />
      </Modal>

      <!-- 排行榜 -->
      <Modal
        width="620"
        title="排行榜"
        v-model="PlanEject"
        :styles="{ top: '100px' }"
      >
        <rankingList></rankingList>
      </Modal>

      <!-- 在线玩家 -->
      <Modal
        width="620"
        title="在线玩家"
        v-model="RackingEject"
        :styles="{ top: '100px' }"
      >
        <playersList></playersList>
      </Modal>
    </div>
  </div>
</template>

<script>
import playersList from "./playersList.vue";
import rankingList from "./rankingList.vue";
import banner1 from "../assets/back1.jpg";
export default {
  name: "headers",
  data() {
    return {
      recordStatus: false, // 战斗记录的一个状态
      mapName: "", // 选择查看的地图名字
      strategy: false, // 攻略弹出框
      pasEject: false, // 修改密码弹出框
      exchangeEject: false, // 兑换码弹出框
      PlanEject: false, // 排行榜弹出框
      RackingEject: false, // 在线玩家弹出框
      organizeEject: false, // 组队弹出框
      armyEject: false, // 队伍名字的弹出框
      armyListEject: false, // 队伍列表弹出框
      setEject: false, // 设置自动出售
      originalPas: "", // 原密码
      newPas: "", // 新密码
      confirmPas: "", // 确认密码
      mapLootsList: undefined, // 地图掉落
      exchange: "", // 兑换码
      teamStatus: 0, // 组队状态 0 无队伍 1有队伍
      armyName: "", //  队伍名字
      armyList: [], // 队伍列表
      organizeList: [], // 组队列表
      mapList: [], // 地图列表
      mapid: "", // 地图id
      combatRecordProcess: [], // 战斗记录过程
      gamePackageBo: {
        // 出售设置
        charaId: "",
        isOpen: 0,
        maxLevel: "",
        minLevel: "",
        quality: [],
      },
    };
  },
  created() {
    this.getPackageBo(); //自动出售设置的获取
    this.armyBoundary();
    this.TeamMapList(); // 获取组队地图列表
  },
  components: { playersList, rankingList },
  methods: {
    // 返回品质颜色
    distinguishColor(color) {
      if (color == 1) {
        return "#000";
      } else if (color == 2) {
        return "#c5c52f";
      } else if (color == 3) {
        return "#049c04";
      } else if (color == 4) {
        return "#005aff";
      } else if (color == 5) {
        return "#ff00e0";
      } else if (color == 6) {
        return "#ff0000";
      } else {
        return "#ff00c3e0";
      }
    },

    // 获取组队地图列表
    TeamMapList() {
      this.$http.get("/gameAmry/queryMapList").then((res) => {
        this.mapList = res.data.data;
      });
    },

    // 查看战斗记录
    CombatRecord() {
      if (!this.mapName) {
        this.$Message.warning("请在上方选择一个地图");
        return;
      }
      for (var i = 0; i < this.mapList.length; i++) {
        if (this.mapName == this.mapList[i].mapName) {
          this.mapid = this.mapList[i].mapId;
        }
      }
      this.$http
        .post(
          "/gameAmry/getAmryRecord?charaNo=" +
            this.$store.state.user.accountId +
            "&mapId=" +
            this.mapid
        )
        .then((res) => {
          this.combatRecordProcess = res.data.data;

          for (let i = 0; i < this.combatRecordProcess.length; i++) {
            if (this.combatRecordProcess[i].gameItemsList != null) {
              this.combatRecordProcess[
                i
              ].gameItemsList1 = this.combatRecordProcess[i].gameItemsList.map(
                (val) => {
                  return val.itemName;
                }
              );
            }
          }

          this.recordStatus = true;
          setTimeout(() => {
            this.recordStatus = false;
          }, 10000);
        })
        .catch((err) => {
          console.log(err);
        });
    },

    // 打开组队
    openOrganize() {
      this.organizeEject = true;
      this.armyBoundary();
    },

    // 进入队伍界面
    armyBoundary() {
      this.$http
        .post("/gameAmry/armyBoundary/?charaId=" + this.getCookie("charaId"))
        .then((res) => {
          if (res.data.msg == "无队伍") {
            this.teamStatus = 0;
            this.organizeList = {};
            this.organizeList.charaInfoVoList = [{}, {}, {}];
            this.$bus.$emit("teamStatusMsg1", "无队伍了");
          } else {
            this.teamStatus = 1;
            this.organizeList = res.data.data;
            this.$bus.$emit("teamStatusMsg", "有队伍了");
          }
        })
        .catch((err) => {
          console.log(err, "进入队伍界面");
        });
    },

    // 打开队伍列表
    join() {
      this.armyListEject = true;
      this.$http.get("/gameAmry/getArmyList?armyName").then((res) => {
        this.armyList = res.data.data;
      });
    },

    // 队伍列表里的加入按钮
    JoinTheTeam(item) {
      this.$http
        .post(
          "/gameAmry/playerJoinAmry/?charaId=" +
            this.getCookie("charaId") +
            "&amryId=" +
            item.armyId
        )
        .then((res) => {
          this.$Message.success(res.data.msg);
          this.armyBoundary();
          this.armyListEject = false;
        });
    },

    // 创建队伍
    establish() {
      this.armyEject = true;
    },

    // 队伍名字的确定
    changeEstablish() {
      this.$http
        .post(
          "/gameAmry/playerCreateAmry/?charaId=" +
            this.getCookie("charaId") +
            "&armyName=" +
            this.armyName
        )
        .then((res) => {
          this.$Message.success(res.data.msg);
          if (res.data.msg == "OK") {
            this.armyBoundary();
          }
          console.log(res, "队伍名字的确定");
        });
    },

    // 退出队伍
    quitTheTeam() {
      this.$http
        .post("/gameAmry/exitAmry/?charaId=" + this.getCookie("charaId"))
        .then((res) => {
          this.armyBoundary();
          this.$Message.success(res.data.msg);
        });
    },

    // 踢出队伍失败
    kickOut(item) {
      this.$http
        .post(
          "/gameAmry/kickOutAmry/?charaId=" +
            this.getCookie("charaId") +
            "&charaNo=" +
            item.charaNo
        )
        .then((res) => {
          this.armyBoundary();
          this.$Message.success(res.data.msg);
        });
    },

    // 自动出售设置的获取
    getPackageBo() {
      this.$http
        .post(
          "/gameMarket/getScreenPackage?charaId=" + this.getCookie("charaId")
        )
        .then((res) => {
          this.gamePackageBo = res.data.data;
          if (this.gamePackageBo.isOpen === null) this.gamePackageBo.isOpen = 0;
          this.gamePackageBo.quality = this.gamePackageBo.quality
            ? this.gamePackageBo.quality.split(",")
            : [];
        })
        .catch((err) => {
          this.$Message.warning("自动出售设置获取列表失败,请联系管理员");
        });
    },

    // 打开设置
    openSetEject() {
      this.setEject = true;
    },

    // 设置的确定
    changeSet() {
      if (!(this.gamePackageBo.minLevel >= 0)) {
        this.$Message.warning("最低等级不能为空");
        return;
      }
      if (!(this.gamePackageBo.maxLevel >= 0)) {
        this.$Message.warning("最高等级不能为空");
        return;
      }
      if (this.gamePackageBo.minLevel >= this.gamePackageBo.maxLevel) {
        this.$Message.warning("最小等级不能超过最大等级");
        return;
      }
      if (this.gamePackageBo.quality.length <= 0) {
        this.$Message.warning("品质列表必须选择一个");
        return;
      }

      this.$http
        .post("/gameMarket/setScreenPackage", {
          ...this.gamePackageBo,
          charaId: this.getCookie("charaId"),
          quality: this.gamePackageBo.quality.join(","),
        })
        .then((res) => {
          this.$Message.success(res.data.data);
        })
        .catch((err) => {
          this.$Message.warning("设置失败");
        });
    },

    // 退出登录
    out() {
      this.$bus.$emit("tcdlMsg", "退出登录那里发送过来的");
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
        .then((res) => {
          this.$Message.success(res.data.msg);
        })
        .catch((err) => {
          this.$Message.warning("修改密码失败,请联系管理员");
        });
    },

    // 打开兑换码
    openExchange() {
      this.exchangeEject = true;
      this.exchange = "";
    },

    // 兑换码确定
    changeExchange() {
      this.$http
        .post(
          "/gamepassport/exchangeCode?charaId=" +
            this.getCookie("charaId") +
            "&exchangeCode=" +
            this.exchange
        )
        .then((res) => {
          this.$Message.warning(res.data.msg);
        })
        .catch((err) => {
          this.$Message.warning("兑换失败,请联系管理员");
        });
      this.$bus.$emit("dhmMsg", "兑换码那里发送过来的");
    },
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

/* 重要更新或者文字突出 */
.importantFont {
  color: #007eff;
  font-size: 16px;
}

/* 组队列表 */
.organizeEject /deep/ .ivu-modal-body {
  /* display: flex;
  justify-content: space-between; */
}

.organizeList {
  margin-bottom: 15px;
  overflow: hidden;
  padding: 10px;
}

.organizeList_face {
  width: 100px;
  height: 100px;
  display: block;
  /* border: 1px dashed skyblue; */
  margin: 0 auto 15px;
  border-radius: 50%;
  box-shadow: 0px 0px 10px #00adff;
}

.organizeList_face::before {
  line-height: 100px;
}

.organizeList_face + span {
  position: absolute;
  font-size: 20px;
  right: 0;
  top: -10px;
  cursor: pointer;
  color: red;
}

.organizeList_name {
  text-align: center;
  font-size: 18px;
  float: left;
  margin-left: 20px;
}
</style>
