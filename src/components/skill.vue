<template>
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
            <Button @click.prevent="skillEquip(item)" size="small" type="info"
              >装备</Button
            >
            <Button @click.prevent="upgrade(item)" size="small" type="info"
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
</template>
       
<script>
export default {
  name: "playersList",
  data() {
    return {
      activeList: [], // 穿戴的主动技能列表
      passiveList: [], // 穿戴的被动技能列表
      skillList: [], // 全部技能
    };
  },
  created() {
    this.waresList();
  },
  methods: {
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
    // 获取角色的技能列表
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
    // 商店购买
    purchaseItem(item) {
      this.$http
        .post(
          "/gameMarket/buyWares?charaId=" +
            this.getCookie("charaId") +
            "&waresId=" +
            item.marketId
        )
        .then((res) => {
          this.$Message.warning(res.data.data);
          this.waresList();
          this.$bus.$emit("shopMsg", "商店购买发送过来的");
        })
        .catch((err) => {
          this.$Message.warning("购买失败,请联系管理员");
        });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
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
</style>
