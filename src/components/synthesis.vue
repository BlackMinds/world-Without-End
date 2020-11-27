<template>
  <!-- 合成 -->
  <Card>
    <p slot="title">合成</p>
    <div slot="extra"></div>
    <div>
      <div class="knapsack">
        <Poptip
          trigger="hover"
          v-for="item in synthesisList"
          :key="item.syntheticName"
          :title="item.syntheticName"
        >
          <p
            style="white-space: nowrap; height: 24px"
            :style="{ color: distinguishColor(item.color) }"
          >
            {{ item.syntheticName }}
            <Button
              @click.prevent="synthesisItem(item)"
              size="small"
              type="info"
            >
              合成
            </Button>
          </p>
          <div slot="content">
            <p v-if="item.syntheticDesci">
              物品描述: {{ item.syntheticDesci }}
            </p>
            <p v-if="item.syntheticExp">
              合成消耗的经验: {{ item.syntheticExp }}
            </p>
            <p v-if="item.syntheticMoney">
              合成消耗的铜钱: {{ item.syntheticMoney }}
            </p>
            <p v-if="item.syntheticRealm">
              合成物品最低境界:
              <span style="color: rgb(45, 196, 255)">
                {{ item.syntheticRealm }}
              </span>
            </p>
            <p v-if="item.materOneName">
              {{ item.materOneName }}: 需要
              {{ item.materOneNum }}
            </p>
            <p v-if="item.materTwoName">
              {{ item.materTwoName }}: 需要
              {{ item.materTwoNum }}
            </p>
            <p v-if="item.materThreeName">
              {{ item.materThreeName }}: 需要
              {{ item.materThreeNum }}
            </p>
            <p v-if="item.coin">
              需要 {{ item.coninType == 0 ? "铜币" : "金币" }}
              {{ item.coin }}
            </p>
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
      synthesisList: [], // 合成列表
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
        return "orange";
      }
    },
    // 获取合成列表
    waresList() {
      this.$http.post("/gameRealm/getSyntheticList").then((res) => {
        this.synthesisList = res.data.data;
      });
    },
    // 合成
    synthesisItem(item) {
      this.$http
        .post(
          "	/gameRealm/breakThroughTheSynthetic?charaId=" +
            this.getCookie("charaId") +
            "&syntheticId=" +
            item.syntheticId
        )
        .then((res) => {
          this.$Message.warning(res.data.data);
          this.$bus.$emit("synthesisMsg", "合成发送过来的");
        })
        .catch((err) => {
          this.$Message.warning("合成失败,请联系管理员");
        });
    },
  },
};
</script>

<style scoped>
/* knapsack 背包 和合成共用的 */
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
</style>
