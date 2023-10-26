<template>
    <!-- <Card> -->
        <!-- {{ tradingBankList }} -->
        <!-- <p slot="title">交易行</p> -->
        <div>
            <div class="knapsack">
                <Poptip trigger="hover" v-for="item in tradingBankList" :key="item.name" :title="item.name">
                    <p style="white-space: nowrap; height: 24px" :style="{ color: distinguishColor(item.color) }">
                        <span v-if="item.itemType != 3 && item.itemType != 4">
                            [LV:{{ item.level }}]
                        </span>
                        {{ item.busName }}
                        <Button @click.prevent="purchase(item)" size="small"  type="info">
                            购买
                        </Button>
                    </p>
                    <div slot="content" class="poptipExplain">
                        <div></div>
                        <div class="Equipment">
                            <p :style="{ color: distinguishColor(item.color) }">
                                {{ item.busName }}
                            </p>
                            <p>物品类型 {{ item.typeDec }}</p>
                            <p>物品等级 {{ item.level }}</p>
                            <p>物品状态 {{ item.bind == 0 ? "解绑" : "绑定" }}</p>
                            <p v-if="item.itemNum != 1">
                                物品数量： {{ item.itemNum }}
                            </p>
                            <Divider v-if="item.decs" />
                            <p v-if="item.decs">描述： {{ item.decs }}</p>
                        </div>
                    </div>
                </Poptip>
            </div>
        </div>
    <!-- </Card> -->
  </template>
         
  <script>
  export default {
    name: "tradingBank",
    data() {
      return {
        tradingBankList: []
      };
    },
    created() {
      this.getTradingBankList();
  
      setInterval(() => {
        this.getTradingBankList();
      }, 1000000);
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
        getTradingBankList() {
            this.$http.get("gameBussiness/getBussinessList?page=" + 1 + "&pageSize=" + 20 + '&bussinessName' + '').then((res) => {
                console.error(res.data.rows)
                this.tradingBankList = res.data.rows;
            })
            .catch((err) => {
                this.$Message.warning("获取交易行失败,请联系管理员");
            });
        },
        purchase(item) {
            this.$http.post(
                "/gameChara/startOffline/?charaId=" +
                this.getCookie("charaId") +
                "&buyId=" +
                item.buyId
            )
            .then((res) => {
                this.$Message.warning(res.data.msg);
                
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
  .knapsack > div {
    margin-right: 10px;
  }
  </style>
  