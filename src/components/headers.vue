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
          <li @click="openPets">真灵</li>
        </ul>
      </div>
      <div class="heade-right">
        <ul>
          <li @click="out">退出登录</li>
        </ul>
      </div>

      <!-- 真灵 -->
      <Modal
        width="720"
        title="真灵"
        v-model="PetsEject"
        :styles="{ top: '100px' }"
      >
        <Tabs v-if="PetsList" :value="Pets" @click.native="cs">
          <TabPane
            v-for="item in PetsList"
            :key="item.auraId"
            :label="
              item.auraStatus == 1 ? item.auraName + '(出战中）' : item.auraName
            "
            :name="item.auraId"
          >
            <div class="PetsListC">
              <img
                :src="item.auraIcon"
                alt="没图片啊 找策划加"
                style="width: 250px; height: 150px"
              />
              <div>
                <!-- <span :style="{color: realmColorF()}">真灵名字:{{ item.auraName }}</span> -->
                <span :style="{ color: realmColorF() }">
                  真灵等级:{{ item.auraLevel }}
                </span>
                <span :style="{ color: realmColorF() }">
                  {{ item.auraExp }} / {{ item.auraUpgradeExp }}
                </span>
                <span :style="{ color: realmColorF() }">
                  状态:{{ PetsStatus(item.auraStatus) }}
                </span>
                <span :style="{ color: realmColorF() }">
                  品级:{{ PetsGrade(item.auraGrade) }}
                </span>
                <span :style="{ color: realmColorF() }">
                  资质:{{ item.auraAptitude }}
                </span>
                <span :style="{ color: realmColorF() }">
                  元素:{{ petsEl(item.auraElement) }}
                </span>
                <span :style="{ color: realmColorF() }">
                  类型:{{ petsType(item.auraType) }}
                </span>
                <span :style="{ color: realmColorF() }">
                  种族:{{ item.auraRace }}
                </span>
                <!-- <span :style="{ color: realmColorF() }">
                  饥饿值:{{ item.auraStarvationValue }}
                </span> -->
                <span :style="{ color: realmColorF() }" v-if="item.auraDesci">
                  描述:{{ item.auraDesci }}
                </span>
                <p>
                  <span :style="{ color: realmColorF() }">
                    灵巧:{{ item.auraDexterity || 0 }}
                  </span>
                  <span :style="{ color: realmColorF() }">
                    体格:{{ item.auraPhysique || 0 }}
                  </span>
                  <span :style="{ color: realmColorF() }">
                    灵力:{{ item.auraSpirit || 0 }}
                  </span>
                  <span :style="{ color: realmColorF() }">
                    攻击力:{{ item.auraAttack || 0 }}
                  </span>
                  <span :style="{ color: realmColorF() }">
                    防御力:{{ item.auraDefence || 0 }}
                  </span>
                  <span :style="{ color: realmColorF() }">
                    灵力攻击:{{ item.auraMagicAttack || 0 }}
                  </span>
                  <span :style="{ color: realmColorF() }">
                    生命值:{{ item.auraLife || 0 }}
                  </span>
                  <span :style="{ color: realmColorF() }">
                    真气值:{{ item.auraMana || 0 }}
                  </span>
                  <span :style="{ color: realmColorF() }">
                    命中率:{{  (item.auraHitRate * 100).toFixed(1) || 0 }}
                  </span>
                  <span :style="{ color: realmColorF() }">
                    闪避率:{{ (item.auraEvade * 100).toFixed(1) || 0 }}
                  </span>
                  <span :style="{ color: realmColorF() }">
                    暴击:{{  (item.auraExplode * 100).toFixed(1) || 0 }} 
                  </span>
                  <span :style="{ color: realmColorF() }">
                    暴击伤害:{{ (item.auraCalDamage * 100).toFixed(1) || 0  }} 
                  </span>
                  <span :style="{ color: realmColorF() }">
                    速度:{{ item.auraActionSpeed || 0 }}
                  </span>
                </p>
              </div>
            </div>
            
            <div class="petsbtn">
              <Button type="error" size="small" @click="Release(item)">
                放生
              </Button>
              <Button type="primary" size="small" @click="goToWar(item)">
                出战/休息
              </Button>
            </div>
          </TabPane>
        </Tabs>
        <div slot="footer">
          <Button type="dashed" size="small" @click="PetsEject = false">
            取消
          </Button>
        </div>
      </Modal>

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
        <updateLog></updateLog>
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
import updateLog from "./updateLog.vue";
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
      PetsEject: false, // 真灵弹出框
      PetsList: [],
      Pets: null,
    };
  },
  created() {
    this.getPackageBo(); //自动出售设置的获取
    this.armyBoundary();
    this.TeamMapList(); // 获取组队地图列表
  },
  components: { playersList, rankingList, updateLog },
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

    // 灵宠状态
    PetsStatus(val) {
      if (val == 0) {
        return "休息";
      } else {
        return "出战";
      }
    },
    // 品级
    PetsGrade(val) {
      if (val == 1) {
        return "普通";
      } else if (val == 2) {
        return "灵级";
      } else if (val == 3) {
        return "圣级";
      } else {
        return "神级";
      }
    },
    // 元素状态
    petsEl(val) {
      if (val == 1) {
        return "金";
      } else if (val == 2) {
        return "木";
      } else if (val == 3) {
        return "水";
      } else if (val == 4) {
        return "火";
      } else if (val == 5) {
        return "土";
      } else if (val == 6) {
        return "风";
      } else if (val == 7) {
        return "雷";
      } else {
        return "无";
      }
    },
    // 类型
    petsType(val) {
      if (val == 1) {
        return "五行系";
      } else if (val == 2) {
        return "风雷系";
      } else if (val == 3) {
        return "防御型";
      } else {
        return "无";
      }
    },
    realmColorF() {
      var roundColor = Math.round(Math.random() * 5);
      if (roundColor == 1) {
        return "red";
      } else if (roundColor == 2) {
        return "skyblue";
      } else if (roundColor == 3) {
        return "#2700ff";
      } else if (roundColor == 4) {
        return "#d800ff";
      } else if (roundColor == 5) {
        return "#06c11d";
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

    // 打开真灵
    openPets() {
      this.PetsEject = true;
      this.$http
        .post("/gameAura/getCharaPet?charaId=" + this.getCookie("charaId"))
        .then((res) => {
          this.PetsList = res.data.data;
        })
        .catch((err) => {
          this.$Message.warning("真灵获取失败,请联系管理员");
        });
    },

    // 真灵放生
    goToWar(item) {
      this.$Message.warning("放生功能暂时没用,请联系管理员");
    },
    // 真灵出战
    goToWar(item) {
      let auraStatus = item.auraStatus == 0 ? 1 : 0
      this.$http
        .post("/gameAura/goToWarByAure?charaId=" + this.getCookie("charaId") + "&petId=" + item.auraId + "&aureStatus=" + auraStatus)
        .then((res) => {
          this.$Message.success(res.data.msg);
          this.openPets()
        })
        .catch((err) => {
          this.$Message.warning("真灵出战失败,请联系管理员");
        });
    },

    cs() {},

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

.importantFontRed {
  color: #ff004c;
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

/* 真灵列表  加c是class的意思 */
.PetsListC {
  overflow: hidden;
}
.PetsListC > img {
  float: left;
}
.PetsListC > div {
  float: left;
  margin-left: 15px;
  font-size: 15px;
  width: 422px;
}

.PetsListC > div > span {
  padding: 0 5px;
}

.PetsListC > div p {
  margin-top: 15px;
}

.PetsListC > div p span {
  display: inline-block;
  width: 102px;
}
.petsbtn {
  margin-top: 15px;
}
</style>
