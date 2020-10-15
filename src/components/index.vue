<template>
  <div id="app2">
    <vue-canvas-nest
      :config="{ color: '255,255,255', count: 100 }"
    ></vue-canvas-nest>
    <headers></headers>
    <div id="box">
      <div id="left">
        <Card>
          <p slot="title">
            <!-- 用户名字 暂时还没写修改的 函数名editName -->
            {{ user.name }}
          </p>
          <Button
            slot="extra"
            @click.prevent="editName"
            size="small"
            type="info"
            >修改名字</Button
          >
          <div>
            <div class="character">
              <Avatar src="https://i.loli.net/2017/08/21/599a521472424.jpg" />
              <div class="rank">
                <span>LV{{ user.level }}</span>
                <div class="bloodVolume">生命: {{ user.health }}</div>
                <div class="blueAmount">法力值: {{ user.mana }}</div>
              </div>
            </div>
            <div class="goldCoin">
              <p>当前经验: {{ user.exp }} / {{ user.upgradeExp }}</p>
              <p>当前铜钱: {{ user.money }}</p>
              <!-- 铜钱买普通物品 -->
              <p>当前金币: {{ user.coin }}</p>
              <!-- 金币升技能和买高级物品 -->
            </div>
          </div>
        </Card>

        <!-- 修改名字弹出框 -->
        <Modal
          title="修改名字"
          v-model="nameEject"
          @on-ok="deteSetName"
          :styles="{ top: '100px' }"
        >
          <Input v-model="userName" placeholder="请输入名字" />
        </Modal>

        <!-- 地图和战斗场景 -->
        <Card>
          <p slot="title">
            {{ mapNameF }}
          </p>
          <Button
            slot="extra"
            @click.prevent="changeMap"
            size="small"
            type="info"
            >切换地图</Button
          >
          <div>
            <div
              class="monster"
              :class="[monsterList.length == 1 ? 'monstercenter' : '']"
            >
              <Poptip
                trigger="hover"
                v-for="item in monsterList"
                :key="item.id"
                :title="item.name"
              >
                ({{ item.prefix || "无" }}){{ item.name }}
                <div slot="content">
                  <p>生命值 {{ item.life }}</p>
                  <p>最小攻击力 {{ item.attackMin }}</p>
                  <p>最大攻击力 {{ item.attackMax }}</p>
                  <p>防御力 {{ item.defence }}</p>
                  <p>暴击率 {{ item.explode * 100 }}%</p>
                  <p>暴击伤害 150%</p>
                  <p>行动速度 {{ item.speed }}</p>
                  <!-- <p>掉落物品 {{ item.wp ||  '无'}}</p> -->
                </div>
              </Poptip>
            </div>
            <div class="recording" id="recording">
              <h6 v-if="monsterList.length <= 0">暂无记录</h6>
              <p v-for="(item, index) in recording" :key="index">
                <span v-if="item.type == 'combat'">
                  <span style="color: #5b00ff">{{ item.attacker }}</span>
                  攻击了<span style="color: red">{{ item.hinjured }}</span>
                  <span v-if="item.isSkill != 1">造成了<span :style="{color: '#5b00ff',fontSize:(item.isCritical == 0 ? '14px' : '20px')}">{{ item.hurt }}</span></span>
                  <span v-else>用{{ item.skillName }}造成了<span :style="{color: 'rgb(110 0 255)'}">{{ item.hurt }}</span></span>
                  剩余<span style="color: red">{{ item.surplusHealth }}</span
                  >血量
                </span>
                <span v-if="item.type == 'result'"
                  >{{ item.result }} {{ item.exp }} {{ item.goods }}</span
                >
              </p>
              <!-- //     newChild.innerHTML = `${combatInfo[i].attacker}攻击了${combatInfo[i].hinjured} 造成了${combatInfo[i].hurt}剩余${combatInfo[i].surplusHealth}` -->
            </div>
          </div>
        </Card>

        <!-- 切换地图弹出框 -->
        <Modal
          title="选择地图"
          v-model="mapEject"
          @on-ok="deteSetMap"
          :styles="{ top: '100px' }"
        >
          <Select v-model="mapName" filterable>
            <!-- ({{ item.minLv }}) -->
            <Option
              v-for="item in mapList"
              :value="item.mapName"
              :name="item.mapId"
              :key="item.mapId"
              >{{ item.mapName }}</Option
            >
          </Select>
        </Modal>
      </div>

      <div id="center">
        <!-- 属性 和 属性加点位置 -->
        <Card>
          <p slot="title">
            属性:
            <span
              style="
                border-radius: 50%;
                background: #d9534f;
                padding: 1px 5px;
                color: #fff;
                font-size: 14px;
              "
              >{{ user.reAttrPoint }}</span
            >
          </p>
          <!-- 这个确认修改不确定要不要 我想的是买药水然后使用修改 -->
          <Button slot="extra" size="small" @click="setAttribute" type="info"
            >确认修改</Button
          >
          <div style="overflow: hidden">
            <div class="basics">
              <h6>基础</h6>
              <p>
                <Tooltip placement="top">
                  体格:
                  <span>{{ user.physique }}</span>
                  <div slot="content">
                    <p>体格影响生命值和防御力</p>
                  </div>
                </Tooltip>
                <button @click="addAttributes(0, 1)">+1</button>
                <button @click="addAttributes(0, 10)">+10</button>
              </p>
              <p>
                <Tooltip placement="top">
                  灵巧:
                  <span>{{ user.dexterous }}</span>
                  <div slot="content">
                    <p>灵巧影响物理攻击</p>
                  </div>
                </Tooltip>
                <button @click="addAttributes(1, 1)">+1</button>
                <button @click="addAttributes(1, 10)">+10</button>
              </p>
              <p>
                <Tooltip placement="top">
                  精神:
                  <span>{{ user.spirit }}</span>
                  <div slot="content">
                    <p>精神影响法力值和法术攻击</p>
                  </div>
                </Tooltip>
                <button @click="addAttributes(2, 1)">+1</button>
                <button @click="addAttributes(2, 10)">+10</button>
              </p>
            </div>
            <div class="basics">
              <h6>全部</h6>
              <p>
                物理攻击力:
                <span>{{ user.physicalAttack }}</span>
              </p>
              <p>
                法术攻击力:
                <span>{{ user.magicAttack }}</span>
              </p>
              <p>
                防御力:
                <span>{{ user.defense }}</span>
              </p>
              <p>
                暴击率:
                <span>{{ user.criticalHitValue*100 || 0 }}%</span>
              </p>
              <p>
                暴击伤害:
                <span>150%</span>
              </p>
              <p>
                行动速度:
                <span>{{ user.movingSpeed }}</span>
              </p>
              <p>
                幸运值:
                <span>{{ user.lucky || 0 }}</span>
              </p>
            </div>
          </div>
        </Card>

        <!-- 背包 -->
        <Card>
          <p slot="title">背包</p>
          <div slot="extra">
            <span>背包容量{{ knapsackList.length }} / 50</span>
            <Button @click.prevent="sellAll" size="small" type="info"
              >出售全部</Button
            >
          </div>
          <div>
            <div class="knapsack">
              <Poptip
                trigger="hover"
                v-for="item in knapsackList"
                :key="item.name"
                :title="item.name"
              >
                <p
                  style="white-space: nowrap; height: 24px"
                  :style="{ color: item.color }"
                >
                  {{ item.itemName }}
                  <Button
                    @click.prevent="equipment(item)"
                    size="small"
                    type="info"
                    >装备</Button
                  >
                  <Button @click.prevent="strengthen" size="small" type="info"
                    >强化</Button
                  >
                  <Button
                    @click.prevent="sellSingle(item)"
                    size="small"
                    type="info"
                    >出售</Button
                  >
                </p>
                <div slot="content">
                  <p>{{ item.itemName }}</p>
                  <p>装备类型 {{ item.typeDec }}</p>
                  <p>装备等级 {{ item.level }}</p>
                  <p v-if="item.life">生命值 {{ item.life }}</p>
                  <p v-if="item.attack">攻击力 {{ item.attack }}</p>
                  <p v-if="item.defence">防御力 {{ item.defence }}</p>
                  <p v-if="item.critical">暴击率 {{ item.critical }}%</p>
                  <p v-if="item.speed">行动速度 {{ item.speed }}</p>
                  <p v-if="item.physique">体格 {{ item.physique }}</p>
                  <p v-if="item.dropProb">灵巧 {{ item.dropProb }}</p>
                  <p v-if="item.spirit">精神 {{ item.spirit }}</p>
                </div>
              </Poptip>
            </div>
          </div>
        </Card>
      </div>

      <div id="right">
        <!-- 装备 -->
        <Card>
          <p slot="title">装备</p>
          <div slot="extra"></div>
          <div>
            <div v-for="(item, key, index) in equipmentList" :key="index">
              <span>{{ key }}:</span>
              <Poptip trigger="hover" :title="item.equitName" v-if="item">
                <p
                  style="white-space: nowrap; height: 24px"
                  :style="{ color: item.color }"
                >
                  {{ item.equitName }}
                </p>
                <div slot="content">
                  <p v-if="item.typeDec">装备类型 {{ item.typeDec }}</p>
                  <p v-if="item.level">装备等级 {{ item.level }}</p>
                  <p v-if="item.life">生命值 {{ item.life }}</p>
                  <p v-if="item.attack">攻击力 {{ item.attack }}</p>
                  <p v-if="item.defence">防御力 {{ item.defence }}</p>
                  <p v-if="item.critical">暴击率 {{ item.critical }}%</p>
                  <p v-if="item.speed">行动速度 {{ item.speed }}</p>
                  <p v-if="item.physique">体格 {{ item.physique }}</p>
                  <p v-if="item.dropProb">灵巧 {{ item.dropProb }}</p>
                  <p v-if="item.spirit">精神 {{ item.spirit }}</p>
                </div>
              </Poptip>
            </div>
          </div>
        </Card>

        <!-- 技能 -->
        <Card>
          <p slot="title">技能</p>
          <div slot="extra"></div>
          <div>
            <div style="overflow: hidden;margin-bottom: 15px">
              <div class="activeSkill">
                <h4>主动技能</h4>
                <div v-for="(item, index) in activeList" :key="index" style="margin-bottom: 4px;">
                  <!-- <span>{{ item.skillName }}:</span> -->
                  <Poptip trigger="hover" :title="item.skillName">
                    <p
                      style="white-space: nowrap; height: 24px"
                    > 
                      <span>{{ item.skillName }}:</span>
                      <Button @click.prevent="TakeSkill(item,1)" size="small" type="info"
                        >脱下</Button
                      >
                    </p>
                    <div slot="content">
                      <p>技能效果: {{ item.skillDesc }}</p>
                      <p>升级所需金币 {{ item.consumeCoin }}</p>
                    </div>
                  </Poptip>
                </div>
              </div>
              <div class="passiveSkill">
                <h4>被动技能</h4>
                <div v-for="(item, index) in passiveList" :key="index" style="margin-bottom: 4px;">
                  <Poptip trigger="hover" :title="item.skillName">
                    <p
                      style="white-space: nowrap; height: 24px"
                    > 
                      <span>{{ item.skillName }}:</span>
                      <Button @click.prevent="TakeSkill(item,2)" size="small" type="info"
                        >脱下</Button
                      >
                    </p>
                    <div slot="content">
                      <p>技能效果: {{ item.skillDesc }}</p>
                      <p>升级所需金币 {{ item.consumeCoin }}</p>
                    </div>
                  </Poptip>
                </div>
              </div>
            </div>
            <div class="skillList">
              <Poptip
                trigger="hover"
                v-for="item in skillList"
                :key="item.skillId"
                :title="item.skillName"
              >
                <p
                  style="white-space: nowrap; height: 24px"
                  :style="{ color: item.color }"
                >
                  {{ item.skillName }}
                  <Button
                    @click.prevent="skillEquip(item)"
                    size="small"
                    type="info"
                    >装备</Button
                  >
                  <Button @click.prevent="strengthen" size="small" type="info"
                    >升级</Button
                  >
                </p>
                <div slot="content">
                  <!-- <p>{{ item.skillName }}</p> -->
                  <p>技能效果: {{ item.skillDesc }}</p>
                  <p>升级所需金币 {{ item.consumeCoin }}</p>
                </div>
              </Poptip>
            </div>
          </div>
        </Card>
      </div>
    </div>
  </div>
</template>

<script>
import vueCanvasNest from "vue-canvas-nest";
import headers from "./headers.vue";
import { EquipSlot, SlotName } from "../const";

export default {
  name: "index",
  data() {
    return {
      EquipSlot,
      SlotName,
      userName: "",
      nameEject: false,
      mapName: "桑林村", // 下拉的地图名字
      mapNameF: "桑林村", // 显示的地图名字
      mapEject: false,
      user: {}, // 用户的全部消息
      mapList: [], // 地图列表
      monsterList: [], //怪物信息
      recording: [], // 战斗信息
      battleState: false, // 战斗状态
      time: null, // 暂时没用到
      battleTimer: null,
      knapsackList: [], // 包裹
      mapid: "", // 地图id
      activeList: [], // 穿戴的主动技能列表
      passiveList: [], // 穿戴的被动技能列表
      skillList: [], // 全部技能
      equipmentList: {} //装备列表
    };
  },
  components: { vueCanvasNest,headers },
  updated() {
    // 战斗记录定位到底部
    let ele = document.getElementById("recording");
    ele.scrollTop = ele.scrollHeight;
  },
  mounted() {
    // 如果没有用户id就退出登录
    if (!this.getCookie("userId")) {
      this.$Message.warning("请重新登录");
      this.$router.push({ name: "Login" });
      return;
    }

    // 如果没有角色id就退出登录
    if (!this.getCookie("charaId")) {
      this.$Message.warning("请重新登录");
      this.$router.push({ name: "Login" });
      return;
    }

    // 获取地图列表
    this.$http
      .get(
        "http://www.yunyingxiaowu.com:8088/foodie-api/gameChara/queryMapList"
      )
      .then(res => {
        this.mapList = res.data.data;
      });

    this.getAllUser(); // 获取用户
    this.getAllPackage(); // 获取全部包裹
    this.getAliiEquip(); // 获取穿戴的装备列表
    this.getAllSkill() // 获取技能列表
    this.getEquipmentSkill() // 获取已装备的技能列表
  },
  methods: {
    // 返回数据处理的装备列表
    getEquipMap(player) {
      const equips = {};
      Object.values(this.EquipSlot).map(slotId => {
        equips[this.SlotName[slotId]] = player.find(
          item => item.geKind === slotId
        );
      });
      return equips;
    },

    // 获取角色的装备列表
    getAliiEquip() {
      this.$http
        .post(
          "http://www.yunyingxiaowu.com:8088/foodie-api/gameCharaEquip/getCharaEquip?charaId=" +
            this.getCookie("charaId")
        )
        .then(res => {
          this.equipmentList = this.getEquipMap(res.data.data);
        })
        .catch(err => {
          this.$Message.warning("获取装备列表失败,请联系管理员");
        });
    },

    // 获取角色的全部消息
    getAllUser() {
      this.$http
        .get(
          "http://www.yunyingxiaowu.com:8088/foodie-api/gamepassport/getGameCharacter?charaId=" +
            this.getCookie("charaId")
        )
        .then(res => {
          this.user = res.data.data;
          this.user.charaId = this.getCookie("charaId");
        })
        .catch(err => {
          this.$Message.warning("获取角色失败,请联系管理员");
        });
    },

    // 获取角色的j技能列表
    getAllSkill() {
      this.$http
        .post(
          "http://www.yunyingxiaowu.com:8088/foodie-api/gameCharaSkill/getCharaSkill/?charaId=" +
            this.getCookie("charaId")
        )
        .then(res => {
          this.skillList = res.data.data
        })
        .catch(err => {
          this.$Message.warning("获取技能列表失败,请联系管理员");
        });
    },

    // 获取角色已装备的技能列表
    getEquipmentSkill() {
      this.$http
        .post(
          "http://www.yunyingxiaowu.com:8088/foodie-api/gameCharaSkill/getCharaUseSkill/?charaId=" +
            this.getCookie("charaId") +
            "&skillType=" +
            1
        )
        .then(res => {
          console.log(res.data.data,"主动")
          this.activeList = res.data.data
        })
        .catch(err => {
          this.$Message.warning("获取已装备的主动技能列表失败,请联系管理员");
        });

      this.$http
        .post(
          "http://www.yunyingxiaowu.com:8088/foodie-api/gameCharaSkill/getCharaUseSkill/?charaId=" +
            this.getCookie("charaId") +
            "&skillType=" +
            2
        )
        .then(res => {
          console.log(res.data.data,"被动")
          this.passiveList = res.data.data
        })
        .catch(err => {
          this.$Message.warning("获取已装备的被动技能列表失败,请联系管理员");
        });
    },

    // 装备技能
    skillEquip (item) {
      this.$http
        .post(
          "http://www.yunyingxiaowu.com:8088/foodie-api/gameCharaSkill/makeSkill/?charaId=" +
            this.getCookie("charaId") +
            "&skillId=" +
            item.skillId+
            "&skillType=" +
            item.skillType
        )
        .then(res => {
          this.$Message.warning("装备技能成功");
          this.getAllUser();
          this.getAllSkill();
          this.getEquipmentSkill()
        })
        .catch(err => {
          this.$Message.warning("装备技能失败,请联系管理员");
        });
    },

    // 脱下技能
    TakeSkill(item) {
      this.$http
        .post(
          "http://www.yunyingxiaowu.com:8088/foodie-api/gameCharaSkill/makeDownSkill/?charaId=" +
            this.getCookie("charaId") +
            "&skillId=" +
            item.skillId+
            "&skillType=" +
            item.skillType
        )
        .then(res => {
          this.$Message.warning("装备脱下成功");
          this.getAllUser();
          this.getAllSkill();
          this.getEquipmentSkill()
        })
        .catch(err => {
          this.$Message.warning("装备脱下失败,请联系管理员");
        });
    },

    // 获取用户全部包裹
    getAllPackage() {
      this.$http
        .post(
          "http://www.yunyingxiaowu.com:8088/foodie-api/gameChara/getCharaPackage?charaId=" +
            this.getCookie("charaId")
        )
        .then(res => {
          this.knapsackList = res.data.data;
        })
        .catch(err => {
          this.$Message.warning("获取包裹失败,请联系管理员");
        });
    },

    // 修改名字的按钮
    editName() {
      this.userName = "";
      this.nameEject = true;
    },

    // 修改名字弹出框的确定
    deteSetName() {
      this.$http
        .post(
          "http://www.yunyingxiaowu.com:8088/foodie-api/gamepassport/updataGameName?charaId=" +
            this.getCookie("charaId") +
            "&name=" +
            this.userName
        )
        .then(res => {
          this.$Message.success("修改成功,请不要联系管理员");
        })
        .catch(err => {
          this.$Message.warning("修改名字失败,请联系管理员");
        });

      this.user.name = this.userName;
    },

    // 切换地图的按钮
    changeMap() {
      this.mapEject = true;
    },

    // 切换地图弹出框的确定
    deteSetMap(mapid) {
      if (this.battleState) {
        this.$Message.warning("还在战斗状态 不能切换");
        return;
      }

      clearTimeout(this.battleTimer);
      this.battleTimer = null;

      this.mapNameF = this.mapName;

      this.mapid = mapid;

      for (var i = 0; i < this.mapList.length; i++) {
        if (this.mapName == this.mapList[i].mapName) {
          this.mapid = this.mapList[i].mapId;
        }
      }

      this.$http
        .post(
          "http://www.yunyingxiaowu.com:8088/foodie-api/gameChara/checkMapGenMon?charaId=" +
            this.getCookie("charaId") +
            "&mapId=" +
            this.mapid
        )
        .then(res => {
          // console.log(res.data.data.dropExp) //本次战斗总掉落经验
          this.monsterList = res.data.data.gameMonList; // 怪物信息
          // console.log(res.data.data.combatInfo) // 战斗信息

          let combatInfo = res.data.data.combatInfo; // 战斗信息

          combatInfo.forEach((val, idx) => {
            this.time = setTimeout(() => {
              val.type = "combat";
              this.recording.push(val);

              this.battleState = true;

              if (idx == combatInfo.length - 1) {
                if (res.data.data.reHealth <= 0) {
                  this.recording.push({
                    type: "result",
                    result: "战斗失败"
                  });

                  this.battleState = false;
                } else {
                  let goods = res.data.data.gameItemsList.map(val => {
                    return val.itemName;
                  });
                  goods = goods.join();

                  this.recording.push({
                    type: "result",
                    result: "战斗成功",
                    exp: "获得" + res.data.data.dropExp + "经验;",
                    goods: "物品:" + (goods || "无")
                  });

                  let exp = (this.user.exp += res.data.data.dropExp);
                  if (exp >= this.user.upgradeExp) {
                    this.getAllUser();
                  }

                  // 背包超过50个以上 不更新背包接口
                  if (this.knapsackList.length >= 50) {
                    this.$Message.warning("背包已满");
                  } else {
                    this.getAllPackage();
                  }

                  this.battleState = false;
                }

                // 战斗记录超过50条 清空
                if (this.recording.length >= 50) {
                  this.recording = [];
                }

                this.battleTimer = setTimeout(() => {
                  this.battleState = false;
                  this.deteSetMap(mapid);
                }, 3000);
              }
            }, idx * 1000);
          });
        })
        .catch(err => {
          this.$Message.warning("战斗开始失败,请联系管理员");
        });
    },

    // 确认修改属性的按钮
    setAttribute() {
      this.user.charaId = this.getCookie("charaId");
      this.$http
        .post(
          "http://www.yunyingxiaowu.com:8088/foodie-api/gameChara/updateAttrPoint",
          this.user
        )
        .then(res => {
          this.$Message.warning("修改成功");
          this.getAllUser();
        })
        .catch(err => {
          this.$Message.warning("修改失败,请联系管理员");
        });
    },

    // 属性加点
    addAttributes(val, spot) {
      if (this.user.reAttrPoint <= 0) {
        this.$Message.warning("没有属性点可加了");
        return;
      }

      if (this.user.reAttrPoint < spot) {
        this.$Message.warning("属性点不够了凹");
        return;
      }

      if (val == 0) {
        this.user.physique += spot;
      } else if (val == 1) {
        this.user.dexterous += spot;
      } else if (val == 2) {
        this.user.spirit += spot;
      }
      this.user.reAttrPoint -= spot;
    },

    // 出售全部装备
    sellAll() {
      let arrKnapsac = [];
      this.knapsackList.forEach(val => {
        arrKnapsac.push(val.packItemId);
      });
      this.$http
        .post(
          "http://www.yunyingxiaowu.com:8088/foodie-api/gameCharaEquip/oneClickSale?charaId=" +
            this.getCookie("charaId") +
            "&packItemIds=" +
            arrKnapsac
        )
        .then(res => {
          this.getAllUser();
          this.getAllPackage();
        })
        .catch(err => {
          this.$Message.warning("出售物品失败,请联系管理员");
        });
    },

    // 出售单件装备
    sellSingle(item) {
      this.$http
        .post(
          "http://www.yunyingxiaowu.com:8088/foodie-api/gameCharaEquip/oneClickSale?charaId=" +
            this.getCookie("charaId") +
            "&packItemIds=" +
            item.packItemId
        )
        .then(res => {
          this.getAllUser();
          this.getAllPackage();
        })
        .catch(err => {
          this.$Message.warning("出售物品失败,请联系管理员");
        });
    },

    // 装备界面上的装备按钮
    equipment(item) {
      this.$http
        .post(
          "http://www.yunyingxiaowu.com:8088/foodie-api/gameCharaEquip/useEquitBypackage?charaId=" +
            this.getCookie("charaId") +
            "&packItemId=" +
            item.packItemId
        )
        .then(res => {
          this.$Message.warning(res.data.data);
          this.getAliiEquip();
          this.getAllPackage();
          this.getAllUser();
        })
        .catch(err => {
          this.$Message.warning("装备失败,请联系管理员");
        });
    }
  }
};
</script>
<style>
.vue-canvas-nest-element {
  opacity: 0.9 !important;
}
.vue-canvas-nest-element canvas {
  background: rgba(0, 0, 0, 0.9);
  opacity: 1 !important;
}
.ivu-card {
  background: rgba(255, 255, 255, 1);
}
.ivu-card-head {
  border-bottom: 1px solid #abb6c0;
}
/* 为了头像宽度 */
.ivu-card-body .ivu-avatar {
  width: 100px;
  height: 100px;
  float: left;
}
/* 为了每一个卡片都往下15px */
.ivu-card-bordered {
  margin-bottom: 15px;
}
</style>
<style scoped>
/* 全局样式 */
#app2 {
  min-width: 1140px;
}
#box {
  width: 1140px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
}
#left,
#center,
#right {
  width: 374px;
  /* background-color: skyblue; */
}

/* 第一块位置 用户头像那一块*/
.character {
  overflow: hidden;
}
.rank {
  float: left;
  margin-left: 10px;
}
.rank > span {
  background-color: #a1afb4;
  color: #fff;
  padding: 1px 3px;
  border-radius: 4px;
}
.bloodVolume,
.blueAmount {
  width: 230px;
  height: 22px;
  background: red;
  border-radius: 5px;
  color: #fff;
  font-size: 12px;
  line-height: 22px;
  text-align: center;
}
.bloodVolume {
  margin-top: 10px;
  margin-bottom: 10px;
}
.blueAmount {
  background: #4850b8;
}
.goldCoin {
  border-top: 1px solid #999;
  margin-top: 22px;
  padding-top: 18px;
}
.goldCoin p {
  margin-top: 2px;
}

/* 地图和战斗场景 */
.monster {
  display: flex;
  justify-content: space-between;
  margin-bottom: 40px;
}
.monstercenter {
  justify-content: center;
}

/* 为了给提示设置 颜色 */
/deep/ .ivu-poptip-popper .ivu-poptip-content .ivu-poptip-inner {
  background-color: rgba(70, 76, 91, 0.9);
  box-shadow: 0 1px 6px rgba(0, 0, 0, 0.2);
}
/deep/ .ivu-poptip-popper .ivu-poptip-content .ivu-poptip-arrow:after {
  border-top-color: rgba(70, 76, 91, 0.9);
}
/deep/ .ivu-poptip-popper .ivu-poptip-content .ivu-poptip-title-inner {
  color: #fff;
}
/deep/ .ivu-poptip-popper .ivu-poptip-content .ivu-poptip-body {
  color: #fff;
}

.recording {
  max-height: 300px;
  overflow: auto;
  transition: all .4s;
}

/* 第二块位置 基础属性 和 加点的位置*/
.basics {
  width: 50%;
  float: left;
}
.basics h6 {
  background-color: #a1afb4;
  color: #fff;
  padding: 1px 3px;
  border-radius: 4px;
  display: inline-block;
  margin-bottom: 12px;
}
.basics p {
  margin-bottom: 4px;
}
.basics p button {
  background-color: #d9534f;
  color: #fff;
  padding: 1px 4px;
  border-radius: 4px;
  border: 0;
  outline: none;
  font-size: 12px;
  margin-right: 3px;
  margin-left: 1px;
  cursor: pointer;
}

/* knapsack 背包 */
.knapsack {
  /* max-height: 516px; */
}
.knapsack /deep/ .ivu-poptip {
  width: 50%;
}
.knapsack /deep/ .ivu-poptip p button {
  display: none;
  height: 22px;
}
.knapsack /deep/ .ivu-poptip p:hover {
  z-index: 9;
  position: relative;
}
.knapsack /deep/ .ivu-poptip p:hover button {
  display: inline-block;
}

/* 技能列表 */
.skillList /deep/ .ivu-poptip {
  width: 50%;
}
.skillList /deep/ .ivu-poptip p button {
  display: none;
  height: 22px;
}
.skillList /deep/ .ivu-poptip p:hover {
  z-index: 9;
  position: relative;
}
.skillList /deep/ .ivu-poptip p:hover button {
  display: inline-block;
}
/* 技能栏 */
.activeSkill {
  float: left;
  width: 50%;
  text-align: center;
}
.passiveSkill {
  float: right;
  width: 50%;
  text-align: center;
}
</style>
