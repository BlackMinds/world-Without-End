<template>
    <Card>
        <p slot="title">排行榜</p>
        <div slot="extra"></div>
        <div>
            <Tabs value="name1">
                <TabPane label="灵石" name="name1">
                    <p v-for="(item, idx) in leaderboardLst1" :key="idx">
                        {{ ++idx }}: {{ item.name }}: {{ item.coin }}
                    </p>
                </TabPane>
                <TabPane label="等级" name="name2" style="max-height: 424px; overflow: auto">
                    <p v-for="(item, idx) in leaderboardLst2" :key="idx">
                        {{ ++idx }}: {{ item.name }}: {{ item.level }}
                    </p>
                </TabPane>
                <TabPane label="铜币" name="name3">
                    <p v-for="(item, idx) in leaderboardLst3" :key="idx">
                        {{ ++idx }}: {{ item.name }}: {{ item.money }}
                    </p>
                </TabPane>
                <TabPane label="天榜" name="name4">
                    <p v-for="(item, idx) in leaderboardLst4" :key="idx">
                        {{ ++idx }}: {{ item.name }}: {{ item.realmName }}
                    </p>
                </TabPane>
                <TabPane label="地榜" name="name5">
                    <p v-for="(item, idx) in leaderboardLst5" :key="idx">
                        {{ ++idx }}: {{ item.name }}: {{ item.realmName }}
                    </p>
                </TabPane>
                <TabPane label="世界boss伤害榜(单次)" name="name6" class="boss">
                    <p v-for="(item, idx) in leaderboardLst6" :key="idx">
                        {{ ++idx }}: {{ item.name }}:
                        <span v-for="(k,i) in item.gameCharaLevelBoList" :key="k.totalDamage" :style="{ color: i==0?'transparent':'black',fontSize:  i==0? '30px' : '16px' }">
                            {{++i}}: {{k.name}} {{k.totalDamage}}
                        </span>
                    </p>
                </TabPane>
                <TabPane label="世界boss伤害榜(总)" name="name7" class="boss">
                    <p v-for="(item, idx) in leaderboardLst7" :key="idx">
                        {{ ++idx }}: {{ item.name }}:
                        <span v-for="(k,i) in item.gameCharaLevelBoList" :key="k.totalDamage" :style="{ color: i == 0 ?'transparent':'black',fontSize:  i==0? '30px' : '16px' }">
                            {{++i}}: {{k.name}} {{k.totalDamage}}
                        </span>
                    </p>
                </TabPane>
            </Tabs>
        </div>
    </Card>
</template>
       
<script>
export default {
    name: "rankingList",
    data() {
        return {
            leaderboardLst1: [], // 灵石排行榜列表
            leaderboardLst2: [], // 等级排行榜列表
            leaderboardLst3: [], // 铜币排行榜列表
            leaderboardLst4: [], // 天榜排行榜列表
            leaderboardLst5: [], // 地榜排行榜列表
            leaderboardLst6: [], // 世界boss伤害(单次)排行榜列表
            leaderboardLst7: [], // 世界boss伤害(总)排行榜列表
        };
    },
    created() {
        this.leader();
        setInterval(() => {
            this.leader();
        }, 1000000);
    },
    methods: {
        // 列表
        leader() {
            // 灵石排行榜
            this.$http.get("/gamepassport/coinRanking").then((res) => {
                this.leaderboardLst1 = res.data.data;
            });

            // 等级排行榜
            this.$http.get("/gamepassport/levelRanking").then((res) => {
                this.leaderboardLst2 = res.data.data;
            });
            // 铜币排行榜
            this.$http.get("/gamepassport/moneyRanking").then((res) => {
                this.leaderboardLst3 = res.data.data;
            });
            // 天榜排行榜
            this.$http.get("/gamepassport/realmHeavenList").then((res) => {
                this.leaderboardLst4 = res.data.data;
            });
            // 地榜排行榜
            this.$http.get("/gamepassport/realmLandList").then((res) => {
                this.leaderboardLst5 = res.data.data;
            });
            // 世界boss伤害榜(单次)
            this.$http.get("/gamepassport/oneDamageRanking").then((res) => {
                this.leaderboardLst6 = res.data.data;
            });
            // 世界boss伤害榜(总)
            this.$http.get("/gamepassport/totalDamageRanking").then((res) => {
                this.leaderboardLst7 = res.data.data;
            });
        },
    },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.boss p span {
    margin-left: 20px;
    display: block;

    background-image: linear-gradient(45deg, #ff6b6b, #8b78e6, #51cf66);
    -webkit-background-clip: text;
    text-align: center;
    font-size: 36px;
    font-weight: bold;
}
</style>
