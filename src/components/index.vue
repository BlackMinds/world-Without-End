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
          >
            修改名字
          </Button>
          <div>
            <div class="character">
              <Upload
                type="drag"
                :before-upload="handleUpload"
                action="http://www.yunyingxiaowu.com:8088/foodie-api/gamepassport/uploadFace"
              >
                <div style="width: 100px; height: 100px; overflow: hidden">
                  <p
                    class="touxiang"
                    :style="{ backgroundImage: 'url(' + user.face + ')' }"
                  ></p>
                </div>
              </Upload>
              <!-- <Avatar src="https://i.loli.net/2017/08/21/599a521472424.jpg" /> -->
              <div class="rank">
                <div>
                  <span style="float: left">LV{{ user.level }}</span>
                  <span style="float: right">编号{{ user.accountId }}</span>
                </div>
                <div class="bloodVolume">生命: {{ user.health }}</div>
                <div class="blueAmount">真气值: {{ user.mana }}</div>
              </div>
            </div>
            <div class="goldCoin">
              <p>当前等级经验: {{ user.exp }} / {{ user.upgradeExp }}</p>
              <!-- 铜钱买普通物品 -->
              <p>当前铜钱: {{ user.money }}</p>
              <p>当前金币: {{ user.coin }}</p>
              <p>副本奖励剩余次数: {{ user.rewardNum }}</p>
              <p>
                境界:
                <Poptip
                  trigger="hover"
                  :title="user.realmName"
                  @on-popper-show="getRealmAttribute"
                >
                  <span
                    :style="{
                      color: realmColor,
                      boxShadow: realmShadow,
                      fontSize: '18px',
                    }"
                  >
                    {{ user.realmName }}
                  </span>
                  <div slot="content" class="realmAttribute">
                    <p v-if="realmAttribute.realmAttack">
                      <span>物理攻击加成</span>
                      {{ (realmAttribute.realmAttack * 100).toFixed(2) || 0 }} %
                    </p>
                    <p v-if="realmAttribute.realmMagicAttack">
                      <span>灵力攻击加成</span>
                      {{
                        (realmAttribute.realmMagicAttack * 100).toFixed(2) || 0
                      }}
                      %
                    </p>
                    <p v-if="realmAttribute.realmHealth">
                      <span>血量加成</span>
                      {{ (realmAttribute.realmHealth * 100).toFixed(2) || 0 }} %
                    </p>
                    <p v-if="realmAttribute.realmMana">
                      <span>真气加成</span>
                      {{ (realmAttribute.realmMana * 100).toFixed(2) || 0 }} %
                    </p>
                    <p v-if="realmAttribute.realmDefense">
                      <span>防御加成</span>
                      {{
                        (realmAttribute.realmDefense * 100).toFixed(2) || 0
                      }}
                      %
                    </p>
                    <p v-if="realmAttribute.realmCriticalHit">
                      <span>暴击加成</span>
                      {{
                        (realmAttribute.realmCriticalHit * 100).toFixed(2) || 0
                      }}
                      %
                    </p>
                    <p v-if="realmAttribute.realmCriticalDamage">
                      <span>暴击伤害加成</span>
                      {{
                        (realmAttribute.realmCriticalDamage * 100).toFixed(2) ||
                        0
                      }}
                      %
                    </p>
                    <p v-if="realmAttribute.realmEvade">
                      <span>闪避加成</span>
                      {{ (realmAttribute.realmEvade * 100).toFixed(2) || 0 }} %
                    </p>
                    <p v-if="realmAttribute.realmHitRat">
                      <span>命中加成</span>
                      {{ (realmAttribute.realmHitRat * 100).toFixed(2) || 0 }} %
                    </p>
                  </div>
                </Poptip>
                &nbsp; 修真经验 {{ user.realmExp }} / {{ user.realmUpExp }}
                <Button size="small" @click="breach">突破</Button>
              </p>
            </div>
          </div>
        </Card>

        <!-- 境界突破提示框 -->
        <Modal
          v-model="realmEject"
          title="境界突破提示"
          @on-ok="changeRealm"
          :styles="{ top: '240px' }"
        >
          <p v-if="breachStuff.realmLevel">
            突破等级要求: {{ breachStuff.realmLevel }}
          </p>
          <p v-if="breachStuff.realmMaxLevel">
            突破后等级上限: {{ breachStuff.realmMaxLevel }}
          </p>
          <p v-if="breachStuff.realmName">
            突破后境界: {{ breachStuff.realmName }}
          </p>
          <p v-if="breachStuff.updateExp">
            突破消耗修真经验: {{ breachStuff.updateExp }}
          </p>
          <p v-if="breachStuff.materOneName">
            {{ breachStuff.materOneName }}: 需要
            {{ breachStuff.materOneNum }}
          </p>
          <p v-if="breachStuff.materTwoName">
            {{ breachStuff.materTwoName }}: 需要
            {{ breachStuff.materTwoNum }}
          </p>
          <p v-if="breachStuff.materThreeName">
            {{ breachStuff.materThreeName }}: 需要
            {{ breachStuff.materThreeNum }}
          </p>
          <p v-if="breachStuff.coin">
            需要 {{ breachStuff.coninType == 0 ? "铜币" : "金币" }}
            {{ breachStuff.coin }}
          </p>
        </Modal>

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
            >
              {{ user.reAttrPoint }}
            </span>
          </p>
          <!-- 这个确认修改不确定要不要 我想的是买药水然后使用修改 -->
          <Button slot="extra" size="small" @click="setAttribute" type="info">
            确认修改
          </Button>
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
                命中率:
                <span>{{ (user.hitRate * 100).toFixed(2) || 0 }}%</span>
              </p>
              <p>
                闪避率:
                <span>{{ (user.evade * 100).toFixed(2) || 0 }}%</span>
              </p>
              <p>
                暴击率:
                <span>
                  {{ (user.criticalHitValue * 100).toFixed(2) || 0 }}%
                </span>
              </p>
              <p>
                暴击伤害:
                <span>{{ (user.critDamage * 100).toFixed(2) || 0 }}%</span>
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
            >
              一键脱下
            </Button>
          </div>
          <div class="equipmentBar">
            <div v-for="(item, key, index) in equipmentList" :key="index">
              <span>{{ key }}:</span>
              <Poptip trigger="hover" v-if="item">
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
                  >
                    +{{ item.enhanLevel }}
                  </span>
                  <Button
                    @click.prevent="TakeOffsingleton(item)"
                    size="small"
                    type="info"
                  >
                    脱下
                  </Button>
                </p>
                <div slot="content">
                  <p>
                    {{ item.equitName }}
                    <span v-if="item.itemType != 3 && item.itemType != 4">
                      +{{ item.enhanLevel }}
                    </span>
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
                    <span v-if="item.strengAttack">
                      ({{ item.strengAttack }})
                    </span>
                  </p>
                  <p v-if="item.magAttack">
                    法术攻击力 {{ item.magAttack }}
                    <span v-if="item.strengMagAttack">
                      ({{ item.strengMagAttack }})
                    </span>
                  </p>
                  <p v-if="item.defense">
                    防御力 {{ item.defense }}
                    <span v-if="item.strengDefense">
                      ({{ item.strengDefense }})
                    </span>
                  </p>
                  <p v-if="item.hitRate">
                    命中率 {{ (item.hitRate * 100).toFixed(2) }}%
                  </p>
                  <p v-if="item.evade">
                    闪避率 {{ (item.evade * 100).toFixed(2) }}%
                  </p>
                  <p v-if="item.critical">
                    暴击率 {{ (item.critical * 100).toFixed(2) }}%
                  </p>
                  <p v-if="item.criDamage">
                    暴击伤害 {{ (item.criDamage * 100).toFixed(2) }}%
                  </p>
                  <p v-if="item.speed">行动速度 {{ item.speed }}</p>
                  <p v-if="item.physique">
                    体格 {{ item.physique }}
                    <span v-if="item.strengPhysique">
                      ({{ item.strengPhysique }})
                    </span>
                  </p>
                  <p v-if="item.dexterous">
                    灵巧 {{ item.dexterous }}
                    <span v-if="item.strengDexterous">
                      ({{ item.strengDexterous }})
                    </span>
                  </p>
                  <p v-if="item.spirit">
                    灵力 {{ item.spirit }}
                    <span v-if="item.strengSpirit">
                      ({{ item.strengSpirit }})
                    </span>
                  </p>
                  <Divider v-if="item.decs" />
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
            >
              {{ item.mapName }}
            </Option>
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
            @click.prevent="withdrawFromAction"
            size="small"
            type="warning"
            :loading="withdrawFromActionStatus"
          >
            退出战斗
          </Button>
          <Button
            style="margin-left: 30px"
            slot="extra"
            @click.prevent="
              withdrawFromActionStatus = false;
              deteSetMap();
            "
            size="small"
            type="info"
          >
            确定
          </Button>
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
                  <p>命中率 {{ item.hitRate * 100 }}%</p>
                  <p>闪避率 {{ item.evade * 100 }}%</p>
                  <p>暴击率 {{ item.explode * 100 }}%</p>
                  <p>暴击伤害 {{ item.criDamage * 100 }}%</p>
                  <p>行动速度 {{ item.speed }}</p>
                  <!-- <p>掉落物品 {{ item.wp ||  '无'}}</p> -->
                </div>
              </Poptip>
            </div>
            <div class="recording" id="recording">
              <h6 v-if="monsterList.length <= 0">暂无记录</h6>
              <p v-for="(item, index) in recording" :key="index">
                <!-- 战斗打架输出的日志 -->
                <span v-if="item.type == 'combat'">
                  <span
                    :style="{
                      color: item.identity == 0 ? 'orange' : '#5b00ff',
                    }"
                  >
                    {{ item.attacker }}
                  </span>
                  攻击了
                  <span style="color: red">{{ item.hinjured }}</span>
                  <span v-if="item.isSkill != 1">
                    造成了
                    <span
                      :style="{
                        color: '#5b00ff',
                        fontSize: item.isCritical == 0 ? '14px' : '20px',
                      }"
                    >
                      {{ item.hurt }}
                    </span>
                  </span>
                  <span v-else>
                    用{{ item.skillName }}造成了
                    <span :style="{ color: 'rgb(110 0 255)' }">
                      {{ item.hurt }}
                    </span>
                  </span>
                  剩余
                  <span style="color: red">{{ item.surplusHealth }}</span>
                  血量
                  <span
                    v-if="item.isCritical == 2"
                    :style="{
                      color: '#5b00ff',
                      fontSize: '20px',
                    }"
                  >
                    被闪避了
                  </span>
                </span>
                <!-- 战斗恢复的地方 -->
                <span v-if="item.type == 'gain'">
                  <span
                    :style="{
                      color: item.identity == 0 ? 'orange' : '#5b00ff',
                    }"
                  >
                    {{ item.attacker }}
                  </span>
                  <span>
                    用{{ item.skillName }}回复
                    <span :style="{ color: 'rgb(110 0 255)' }">
                      {{ item.hurt }}
                    </span>
                  </span>
                  剩余
                  <span style="color: red">{{ item.surplusHealth }}</span>
                  血量
                </span>
                <!-- 战斗失败或成功输出的地方 -->
                <span
                  style="color: rgb(255, 0, 224)"
                  v-if="item.type == 'result'"
                >
                  {{ item.result }}: 剩余血量:{{ item.reHealth }} 剩余蓝量:
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
            <span>
              背包容量{{ knapsackList.length }} / {{ user.packageNum }}
            </span>
            <Button @click.prevent="sellAll" size="small" type="info">
              出售全部装备
            </Button>
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
                @on-popper-show="openEquipment(item)"
                @on-popper-hide="Equipment = null"
              >
                <p
                  style="white-space: nowrap; height: 24px"
                  :style="{ color: distinguishColor(item.color) }"
                >
                  <span v-if="item.itemType != 3 && item.itemType != 4">
                    [LV:{{ item.level }}]
                  </span>
                  {{ item.itemName }}
                  <span
                    v-if="
                      item.itemType != 3 &&
                      item.itemType != 4 &&
                      item.enhanLevel != 0
                    "
                  >
                    +{{ item.enhanLevel }}
                  </span>
                  <span
                    class="jjt_smail"
                    v-if="item.itemType == 3 || item.itemType == 4"
                  >
                    {{ item.itemNum }}
                  </span>
                  <Button
                    @click.prevent="useItems(item)"
                    size="small"
                    v-if="item.itemType == 3 && item.itemType != 4"
                    type="info"
                  >
                    使用
                  </Button>
                  <Button
                    @click.prevent="equipment(item)"
                    size="small"
                    v-if="item.itemType != 3 && item.itemType != 4"
                    type="info"
                  >
                    装备
                  </Button>
                  <Button
                    @click.prevent="strengthen(item)"
                    size="small"
                    type="info"
                    v-if="item.itemType != 3 && item.itemType != 4"
                  >
                    强化
                  </Button>
                  <Button
                    @click.prevent="locking(item)"
                    size="small"
                    type="info"
                  >
                    {{ item.bind == 0 ? "绑定" : "解绑" }}
                  </Button>
                  <Button
                    @click.prevent="sellSingle(item)"
                    size="small"
                    v-if="item.itemType != 3 && item.itemType != 4"
                    type="info"
                  >
                    出售
                  </Button>
                </p>
                <div slot="content" class="poptipExplain">
                  <!-- 装备栏显示的数据 -->
                  <div v-if="Equipment" class="Equipment">
                    <p :style="{ color: distinguishColor(Equipment.color) }">
                      {{ Equipment.equitName }}
                      <span
                        v-if="
                          Equipment.itemType != 3 && Equipment.itemType != 4
                        "
                      >
                        +{{ Equipment.enhanLevel }}
                      </span>
                      &nbsp;&nbsp;(已装备)
                    </p>
                    <p>装备类型 {{ Equipment.typeDec }}</p>
                    <p>装备等级 {{ Equipment.level }}</p>
                    <p>物品状态 {{ Equipment.bind == 0 ? "解绑" : "绑定" }}</p>
                    <p v-if="Equipment.itemNum">
                      物品数量： {{ Equipment.itemNum }}
                    </p>
                    <p v-if="Equipment.life">
                      生命值 {{ Equipment.life }}
                      <span v-if="Equipment.strengLife">
                        ({{ Equipment.strengLife }})
                      </span>
                    </p>
                    <p v-if="Equipment.mana">
                      真气值 {{ Equipment.mana }}
                      <span v-if="Equipment.strengMana">
                        ({{ Equipment.strengMana }})
                      </span>
                    </p>
                    <p v-if="Equipment.attack">
                      攻击力 {{ Equipment.attack }}
                      <span v-if="Equipment.strengAttack">
                        ({{ Equipment.strengAttack }})
                      </span>
                    </p>
                    <p v-if="Equipment.magAttack">
                      法术攻击力 {{ Equipment.magAttack }}
                      <span v-if="Equipment.strengMagAttack">
                        ({{ Equipment.strengMagAttack }})
                      </span>
                    </p>
                    <p v-if="Equipment.defense">
                      防御力 {{ Equipment.defense }}
                      <span v-if="Equipment.strengDefense">
                        ({{ Equipment.strengDefense }})
                      </span>
                    </p>
                    <p v-if="Equipment.hitRate">
                      命中率 {{ (Equipment.hitRate * 100).toFixed(2) }}%
                    </p>
                    <p v-if="Equipment.evade">
                      闪避率 {{ (Equipment.evade * 100).toFixed(2) }}%
                    </p>
                    <p v-if="Equipment.critical">
                      暴击率 {{ (Equipment.critical * 100).toFixed(2) }}%
                    </p>
                    <p v-if="Equipment.criDamage">
                      暴击伤害 {{ (Equipment.criDamage * 100).toFixed(2) }}%
                    </p>
                    <p v-if="Equipment.speed">行动速度 {{ Equipment.speed }}</p>
                    <p v-if="Equipment.physique">
                      体格 {{ Equipment.physique }}
                      <span v-if="Equipment.strengPhysique">
                        ({{ Equipment.strengPhysique }})
                      </span>
                    </p>
                    <p v-if="Equipment.dexterous">
                      灵巧 {{ Equipment.dexterous }}
                      <span v-if="Equipment.strengDexterous">
                        ({{ Equipment.strengDexterous }})
                      </span>
                    </p>
                    <p v-if="Equipment.spirit">
                      灵力 {{ Equipment.spirit }}
                      <span v-if="Equipment.strengSpirit">
                        ({{ Equipment.strengSpirit }})
                      </span>
                    </p>
                    <Divider v-if="Equipment.equitDec" />
                    <p v-if="Equipment.equitDec">
                      描述： {{ Equipment.equitDec }}
                    </p>
                  </div>

                  <div>
                    <p :style="{ color: distinguishColor(item.color) }">
                      {{ item.itemName }}
                      <span v-if="item.itemType != 3 && item.itemType != 4">
                        +{{ item.enhanLevel }}
                      </span>
                    </p>
                    <p>装备类型 {{ item.typeDec }}</p>
                    <p>装备等级 {{ item.level }}</p>
                    <p>物品状态 {{ item.bind == 0 ? "解绑" : "绑定" }}</p>
                    <p v-if="item.itemNum != 1">
                      物品数量： {{ item.itemNum }}
                    </p>
                    <p v-if="item.life" class="discrimination">
                      生命值 {{ item.life }}
                      <span v-if="item.strengLife">
                        ({{ item.strengLife }})
                      </span>
                    </p>
                    <p v-if="item.mana" class="discrimination">
                      真气值 {{ item.mana }}
                      <span v-if="item.strengMana">
                        ({{ item.strengMana }})
                      </span>
                    </p>
                    <p v-if="item.attack" class="discrimination">
                      攻击力 {{ item.attack }}
                      <span v-if="item.strengAttack">
                        ({{ item.strengAttack }})
                      </span>
                    </p>
                    <p v-if="item.magAttack" class="discrimination">
                      法术攻击力 {{ item.magAttack }}
                      <span v-if="item.strengMagAttack">
                        ({{ item.strengMagAttack }})
                      </span>
                    </p>
                    <p v-if="item.defense" class="discrimination">
                      防御力 {{ item.defense }}
                      <span v-if="item.strengDefense">
                        ({{ item.strengDefense }})
                      </span>
                    </p>
                    <p v-if="item.hitRate" class="discrimination">
                      命中率 {{ (item.hitRate * 100).toFixed(2) }}%
                    </p>
                    <p v-if="item.evade" class="discrimination">
                      闪避率 {{ (item.evade * 100).toFixed(2) }}%
                    </p>
                    <p v-if="item.critical" class="discrimination">
                      暴击率 {{ (item.critical * 100).toFixed(2) }}%
                    </p>
                    <p v-if="item.criDamage" class="discrimination">
                      暴击伤害 {{ (item.criDamage * 100).toFixed(2) }}%
                    </p>
                    <p v-if="item.speed" class="discrimination">
                      行动速度 {{ item.speed }}
                    </p>
                    <p v-if="item.physique" class="discrimination">
                      体格 {{ item.physique }}
                      <span v-if="item.strengPhysique">
                        ({{ item.strengPhysique }})
                      </span>
                    </p>
                    <p v-if="item.dexterous" class="discrimination">
                      灵巧 {{ item.dexterous }}
                      <span v-if="item.strengDexterous">
                        ({{ item.strengDexterous }})
                      </span>
                    </p>
                    <p v-if="item.spirit" class="discrimination">
                      灵力 {{ item.spirit }}
                      <span v-if="item.strengSpirit">
                        ({{ item.strengSpirit }})
                      </span>
                    </p>
                    <Divider v-if="item.decs" />
                    <p v-if="item.decs">描述： {{ item.decs }}</p>
                  </div>
                </div>
              </Poptip>
            </div>
          </div>
        </Card>

        <shop></shop>
      </div>

      <div id="right">
        <div class="line"></div>
        {{ teamStatus }}
        <!-- 技能 -->
        <skill></skill>
        <synthesis></synthesis>
      </div>
    </div>
  </div>
</template>

<script>
import qs from "query-string";
import vueCanvasNest from "vue-canvas-nest";
import headers from "./headers.vue";
// import playersList from "./playersList.vue";
// import rankingList from "./rankingList.vue";
import shop from "./shop.vue";
import skill from "./skill.vue";
import synthesis from "./synthesis.vue";
import { EquipSlot, SlotName } from "../const";
import { sleep } from "../utils";

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
      teamStatus: 0,
      mapEject: false,
      user: {}, // 用户的全部消息
      mapList: [], // 地图列表
      monsterList: [], //怪物信息
      recording: [], // 战斗信息
      battleState: false, // 战斗状态
      time: null, // 暂时没用到
      Equipment: undefined, // 装备栏显示
      battleTimer: null,
      knapsackList: [], // 包裹
      mapid: null, // 地图id
      activeList: [], // 穿戴的主动技能列表
      passiveList: [], // 穿戴的被动技能列表
      skillList: [], // 全部技能
      equipmentList: {}, //装备列表
      electronicEquipment: false, // 电子设备
      shopList: [], // 商店列表
      classification: "0", // 装备分类
      quality: "0", // 装备品质
      strengthenEject: false, // 强化提示框
      realmEject: false, // 境界突破提示框
      breachStuff: [], // 突破的材料
      realmColor: "skyblue", // 境界的文字颜色
      realmShadow: "0px 0px 10px #5d5f4d", // 境界的阴影
      withdrawFromActionStatus: false, // 退出战斗的判断
      realmAttribute: "", // 境界加成获取到的属性
      materialsRequired: {}, // 所需材料
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

  components: { vueCanvasNest, headers, shop, skill, synthesis },
  updated() {
    // 战斗记录定位到底部
    const el = document.getElementById("recording");
    const max = el.scrollHeight - el.clientHeight;
    if (max - el.scrollTop < 80) {
      el.scrollTop = el.scrollHeight;
    }
  },
  mounted() {
    // 生成境界的随机颜色
    var roundColor = Math.round(Math.random() * 5);
    if (roundColor == 1) {
      this.realmColor = "red";
    } else if (roundColor == 2) {
      this.realmColor = "skyblue";
    } else if (roundColor == 3) {
      this.realmColor = "orange";
    } else if (roundColor == 4) {
      this.realmColor = "greenyellow";
    } else if (roundColor == 5) {
      this.realmColor = "pink";
    }
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

    this.refreshUserInfo(); // 获取用户
    this.refreshPackage(); // 获取全部包裹
    this.refreshEquips(); // 获取穿戴的装备列表

    setTimeout(() => {
      this.$store.commit("edit", this.user);
    }, 600);
    // 兑换码发送过来的
    this.$bus.$on("dhmMsg", (msg) => {
      setTimeout(() => this.refreshPackage(), 500);
    });

    // 商店购买发送过来的
    this.$bus.$on("shopMsg", (msg) => {
      this.refreshUserInfo();
      this.refreshPackage();
    });

    // 商店批量购买发送过来的
    this.$bus.$on("shopMsg1", (msg) => {
      this.refreshUserInfo();
      this.refreshPackage();
    });

    // 脱下技能发送过来的
    this.$bus.$on("skillMsg", (msg) => {
      this.refreshUserInfo();
    });

    // 装备技能发送过来的
    this.$bus.$on("skillMsg1", (msg) => {
      this.refreshUserInfo();
    });

    // 退出登录发送过来的
    this.$bus.$on("tcdlMsg", (msg) => {
      this.closeBattleTimer();
    });

    // 有队伍之后才发送的
    this.$bus.$on("teamStatusMsg", (msg) => {
      this.closeBattleTimer();
      this.teamStatus = 1;
      this.TeamMapList();
    });

    // 没队伍之后才发送的
    this.$bus.$on("teamStatusMsg1", (msg) => {
      this.teamStatus = 0;
      this.MapList();
    });

    // 合成之后才发送的
    this.$bus.$on("synthesisMsg", (msg) => {
      this.refreshPackage();
    });

    setTimeout(() => {
      if (this.teamStatus == 0) {
        this.automaticCombat(); // 自动战斗
      }
    }, 500);
  },
  methods: {
    getRealmAttribute() {
      this.$http
        .post("/gameRealm/getRealmBonus?charaId=" + this.getCookie("charaId"))
        .then((res) => {
          this.realmAttribute = res.data.data;
        })
        .catch((err) => {
          this.$Message.warning("获取境界属性失败,请联系管理员");
        });
    },
    addRecord(record) {
      this.recording.push(record);
      // 战斗记录超过一定条数时移除一部分
      if (this.recording.length >= 200) {
        this.recording.splice(0, 100);
      }
    },

    // 获取地图列表
    MapList() {
      this.$http.get("gameChara/queryMapList").then((res) => {
        this.mapList = res.data.data;
      });
    },

    // 获取组队地图列表
    TeamMapList() {
      this.$http.get("/gameAmry/queryMapList").then((res) => {
        this.mapList = res.data.data;
      });
    },

    // 自动战斗
    automaticCombat() {
      // 如果地图id存在就自动战斗
      var mapid = this.user.accountId + "mapid";
      var mapName = this.user.accountId + "mapName";
      if (window.localStorage[mapid]) {
        this.mapName = window.localStorage[mapName];
        this.mapNameF = window.localStorage[mapName];
        this.deteSetMap(window.localStorage[mapid]);
      }
    },

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
      } else if (color == 7) {
        return "orange";
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
          this.refreshUserInfo(); // 获取用户
          this.refreshEquips(); // 获取穿戴的装备列表
          this.refreshPackage(); // 获取全部包裹
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

    // 头像上传
    handleUpload(file) {
      var image = new FormData();
      image.append("file", file);
      this.$http
        .post(
          "/gamepassport/uploadFace/?charaId=" + this.getCookie("charaId"),
          image,
          {
            headers: {
              "Content-Type": "multipart/form-data",
            },
          }
        )
        .then((res) => {
          this.refreshUserInfo();
          this.$Message.warning(res.data.msg);
        });
      return false;
    },

    // 获取角色的装备列表
    refreshEquips() {
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
    refreshUserInfo() {
      this.$http
        .get(
          "gamepassport/getGameCharacter?charaId=" + this.getCookie("charaId")
        )
        .then((res) => {
          this.user = res.data.data;
          this.user.charaId = this.getCookie("charaId");
          // console.log(this.user);
        })
        .catch((err) => {
          this.$Message.warning("获取角色失败,请联系管理员");
        });
    },

    // 获取用户全部包裹
    refreshPackage() {
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

    // 退出战斗
    withdrawFromAction() {
      this.withdrawFromActionStatus = true;
    },

    // 关闭战斗定时
    closeBattleTimer() {
      clearTimeout(this.battleTimer);
      this.battleTimer = null;
    },

    // 切换地图弹出框的确定
    async deteSetMap(mapid) {
      if (this.battleState) {
        this.$Message.warning("还在战斗状态 不能切换");
        return;
      }

      if (this.withdrawFromActionStatus) {
        this.withdrawFromActionStatus = false;
        return;
      }

      this.mapNameF = this.mapName;
      this.mapid = mapid;

      for (var i = 0; i < this.mapList.length; i++) {
        if (this.mapName == this.mapList[i].mapName) {
          this.mapid = this.mapList[i].mapId;
        }
      }

      // 存进locastorange做自动战斗
      window.localStorage.setItem(this.user.accountId + "mapid", this.mapid);
      window.localStorage.setItem(
        this.user.accountId + "mapName",
        this.mapName
      );

      // 是否是游戏副本
      const isInstanceZones = this.teamStatus !== 0;
      // 副本使用不同的战斗接口
      let combatAddress = isInstanceZones
        ? "/gameAmry/checkMapGenMon"
        : "/gameChara/checkMapGenMon";

      try {
        const res = await this.$http.post(
          `${combatAddress}?${qs.stringify({
            charaId: this.getCookie("charaId"),
            mapId: this.mapid,
          })}`
        );
        const body = res.data;
        // 判断他有没有战斗结束就重新调用接口
        if (body.status == 205) {
          this.closeBattleTimer();
          this.$Message.warning(body.msg);
          await sleep(6000);
          this.battleState = false;
          this.deteSetMap(mapid);
          return;
        }

        // // 有错误提示 就这里出现 并且重新请求战斗
        // if (body.msg) {
        //   this.$Message.warning(body.msg + ",6秒后自动请求战斗")
        //   this.battleTimer = setTimeout(() => {
        //       this.battleState = false;
        //     this.deteSetMap(mapid);
        //   }, 6000);
        //   return
        // }

        this.battleState = true;
        // 非副本战斗返回的结果是一个直接的对象，这里转换成一样的结构
        const combatResults = isInstanceZones ? body.data : [body.data];
        // 按顺序播报所有的战斗结果
        for (let i = 0; i < combatResults.length; i++) {
          await this.boardcastCombatResult(combatResults[i]);
        }
        this.battleState = false;
      } catch (err) {
        console.log(err);
        this.$Message.warning("战斗开始失败6秒后自动请求战斗,请联系管理员");
      }

      this.battleTimer = setTimeout(() => {
        this.battleState = false;
        this.deteSetMap(mapid);
      }, 6000);
    },

    // 播报战斗返回的日志
    async boardcastCombatResult(result) {
      this.monsterList = result.gameMonList; // 怪物信息
      let combatInfo = result.combatInfo;

      for (let i = 0; i < combatInfo.length; i++) {
        // 每条记录的输出相隔 1.5 秒
        await sleep(1500);

        const record = combatInfo[i];
        // 1 物理伤害 2灵力伤害 3持续伤害 4恢复血量
        // console.log(record, "战斗日志");
        if (record.hurtType == 4) {
          record.type = "gain";
        } else {
          record.type = "combat";
        }
        this.addRecord(record);
      }

      // 额外增加一个战斗结果的消息记录
      if (result.reHealth <= 0) {
        this.addRecord({
          type: "result",
          result: "战斗失败",
        });
      } else {
        // 物品列表
        const goods = result.gameItemsList.map((val) => {
          return val.itemName;
        });
        const droppedGood = goods.length > 0;

        this.addRecord({
          type: "result",
          result: "战斗成功",
          exp: "获得" + result.dropExp + "经验;",
          goods: "物品:" + (droppedGood ? goods.join() : "无"),
          reHealth: result.reHealth,
          reMana: result.reMana,
        });

        if (droppedGood) {
          this.refreshPackage();
          // 通知技能刷新检查
          this.$bus.$emit("getSkillMsg");
        }
        this.refreshUserInfo();
      }
    },

    // 确认修改属性的按钮
    setAttribute() {
      this.user.charaId = this.getCookie("charaId");
      this.$http
        .post("gameChara/updateAttrPoint", this.user)
        .then((res) => {
          this.$Message.warning("修改成功");
          this.refreshUserInfo();
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

    // 显示装备对比属性
    openEquipment(item) {
      for (let key in this.equipmentList) {
        if (item.kind == this.equipmentList[key].geKind) {
          this.Equipment = this.equipmentList[key];
        }
      }
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
          this.refreshUserInfo();
          this.refreshPackage();
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
          this.refreshEquips();
          this.refreshUserInfo();
          this.refreshPackage();
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
          this.refreshUserInfo();
          this.refreshPackage();
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
          // this.refreshEquips();
          this.refreshPackage();
          this.refreshUserInfo();
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
          this.refreshEquips();
          this.refreshPackage();
          this.refreshUserInfo();
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
          this.refreshPackage();
          this.refreshUserInfo();
        })
        .catch((err) => {
          this.$Message.warning("强化失败,请联系管理员");
        });
    },

    // 突破按钮
    breach() {
      this.realmEject = true;
      this.$http
        .post("/gameRealm/getCaiRealm?charaId=" + this.getCookie("charaId"))
        .then((res) => {
          this.breachStuff = res.data.data;
        })
        .catch((err) => {
          this.$Message.warning("突破获取材料失败,请联系管理员");
        });
    },

    // 境界突破确定
    changeRealm() {
      this.$http
        .post(
          "/gameRealm/breakThroughTheRealm?charaId=" + this.getCookie("charaId")
        )
        .then((res) => {
          this.$Message.warning(res.data.msg);
          this.refreshUserInfo(); // 获取用户
          this.refreshPackage(); // 获取全部包裹
        })
        .catch((err) => {
          this.$Message.warning("突破失败,请联系管理员");
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
          this.refreshPackage();
        })
        .catch((err) => {
          this.$Message.warning("绑定失败,请联系管理员");
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
.character /deep/ .ivu-upload-drag {
  float: left;
  width: 100px;
  height: 100px;
  border-radius: 50%;
}
.touxiang {
  width: 100%;
  height: 100%;
  background-position: center center;
  background-size: cover;
}
.rank {
  float: left;
  margin-left: 10px;
}
.rank div {
  overflow: hidden;
}
.rank div > span {
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

/*  战斗场景  */
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
.knapsack /deep/ .ivu-poptip-popper,
.equipmentBar /deep/ .ivu-poptip-popper {
  pointer-events: none;
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

/* 装备栏数据对比显示 */
.Equipment {
  width: 200px;
  white-space: normal;
  word-break: break-all;
  margin-right: 15px;
}

.Equipment + div {
  width: 200px;
  white-space: normal;
  word-break: break-all;
}

/* 装备栏数据对比显示左右对比数据显示 */
.poptipExplain {
  display: flex;
}

/* 调整文字颜色 */
.poptipExplain div p {
  color: rgb(73, 193, 248);
}
.poptipExplain div p span {
  color: rgb(0, 255, 55);
}
.poptipExplain div p:nth-child(1),
.poptipExplain div p:nth-child(2),
.poptipExplain div p:nth-child(3),
.poptipExplain div p:nth-child(4) {
  color: inherit;
}

/* 境界属性带来的提升 */
.realmAttribute p span {
  color: rgb(45, 196, 255);
}
</style>
