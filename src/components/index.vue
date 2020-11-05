<template>
  <div
    id="app2"
    :style="{ background: this.electronicEquipment == false ? '#666' : '' }"
  >
    <vue-canvas-nest
      v-if="this.electronicEquipment == true"
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
                <div class="blueAmount">真气值: {{ user.mana }}</div>
              </div>
            </div>
            <div class="goldCoin">
              <p>当前经验: {{ user.exp }} / {{ user.upgradeExp }}</p>
              <!-- 铜钱买普通物品 -->
              <p>
                当前铜钱: {{ user.money }}
                <!-- 一键兑换金币 -->
                <!-- 暂时关闭 -->
                <!-- <Button
                  slot="extra"
                  changeMap
                  @click.prevent="exchangeCoin"
                  size="small"
                  type="info"
                  >兑换金币</Button
                > -->
              </p>
              <!-- 金币升技能和买高级物品 -->
              <p>当前金币: {{ user.coin }}</p>
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
                  灵力:
                  <span>{{ user.spirit }}</span>
                  <div slot="content">
                    <p>灵影响真气值和法术攻击</p>
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
                <span
                  >{{ (user.criticalHitValue * 100).toFixed(2) || 0 }}%</span
                >
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

        <!-- 装备 -->
        <Card>
          <p slot="title">装备</p>
          <div slot="extra">
            <Button
              slot="extra"
              @click.prevent="allTake"
              size="small"
              type="info"
              >一键脱下</Button
            >
          </div>
          <div>
            <div v-for="(item, key, index) in equipmentList" :key="index">
              <span>{{ key }}:</span>
              <Poptip
                trigger="hover"
                :style="{ color: distinguishColor(item.color) }"
                v-if="item"
              >
                <p
                  style="white-space: nowrap; height: 24px"
                  :style="{ color: distinguishColor(item.color) }"
                >
                  {{ item.equitName }}
                  <span
                    v-if="
                      item.itemType != 3 &&
                      item.itemType != 4 &&
                      item.enhanLevel != 0
                    "
                    >+{{ item.enhanLevel }}</span
                  >
                  <Button
                    @click.prevent="TakeOffsingleton(item)"
                    size="small"
                    type="info"
                    >脱下</Button
                  >
                </p>
                <div slot="content">
                  <p>
                    {{ item.equitName }}
                    <span v-if="item.itemType != 3 && item.itemType != 4"
                      >+{{ item.enhanLevel }}</span
                    >
                  </p>
                  <p v-if="item.typeDec">装备类型 {{ item.typeDec }}</p>
                  <p v-if="item.level">装备等级 {{ item.level }}</p>
                  <p>物品状态 {{ item.bind == 0 ? "解绑" : "绑定" }}</p>
                  <p v-if="item.life">
                    生命值 {{ item.life }}
                    <span v-if="item.strengLife">({{ item.strengLife }})</span>
                  </p>
                  <p v-if="item.mana">
                    真气值 {{ item.mana }}
                    <span v-if="item.strengMana">({{ item.strengMana }})</span>
                  </p>
                  <p v-if="item.attack">
                    攻击力 {{ item.attack }}
                    <span v-if="item.strengAttack"
                      >({{ item.strengAttack }})</span
                    >
                  </p>
                  <p v-if="item.magAttack">
                    法术攻击力 {{ item.magAttack }}
                    <span v-if="item.strengMagAttack"
                      >({{ item.strengMagAttack }})</span
                    >
                  </p>
                  <p v-if="item.defense">
                    防御力 {{ item.defense }}
                    <span v-if="item.strengDefense"
                      >({{ item.strengDefense }})</span
                    >
                  </p>
                  <p v-if="item.critical">
                    暴击率 {{ (item.critical * 100).toFixed(2) }}%
                  </p>
                  <p v-if="item.speed">行动速度 {{ item.speed }}</p>
                  <p v-if="item.physique">
                    体格 {{ item.physique }}
                    <span v-if="item.strengPhysique"
                      >({{ item.strengPhysique }})</span
                    >
                  </p>
                  <p v-if="item.dexterous">
                    灵巧 {{ item.dexterous }}
                    <span v-if="item.strengDexterous"
                      >({{ item.strengDexterous }})</span
                    >
                  </p>
                  <p v-if="item.spirit">
                    灵力 {{ item.spirit }}
                    <span v-if="item.strengSpirit"
                      >({{ item.strengSpirit }})</span
                    >
                  </p>
                  <p v-if="item.decs">描述： {{ item.decs }}</p>
                </div>
              </Poptip>
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
          <Select v-model="mapName">
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
        <!-- 地图和战斗场景 -->
        <Card class="two">
          <p slot="title">
            {{ mapNameF }}
          </p>

          <!-- changeMap -->
          <Button
            slot="extra"
            changeMap
            @click.prevent="deteSetMap"
            size="small"
            type="info"
            >确定</Button
          >
          <Select v-model="mapName">
            <!-- ({{ item.minLv }}) -->
            <Option
              v-for="item in mapList"
              :value="item.mapName"
              :name="item.mapId"
              :key="item.mapId"
              >{{ item.mapName }}{{ item.minLv }}-{{ item.maxLv }}</Option
            >
          </Select>
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
                ({{ item.prefix == 0 ? "无" : item.prefix }}){{ item.name }}
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
                  <span
                    :style="{
                      color: item.identity == 0 ? 'orange' : '#5b00ff',
                    }"
                    >{{ item.attacker }}</span
                  >
                  攻击了<span style="color: red">{{ item.hinjured }}</span>
                  <span v-if="item.isSkill != 1"
                    >造成了<span
                      :style="{
                        color: '#5b00ff',
                        fontSize: item.isCritical == 0 ? '14px' : '20px',
                      }"
                      >{{ item.hurt }}</span
                    ></span
                  >
                  <span v-else
                    >用{{ item.skillName }}造成了<span
                      :style="{ color: 'rgb(110 0 255)' }"
                      >{{ item.hurt }}</span
                    ></span
                  >
                  剩余<span style="color: red">{{ item.surplusHealth }}</span
                  >血量
                </span>
                <span
                  style="color: rgb(255, 0, 224)"
                  v-if="item.type == 'result'"
                  >{{ item.result }}: 剩余血量:{{ item.reHealth }} 剩余蓝量:
                  {{ item.reMana }} {{ item.exp }} {{ item.goods }}
                </span>
              </p>
              <!-- //     newChild.innerHTML = `${combatInfo[i].attacker}攻击了${combatInfo[i].hinjured} 造成了${combatInfo[i].hurt}剩余${combatInfo[i].surplusHealth}` -->
            </div>
          </div>
        </Card>

        <div class="line"></div>

        <!-- 强化提示框 -->
        <Modal
          v-model="strengthenEject"
          title="强化提示"
          @on-ok="changeStrengthen"
          :styles="{ top: '240px' }"
        >
          <p v-if="materialsRequired.materOneName">
            {{ materialsRequired.materOneName }}: 需要
            {{ materialsRequired.materOneNum }}
          </p>
          <p v-if="materialsRequired.materTwoName">
            {{ materialsRequired.materTwoName }}: 需要
            {{ materialsRequired.materTwoNum }}
          </p>
          <p v-if="materialsRequired.coin">
            需要 {{ materialsRequired.coninType == 0 ? "铜币" : "金币" }}
            {{ materialsRequired.coin }}
          </p>
        </Modal>

        <!-- 背包 -->
        <Card>
          <p slot="title">背包</p>
          <div slot="extra">
            <span
              >背包容量{{ knapsackList.length }} / {{ user.packageNum }}</span
            >
            <Button @click.prevent="sellAll" size="small" type="info"
              >出售全部装备</Button
            >
          </div>
          <div>
            <div>
              <p>物品分类</p>
              <RadioGroup v-model="classification">
                <Radio label="0">全部</Radio>
                <Radio label="1">装备</Radio>
                <Radio label="2">灵力武器</Radio>
                <Radio label="3">道具</Radio>
                <Radio label="4">材料</Radio>
              </RadioGroup>
              <p>装备品质</p>
              <RadioGroup v-model="quality">
                <Radio label="0">全部</Radio>
                <Radio label="1">黑</Radio>
                <Radio label="2">黄</Radio>
                <Radio label="3">绿</Radio>
                <Radio label="4">蓝</Radio>
                <Radio label="5">紫</Radio>
                <Radio label="6">红</Radio>
              </RadioGroup>
            </div>
            <br />
            <div class="knapsack">
              <Poptip
                trigger="hover"
                v-for="item in filteredItems"
                :key="item.name"
                :title="item.name"
              >
                <p
                  style="white-space: nowrap; height: 24px"
                  :style="{ color: distinguishColor(item.color) }"
                >
                  <span v-if="item.itemType != 3 && item.itemType != 4"
                    >[LV:{{ item.level }}]</span
                  >
                  {{ item.itemName }}
                  <span
                    v-if="
                      item.itemType != 3 &&
                      item.itemType != 4 &&
                      item.enhanLevel != 0
                    "
                    >+{{ item.enhanLevel }}</span
                  >
                  <span
                    class="jjt_smail"
                    v-if="item.itemType == 3 || item.itemType == 4"
                    >{{ item.itemNum }}</span
                  >
                  <Button
                    @click.prevent="useItems(item)"
                    size="small"
                    v-if="item.itemType == 3 && item.itemType != 4"
                    type="info"
                    >使用</Button
                  >
                  <Button
                    @click.prevent="equipment(item)"
                    size="small"
                    v-if="item.itemType != 3 && item.itemType != 4"
                    type="info"
                    >装备</Button
                  >
                  <Button
                    @click.prevent="strengthen(item)"
                    size="small"
                    type="info"
                    v-if="item.itemType != 3 && item.itemType != 4"
                    >强化</Button
                  >
                  <Button
                    @click.prevent="locking(item)"
                    size="small"
                    type="info"
                    >{{ item.bind == 0 ? "绑定" : "解绑" }}</Button
                  >
                  <Button
                    @click.prevent="sellSingle(item)"
                    size="small"
                    v-if="item.itemType != 3 && item.itemType != 4"
                    type="info"
                    >出售</Button
                  >
                </p>
                <div slot="content">
                  <p>
                    {{ item.itemName
                    }}<span v-if="item.itemType != 3 && item.itemType != 4"
                      >+{{ item.enhanLevel }}</span
                    >
                  </p>
                  <p>装备类型 {{ item.typeDec }}</p>
                  <p>装备等级 {{ item.level }}</p>
                  <p>物品状态 {{ item.bind == 0 ? "解绑" : "绑定" }}</p>
                  <p v-if="item.itemNum">物品数量： {{ item.itemNum }}</p>
                  <p v-if="item.life">
                    生命值 {{ item.life }}
                    <span v-if="item.strengLife">({{ item.strengLife }})</span>
                  </p>
                  <p v-if="item.mana">
                    真气值 {{ item.mana }}
                    <span v-if="item.strengMana">({{ item.strengMana }})</span>
                  </p>
                  <p v-if="item.attack">
                    攻击力 {{ item.attack }}
                    <span v-if="item.strengAttack"
                      >({{ item.strengAttack }})</span
                    >
                  </p>
                  <p v-if="item.magAttack">
                    法术攻击力 {{ item.magAttack }}
                    <span v-if="item.strengMagAttack"
                      >({{ item.strengMagAttack }})</span
                    >
                  </p>
                  <p v-if="item.defense">
                    防御力 {{ item.defense }}
                    <span v-if="item.strengDefense"
                      >({{ item.strengDefense }})</span
                    >
                  </p>
                  <p v-if="item.critical">
                    暴击率 {{ (item.critical * 100).toFixed(2) }}%
                  </p>
                  <p v-if="item.speed">行动速度 {{ item.speed }}</p>
                  <p v-if="item.physique">
                    体格 {{ item.physique }}
                    <span v-if="item.strengPhysique"
                      >({{ item.strengPhysique }})</span
                    >
                  </p>
                  <p v-if="item.dexterous">
                    灵巧 {{ item.dexterous }}
                    <span v-if="item.strengDexterous"
                      >({{ item.strengDexterous }})</span
                    >
                  </p>
                  <p v-if="item.spirit">
                    灵力 {{ item.spirit }}
                    <span v-if="item.strengSpirit"
                      >({{ item.strengSpirit }})</span
                    >
                  </p>
                  <p v-if="item.decs">描述： {{ item.decs }}</p>
                </div>
              </Poptip>
            </div>
          </div>
        </Card>

        <shop></shop>

        <rankingList></rankingList>
      </div>

      <div id="right">
        <div class="line"></div>

        <!-- 技能 -->
        <Card>
          <p slot="title">技能</p>
          <div slot="extra"></div>
          <div>
            <div style="overflow: hidden; margin-bottom: 15px">
              <div class="activeSkill">
                <h4>主动技能</h4>
                <div
                  v-for="(item, index) in activeList"
                  :key="index"
                  style="margin-bottom: 4px"
                >
                  <!-- <span>{{ item.skillName }}:</span> -->
                  <Poptip trigger="hover" :title="item.skillName">
                    <p style="white-space: nowrap; height: 24px">
                      <span>{{ item.skillName }}:</span>
                      <Button
                        @click.prevent="TakeSkill(item, 1)"
                        size="small"
                        type="info"
                        >脱下</Button
                      >
                    </p>
                    <div slot="content">
                      <p>技能效果: {{ item.skillDesc }}</p>
                      <p>技能耗蓝: {{ item.conMana }}</p>
                      <p>升级所需金币 {{ item.consumeCoin }}</p>
                      <p>下一等级技能效果: {{ item.skillNextDesc }}</p>
                    </div>
                  </Poptip>
                </div>
              </div>
              <div class="passiveSkill">
                <h4>被动技能</h4>
                <div
                  v-for="(item, index) in passiveList"
                  :key="index"
                  style="margin-bottom: 4px"
                >
                  <Poptip trigger="hover" :title="item.skillName">
                    <p style="white-space: nowrap; height: 24px">
                      <span>{{ item.skillName }}:</span>
                      <Button
                        @click.prevent="TakeSkill(item, 2)"
                        size="small"
                        type="info"
                        >脱下</Button
                      >
                    </p>
                    <div slot="content">
                      <p>技能效果: {{ item.skillDesc }}</p>
                      <p>技能耗蓝: {{ item.conMana }}</p>
                      <p>升级所需金币 {{ item.consumeCoin }}</p>
                      <p>下一等级技能效果: {{ item.skillNextDesc }}</p>
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
                  <Button
                    @click.prevent="upgrade(item)"
                    size="small"
                    type="info"
                    >升级</Button
                  >
                </p>
                <div slot="content">
                  <!-- <p>{{ item.skillName }}</p> -->
                  <p>技能效果: {{ item.skillDesc }}</p>
                  <p>技能耗蓝: {{ item.conMana }}</p>
                  <p>升级所需金币 {{ item.consumeCoin }}</p>
                  <p>下一等级技能效果: {{ item.skillNextDesc }}</p>
                </div>
              </Poptip>
            </div>
          </div>
        </Card>

        <playersList></playersList>
      </div>
    </div>
  </div>
</template>

<script>
import vueCanvasNest from "vue-canvas-nest";
import headers from "./headers.vue";
import playersList from "./playersList.vue";
import rankingList from "./rankingList.vue";
import shop from "./shop.vue";
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
      equipmentList: {}, //装备列表
      electronicEquipment: false, // 电子设备
      shopList: [], // 商店列表
      classification: "0", // 装备分类
      quality: "0", // 装备品质
      strengthenEject: false, // 强化提示框
      materialsRequired: {
        coin: "",
        coninType: "",
        materOneName: "",
        materOneNum: "",
        materTwoName: "",
        materTwoNum: "",
      }, // 所需材料
      packItemId: "", // 强化需要用到的
    };
  },
  computed: {
    filteredItems: function () {
      let knapsackList = this.knapsackList.slice(0);

      if (this.classification != 0) {
        knapsackList = knapsackList.filter(
          (item) => item.itemType == this.classification
        );
      }

      if (this.quality != 0) {
        knapsackList = knapsackList.filter(
          (item) => item.color == this.quality
        );
      }

      return knapsackList;
    },
  },

  components: { vueCanvasNest, headers, playersList, rankingList, shop },
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

    // 判断是手机端还是pc端
    if (this._isMobile()) {
      this.electronicEquipment = false;
    } else {
      this.electronicEquipment = true;
    }

    // 如果没有角色id就退出登录
    if (!this.getCookie("charaId")) {
      this.$Message.warning("请重新登录");
      this.$router.push({ name: "Login" });
      return;
    }

    // 获取地图列表
    this.$http.get("gameChara/queryMapList").then((res) => {
      this.mapList = res.data.data;
    });
    this.getAllUser(); // 获取用户
    this.getAllPackage(); // 获取全部包裹
    this.getAliiEquip(); // 获取穿戴的装备列表
    this.getAllSkill(); // 获取技能列表
    this.getEquipmentSkill(); // 获取已装备的技能列表
    // this.waresList(); // 获取商店列表

    setInterval(() => {
      this.leader();
    }, 1800000);

    this.$bus.$on("aMsg", (msg) => {
      setTimeout(() => {
        this.getAllPackage();
        this.getAllSkill();
        this.getAllUser();
      });
    });

    this.$bus.$on("shopMsg", (msg) => {
      this.getAllPackage();
      this.getAllUser();
      this.getAllSkill();
    });
  },
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

    // 一键脱下
    allTake() {
      this.$http
        .post(
          "/gameCharaEquip/oneDownEquitBypackage?charaId=" +
            this.getCookie("charaId")
        )
        .then((res) => {
          this.$Message.warning(res.data.data);
          this.getAllUser(); // 获取用户
          this.getAliiEquip(); // 获取穿戴的装备列表
          this.getAllPackage(); // 获取全部包裹
        })
        .catch((err) => {
          this.$Message.warning("装备一键脱下失败,请联系管理员");
        });
    },

    // 判断是手机端还是pc端
    _isMobile() {
      let flag = navigator.userAgent.match(
        /(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|BlackBerry|IEMobile|MQQBrowser|JUC|Fennec|wOSBrowser|BrowserNG|WebOS|Symbian|Windows Phone)/i
      );
      return flag;
    },

    // 返回数据处理的装备列表
    getEquipMap(player) {
      const equips = {};
      Object.values(this.EquipSlot).map((slotId) => {
        equips[this.SlotName[slotId]] = player.find(
          (item) => item.geKind === slotId
        );
      });
      return equips;
    },

    // 获取角色的装备列表
    getAliiEquip() {
      this.$http
        .post(
          "gameCharaEquip/getCharaEquip?charaId=" + this.getCookie("charaId")
        )
        .then((res) => {
          this.equipmentList = this.getEquipMap(res.data.data);
        })
        .catch((err) => {
          this.$Message.warning("获取装备列表失败,请联系管理员");
        });
    },

    // 获取角色的全部消息
    getAllUser() {
      this.$http
        .get(
          "gamepassport/getGameCharacter?charaId=" + this.getCookie("charaId")
        )
        .then((res) => {
          this.user = res.data.data;
          this.user.charaId = this.getCookie("charaId");
        })
        .catch((err) => {
          this.$Message.warning("获取角色失败,请联系管理员");
        });
    },

    // 获取角色的j技能列表
    getAllSkill() {
      this.$http
        .post(
          "gameCharaSkill/getCharaSkill/?charaId=" + this.getCookie("charaId")
        )
        .then((res) => {
          this.skillList = res.data.data;
        })
        .catch((err) => {
          this.$Message.warning("获取技能列表失败,请联系管理员");
        });
    },

    // 获取角色已装备的技能列表
    getEquipmentSkill() {
      this.$http
        .post(
          "gameCharaSkill/getCharaUseSkill/?charaId=" +
            this.getCookie("charaId") +
            "&skillType=" +
            1
        )
        .then((res) => {
          this.activeList = res.data.data;
        })
        .catch((err) => {
          this.$Message.warning("获取已装备的主动技能列表失败,请联系管理员");
        });

      this.$http
        .post(
          "gameCharaSkill/getCharaUseSkill/?charaId=" +
            this.getCookie("charaId") +
            "&skillType=" +
            2
        )
        .then((res) => {
          this.passiveList = res.data.data;
        })
        .catch((err) => {
          this.$Message.warning("获取已装备的被动技能列表失败,请联系管理员");
        });
    },

    // 装备技能
    skillEquip(item) {
      this.$http
        .post(
          "gameCharaSkill/makeSkill/?charaId=" +
            this.getCookie("charaId") +
            "&skillId=" +
            item.skillId +
            "&skillType=" +
            item.skillType
        )
        .then((res) => {
          this.$Message.warning("装备技能成功");
          this.getAllUser();
          this.getAllSkill();
          this.getEquipmentSkill();
        })
        .catch((err) => {
          this.$Message.warning("装备技能失败,请联系管理员");
        });
    },

    // 技能升级
    upgrade(item) {
      this.$http
        .post(
          "/gameCharaSkill/upgradeSkill?charaId=" +
            this.getCookie("charaId") +
            "&skillId=" +
            item.skillId
        )
        .then((res) => {
          this.$Message.warning(res.data.data);
          this.getAllUser();
          this.getAllSkill();
          this.getEquipmentSkill();
        })
        .catch((err) => {
          this.$Message.warning("技能升级失败,请联系管理员");
        });
    },

    // 脱下技能
    TakeSkill(item) {
      this.$http
        .post(
          "gameCharaSkill/makeDownSkill/?charaId=" +
            this.getCookie("charaId") +
            "&skillId=" +
            item.skillId +
            "&skillType=" +
            item.skillType
        )
        .then((res) => {
          this.$Message.warning("技能脱下成功");
          this.getAllUser();
          this.getAllSkill();
          this.getEquipmentSkill();
        })
        .catch((err) => {
          this.$Message.warning("技能脱下失败,请联系管理员");
        });
    },

    // 获取用户全部包裹
    getAllPackage() {
      this.$http
        .post("gameChara/getCharaPackage?charaId=" + this.getCookie("charaId"))
        .then((res) => {
          this.knapsackList = res.data.data;
        })
        .catch((err) => {
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
          "gamepassport/updataGameName?charaId=" +
            this.getCookie("charaId") +
            "&name=" +
            this.userName
        )
        .then((res) => {
          this.$Message.success("修改成功,请不要联系管理员");
        })
        .catch((err) => {
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
          "gameChara/checkMapGenMon?charaId=" +
            this.getCookie("charaId") +
            "&mapId=" +
            this.mapid
        )
        .then((res) => {
          if (res.data.status == 205) {
            this.$Message.warning(res.data.msg);
            setTimeout(() => {
              this.battleState = false;
              this.deteSetMap(mapid);
            }, 5000);
            return;
          }
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
                    result: "战斗失败",
                  });

                  this.battleState = false;
                } else {
                  let goods = res.data.data.gameItemsList.map((val) => {
                    return val.itemName;
                  });
                  goods = goods.join();

                  this.recording.push({
                    type: "result",
                    result: "战斗成功",
                    exp: "获得" + res.data.data.dropExp + "经验;",
                    goods: "物品:" + (goods || "无"),
                    reHealth: res.data.data.reHealth,
                    reMana: res.data.data.reMana,
                  });

                  // // 背包超过50个以上 不更新背包接口
                  // if (this.knapsackList.length >= 50) {
                  //   this.$Message.warning("背包已满");
                  // } else {
                  //   this.getAllPackage();
                  // }
                  this.getAllPackage(); // 获取全部物品
                  this.getAllSkill(); // 获取技能
                  this.getAllUser();

                  this.battleState = false;
                }

                // 战斗记录超过50条 清空
                if (this.recording.length >= 50) {
                  this.recording = [];
                }

                this.battleTimer = setTimeout(() => {
                  this.battleState = false;
                  this.deteSetMap(mapid);
                }, 6000);
              }
            }, idx * 1500);
          });
        })
        .catch((err) => {
          this.$Message.warning("战斗开始失败,请联系管理员");
          setTimeout(() => {
            this.battleState = false;
            this.deteSetMap(mapid);
          }, 3000);
        });
    },

    // 确认修改属性的按钮
    setAttribute() {
      this.user.charaId = this.getCookie("charaId");
      this.$http
        .post("gameChara/updateAttrPoint", this.user)
        .then((res) => {
          this.$Message.warning("修改成功");
          this.getAllUser();
        })
        .catch((err) => {
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
      this.filteredItems.forEach((val) => {
        arrKnapsac.push(val.packItemId);
      });
      this.$http
        .post(
          "gameCharaEquip/oneClickSale?charaId=" +
            this.getCookie("charaId") +
            "&packItemIds=" +
            arrKnapsac
        )
        .then((res) => {
          this.getAllUser();
          this.getAllPackage();
          // 每次出售完 就退到2个全部
          this.classification = "0";
          this.quality = "0";
        })
        .catch((err) => {
          this.$Message.warning("出售全部物品失败,请联系管理员");
        });
    },

    // 脱下单件装备
    TakeOffsingleton(item) {
      this.$http
        .post(
          "/gameCharaEquip/downEquitBypackage?charaId=" +
            this.getCookie("charaId") +
            "&equitId=" +
            item.equitId
        )
        .then((res) => {
          this.getAliiEquip();
          this.getAllUser();
          this.getAllPackage();
        })
        .catch((err) => {
          this.$Message.warning("脱下单件装备失败,请联系管理员");
        });
    },

    // 出售单件装备
    sellSingle(item) {
      this.$http
        .post(
          "gameCharaEquip/oneClickSale?charaId=" +
            this.getCookie("charaId") +
            "&packItemIds=" +
            item.packItemId
        )
        .then((res) => {
          this.getAllUser();
          this.getAllPackage();
        })
        .catch((err) => {
          this.$Message.warning("出售单件物品失败,请联系管理员");
        });
    },

    // 物品使用按钮
    useItems(item) {
      this.$http
        .post(
          "/gameCharaEquip/usePropypackage?charaId=" +
            this.getCookie("charaId") +
            "&packItemId=" +
            item.packItemId
        )
        .then((res) => {
          this.$Message.warning(res.data.msg);
          // this.getAliiEquip();
          this.getAllPackage();
          this.getAllUser();
        })
        .catch((err) => {
          this.$Message.warning("使用失败,请联系管理员");
        });
    },

    // 装备界面上的装备按钮
    equipment(item) {
      this.$http
        .post(
          "gameCharaEquip/useEquitBypackage?charaId=" +
            this.getCookie("charaId") +
            "&packItemId=" +
            item.packItemId
        )
        .then((res) => {
          this.$Message.warning(res.data.data);
          this.getAliiEquip();
          this.getAllPackage();
          this.getAllUser();
        })
        .catch((err) => {
          this.$Message.warning("装备失败,请联系管理员");
        });
    },

    // 装备界面上的强化按钮
    strengthen(item) {
      this.strengthenEject = true;
      this.packItemId = item.packItemId;
      this.$http
        .post(
          "/gameCharaEquip/getCaiEquitStrengThen?charaId=" +
            this.getCookie("charaId") +
            "&packItemId=" +
            item.packItemId
        )
        .then((res) => {
          this.materialsRequired = res.data.data;
        })
        .catch((err) => {
          this.$Message.warning("强化材料失败,请联系管理员");
        });
    },

    // 强化提示的确定
    changeStrengthen() {
      this.$http
        .post(
          "/gameCharaEquip/equitStrengThen?charaId=" +
            this.getCookie("charaId") +
            "&packItemId=" +
            this.packItemId
        )
        .then((res) => {
          this.$Message.warning(res.data.msg);
          this.getAllPackage();
          this.getAllUser();
        })
        .catch((err) => {
          this.$Message.warning("强化失败,请联系管理员");
        });
    },

    // 装备界面上的锁定按钮
    locking(item) {
      this.$http
        .post(
          "/gameMarket/itemBind?charaId=" +
            this.getCookie("charaId") +
            "&packItemId=" +
            item.packItemId +
            "&bind=" +
            (item.bind == 0 ? 1 : 0)
        )
        .then((res) => {
          this.$Message.warning(res.data.data);
          this.getAllPackage();
        })
        .catch((err) => {
          this.$Message.warning("绑定失败,请联系管理员");
        });
    },

    // 兑换金币  暂时关闭
    exchangeCoin() {
      this.$http
        .post("	/gamepassport/exchange?charaId=" + this.getCookie("charaId"))
        .then((res) => {
          this.$Message.warning(res.data.msg);
          this.getAllUser();
        })
        .catch((err) => {
          this.$Message.warning("兑换失败.请联系管理员");
          this.getAllUser();
        });
    },
  },
};
</script>
<style>
html {
  height: 100%;
}
body {
  height: 100%;
}
#app {
  height: 100%;
}
#app2 {
  height: 100%;
  overflow: auto;
}
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
.two {
  position: absolute;
  width: 760px;
  height: 340px;
}
.line {
  height: 354px;
}
#app2 {
  min-width: 1140px;
}
#box {
  width: 1140px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  position: relative;
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
  /* margin-bottom: 40px; */
  margin-bottom: 18px;
  margin-top: 8px;
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
  max-height: 168px;
  overflow: auto;
  transition: all 0.4s;
}

/* 头像下面 基础属性 和 加点的位置*/
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
.jjt_smail {
  color: #b50111;
  font-size: 12px;
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
