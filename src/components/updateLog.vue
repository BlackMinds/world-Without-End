<template>
  <!-- 日志 -->
  <div>
    <p class="importantFont">进群可以免费领取兑换码</p>
    <p class="importantFontRed">
      角色技能3个主动2个被动, 试炼副本掉落技能书 角色50级之后可以带3个被动
      前期35级前尽量别强化装备 用来升级技能或者后面用
    </p>
    <br />
    <p class="importantFontRed">
      组队的时候 必须满3个人才可进行战斗 组队只需要队长操作就行,
      队员不能在继续挂机普通图 队长战斗开始失败
      请看一下组队副本要求等级和自己队伍玩家的等级是否符合,组队副本每次奖励上限30次在金币下面可看
    </p>
    <br />
    <p class="importantFontRed">
      每升一级加3个属性点,装备品质分为,黑色,黄色,绿色,蓝色,紫色,红色,橘色
    </p>
    <br />
    <p class="importantFontRed">
      如果有任何bug 或者有任何意见 反馈qq群:
      <span class="importantFont">808647555</span>
    </p>
    <br />
    <p class="importantFontRed">地图掉落说明</p>
    <p class="importantFontRed">
      普通图只掉落除红色以下与地图同等级的装备，挑战掉落紫色以上同等级装备。地狱掉落技能
      材料副本掉落材料
    </p>
    <br />
    <p>地图掉落说明</p>
    <div class="mapLoots">
      <p v-for="(mapLoots, index) in mapLootsList" :key="index">
        {{ mapLoots.mapName }}:
        <template v-if="mapLoots.itemsVoList">
          <span v-for="(loot, idx) in mapLoots.itemsVoList" :key="idx">
            <template v-if="loot">
              <span :style="{ color: distinguishColor(loot.color) }">
                {{ loot.itemName }}
              </span>
            </template>
          </span>
        </template>
      </p>
    </div>
    <br />
    <p>测试版: v0.11.4(2020-12-14)</p>
    <p class="importantFont">1: 前20级怪物属性大幅度削弱</p>
    <p class="importantFont">
      2: 20级后的怪物血量大幅度提升,攻击下降,防御提升一点点
    </p>
    <br />
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
      3: 新增自动战斗 (修改了 已角色id记录最近的地图 不会出现换小号就不行的情况)
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
      新增：法系被动技能：冥思。 新增：物理主动技能：连斩，疾风斩，真气爆发。
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
  </div>
</template>
       
<script>
export default {
  name: "playersList",
  data() {
    return {
      mapLootsList: [],
    };
  },
  created() {},
  mounted() {
    this.$http
      .post("/gameChara/mapDropItems")
      .then((res) => {
        this.mapLootsList = res.data.data;
      })
      .catch((err) => {
        this.$Message.warning("获取地图掉落失败");
      });
  },
  methods: {
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
  },
};
</script>

<style scoped>
</style>
