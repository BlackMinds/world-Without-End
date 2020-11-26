<template>
  <!-- 商店 -->
  <Card>
    <p slot="title">商店</p>
    <div slot="extra"></div>
    <div>
      <div class="knapsack">
        <Poptip
          trigger="hover"
          v-for="item in shopList"
          :key="item.marketName"
          :title="item.marketName"
        >
          <p
            style="white-space: nowrap; height: 24px"
            :style="{ color: distinguishColor(item.color) }"
          >
            {{ item.marketName }}
            <span class="jjt_smail">{{ item.marketNum }}</span>
            <Button
              @click.prevent="purchaseItem(item)"
              size="small"
              type="info"
            >
              购买
            </Button>
          </p>
          <div slot="content">
            <p v-if="item.marketPrice">
              商品价格 {{ item.marketPrice }}
              {{ item.priceType == 0 ? "铜钱" : "金币" }}
            </p>
            <p v-if="item.marketNum">购买次数 {{ item.marketNum }}</p>
            <p v-if="item.marketLevel">
              商品最低购买等级 {{ item.marketLevel }}
            </p>
            <p v-if="item.typeDec">装备类型 {{ item.typeDec }}</p>
            <p v-if="item.level">装备等级 {{ item.level }}</p>
            <p v-if="item.itemNum">物品数量： {{ item.itemNum }}</p>
            <p v-if="item.life">生命值 {{ item.life }}</p>
            <p v-if="item.mana">真气值 {{ item.mana }}</p>
            <p v-if="item.attack">攻击力 {{ item.attack }}</p>
            <p v-if="item.magAttack">法术攻击力 {{ item.magAttack }}</p>
            <p v-if="item.defense">防御力 {{ item.defense }}</p>
            <p v-if="item.critical">暴击率 {{ item.critical * 100 }}%</p>
            <p v-if="item.speed">行动速度 {{ item.speed }}</p>
            <p v-if="item.physique">体格 {{ item.physique }}</p>
            <p v-if="item.dexterous">灵巧 {{ item.dexterous }}</p>
            <p v-if="item.spirit">灵力 {{ item.spirit }}</p>
            <p v-if="item.marketDec">物品描述： {{ item.marketDec }}</p>
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
      shopList: [], // 商店列表
    };
  },
  created() {
    this.waresList();
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
    // 获取商店列表
    waresList() {
      this.$http
        .post("/gameMarket/waresList?charaId=" + this.getCookie("charaId"))
        .then((res) => {
          this.shopList = res.data.data;
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

<style scoped>
/* knapsack 背包 和商店共用的 */
.knapsack {
  /* max-height: 516px; */
}
.knapsack /deep/ .ivu-poptip-popper {
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
</style>
