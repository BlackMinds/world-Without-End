<template>
    <!-- <Card> -->
    <!-- {{ tradingBankList }} -->
    <!-- <p slot="title">黑市</p> -->
    <div>
        商品名称: <Input v-model="bussinessName" @input="getTradingBankList()" placeholder="" clearable style="margin-bottom:20px;" />
        <div class="knapsack">
            <Poptip trigger="hover" v-for="item in tradingBankList" :key="item.name" :title="item.name">
                <p style="white-space: nowrap; height: 24px" :style="{ color: distinguishColor(item.color) }">
                    {{ item.busName }}
                    <span>
                        {{ item.price }}$
                    </span>
                    <Button @click.prevent="purchase(item)" size="small" type="info">
                        购买
                    </Button>
                </p>
                <div slot="content" class="poptipExplain">
                    <div></div>
                    <div class="Equipment">
                        <p :style="{ color: distinguishColor(item.color) }">
                            {{ item.busName }}
                        </p>
                        <p>交易状态 {{ item.busStatus }}</p>
                        <p>售卖时间 {{ item.saleTime }}</p>
                        <p>售价 {{ item.price }}</p>
                        <Divider v-if="item.sellerName" />
                        <p v-if="item.sellerName">描述： {{ item.sellerName }}</p>
                    </div>
                </div>
            </Poptip>
        </div>

        <Page :total="total" :page-size="pageSize" @on-change="changepage"></Page>
    </div>
    <!-- </Card> -->
</template>
         
  <script>
export default {
    name: "tradingBank",
    data() {
        return {
            tradingBankList: [],
            bussinessName: '',
            pageSize: 20,
            total: 0
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
        getTradingBankList(index = 1) {
            this.$http.post("gameBussiness/getBussinessList?page=" + index + "&pageSize=" + this.pageSize + '&bussinessName=' + this.bussinessName).then((res) => {
                console.error(res.data.rows)
                this.tradingBankList = res.data.rows;
                this.total = res.data.records
            }).catch((err) => {
                this.$Message.warning("获取黑市失败,请联系管理员");
            });
        },
        changepage(index) {
            this.getTradingBankList(index)
        },
        pageSizeChange(index) {
            console.error(index)
            this.getTradingBankList()
        },
        purchase(item) {
            this.$http.post(
                "/gameBussiness/playersBuyItem/?charaId=" +
                this.getCookie("charaId") +
                "&buyId=" +
                item.busId
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
    margin-right: 20px;
    margin-bottom: 20px;
}

.Equipment p {
    color: white;
}
</style>
  