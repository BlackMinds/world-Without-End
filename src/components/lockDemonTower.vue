<template>
    <!-- {{ mapList }} -->
    <!-- <p slot="title">锁妖塔</p> -->
    <div>
        <div>
            <Select v-model="mapName" style="width: 50%">
                <!-- ({{ item.minLv }}) -->
                <Option v-for="item in mapList" :value="item.mapName" :name="item.mapId" :key="item.mapId">
                    {{ item.mapName }} {{item.minLv}}-{{item.maxLv}}
                </Option>
            </Select>

            <Button @click.prevent="startTheBattle" size="small" type="info">
                开始战斗
            </Button>
        </div>

        <div>
            <div class="monster" :class="[monsterList.length == 1 ? 'monstercenter' : '']">
                <Poptip trigger="hover" v-for="item in monsterList" :key="item.id" :title="item.name">
                    ({{ item.prefix == 0 ? "无" : item.prefix }}){{ item.name }}
                    <Progress :percent="(Math.max(item.life, 0) / item.maxLife) * 100">
                        <span>{{ item.life }}</span>
                    </Progress>
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
                    </div>
                </Poptip>
            </div>
            <div class="monster" :class="[gameCharacterVO.length == 1 ? 'monstercenter' : '']" v-if="this.teamStatus != 1">
                <Poptip trigger="hover" v-for="(item,idx) in gameCharacterVO" :key="idx" :title="item.name">
                    {{ idx == 0 ? '角色: ' : '真灵: ' }}{{ item.name || '' }}
                    <br>
                    <Progress :percent="(Math.max(item.health || auraLife, 0) / item.maxLife) * 100" style="width: 200px">
                        <span>{{ item.health || auraLife }}</span>
                    </Progress>
                    <div slot="content">
                        <p>生命值 {{ item.health }}</p>
                        <p>最小攻击力 {{ item.attackMin }}</p>
                        <p>最大攻击力 {{ item.attackMax }}</p>
                        <p>防御力 {{ item.defence }}</p>
                        <p>命中率 {{ item.hitRate * 100 }}%</p>
                        <p>闪避率 {{ item.evade * 100  || 0}}%</p>
                        <p>暴击率 {{ (item.criticalHitValue * 100).toFixed(2)  || 0}}%</p>
                        <p>暴击伤害 {{ (item.critDamage * 100).toFixed(2) || 0 }}%</p>
                        <p>行动速度 {{ item.movingSpeed }}</p>
                    </div>
                </Poptip>
            </div>

            <div class="recording" id="recording">
                <h2 style="margin-top: 20px" v-if="monsterList.length <= 0">暂无记录</h2>
                <battle-process v-bind:recordings="recording"></battle-process>
            </div>
        </div>

    </div>
</template>
         
  <script>
import battleProcess from "./battleProcess.vue";

import qs from "query-string";
import { sleep } from "../utils";

export default {
    components: { battleProcess },
    name: "tradingBank",
    data() {
        return {
            mapList: [],
            mapid: '',
            mapName: '',
            monsterList: [],
            gameCharacterVO: [],
            recording: [],
            teamStatus: 1
        };
    },
    created() {
        this.getMapList();

        setInterval(() => {
            this.getMapList();
        }, 1000000);
    },
    updated() {
        // 战斗记录定位到底部
        const el = document.getElementById("recording");
        const max = el.scrollHeight - el.clientHeight;
        if (max - el.scrollTop < 80) {
            el.scrollTop = el.scrollHeight;
        }
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
            } else if (color == 7) {
                return "orange";
            } else {
                return "#ff00c3e0";
            }
        },
        getMapList() {
            this.$http.get("gameLockDemon/queryMapList").then((res) => {
                this.mapList = res.data.data;
                if (!res.data.data) {
                    this.$Message.success(res.data.msg);
                }
            }).catch((err) => {
                this.$Message.warning("获取锁妖塔地图失败,请联系管理员");
            });
        },
        async startTheBattle() {
            this.mapid = '';
            for (var i = 0; i < this.mapList.length; i++) {
                if (this.mapName == this.mapList[i].mapName) {
                    this.mapid = this.mapList[i].mapId;
                }
            }

            try {
                const res = await this.$http.post(
                    `gameLockDemon/checkMapGenMon?${qs.stringify({
                        charaId: this.getCookie("charaId"),
                        mapId: this.mapid,
                    })}`
                );

                if (res.data.msg != 'OK') {
                    this.$Message.warning(res.data.msg);
                    return
                }

                const body = res.data;

                if (body.status == 205) {
                    this.$Message.warning(body.msg);
                    // await sleep(6000);
                    // this.startTheBattle();
                    return;
                }

                const combatResults = body.data
                // 按顺序播报所有的战斗结果
                for (let i = 0; i < combatResults.length; i++) {
                    await this.boardcastCombatResult(combatResults[i]);
                }
            } catch (err) {
                this.$Message.warning("战斗开始失败,请联系管理员");
            }

        },

        // 播报战斗返回的日志
        async boardcastCombatResult(result) {
            this.monsterList = result.gameMonList; // 怪物信息
            this.monsterList.forEach((m) => (m.maxLife = m.life));
            if (this.teamStatus != 1) {
                if (result.gamePetVO) {
                    result.gamePetVO = {
                        health: result.gamePetVO.life,
                        name: result.gamePetVO.auraName,
                        ...result.gamePetVO
                    }
                    this.gameCharacterVO = [result.gameCharacterVO, result.gamePetVO]; // 人物 加宠物信息信息
                    this.gameCharacterVO.forEach((m) => (m.maxLife = m.health));
                } else {
                    this.gameCharacterVO = [result.gameCharacterVO]; // 人物加信息
                    this.gameCharacterVO.forEach((m) => (m.maxLife = m.health));
                }
            }
            let combatInfo = result.combatInfo;
            for (let i = 0; i < combatInfo.length; i++) {
                // 每条记录的输出相隔 1.5 秒
                await sleep(1500);
                const record = combatInfo[i];
                const targets = this.monsterList.filter(
                    (m) => m.name == record.hinjured
                );
                if (targets.length > 0) {
                    targets[0].life = record.surplusHealth;
                }

                if (this.teamStatus != 1) {
                    const targets1 = this.gameCharacterVO.filter(
                        (m) => m.name == record.hinjured
                    );
                    if (targets1.length > 0) {
                        targets1[0].health = record.surplusHealth;
                    }
                }
                // 1 物理伤害 2灵力伤害 3持续伤害 4恢复血量
                // console.log(record, "战斗日志");
                if (record.hurtType == 5) {
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
                    // 通知技能刷新检查
                    this.$bus.$emit("skillMsg", "锁妖塔发送过来的");
                    this.$bus.$emit("synthesisMsg", "锁妖塔发送过来的");
                }
            }
        },
        addRecord(record) {
            this.recording.push(record);
            // 战斗记录超过一定条数时移除一部分
            if (this.recording.length >= 200) {
                this.recording.splice(0, 100);
            }
        },
    },
};
  </script>
  
  <!-- Add "scoped" attribute to limit CSS to this component only -->
  <style scoped>
.knapsack > div {
    margin-right: 20px;
    margin-bottom: 20px;
}

.Equipment p {
    color: white;
}

/* 地图和战斗场景 */
.monster {
    display: flex;
    justify-content: space-between;
    /* margin-bottom: 40px; */
    margin-bottom: 8px;
    margin-top: 8px;
}
.monstercenter {
    justify-content: center;
}
/*  战斗场景  */
.recording {
    max-height: 174px;
    overflow: auto;
    transition: all 0.4s;
}
</style>