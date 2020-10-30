<template>
  <Card>
    <p slot="title">在线玩家列表</p>
    <div slot="extra"></div>
    <div>
      <div class="playersList">
        <Poptip
          trigger="hover"
          v-for="(item, idx) in playersList"
          :key="idx"
          :title="item.name"
        >
          <p style="white-space: nowrap; height: 24px">
            <span>{{ ++idx }}:{{ item.name }}:</span>
          </p>
          <div slot="content">
            <p>等级: {{ item.level }}</p>
            <p>攻击力: {{ item.attack }}</p>
            <p>防御力: {{ item.defense }}</p>
            <p>生命值 {{ item.health }}</p>
            <p>真气值: {{ item.mana }}</p>
            <p>当前经验值: {{ item.exp }}</p>
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
      playersList: [], // 在线玩家列表
    };
  },
  created() {
    this.getPlaters();

    setInterval(() => {
      this.getPlaters();
    }, 1000000);
  },
  methods: {
    // 列表
    getPlaters() {
      // 在线玩家列表
      this.$http.post("/gamepassport/playerOnline").then((res) => {
        this.playersList = res.data.data;
      });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
/* 在线玩家列表 */
.playersList {
  max-height: 306px;
  overflow: auto;
}
.playersList /deep/ .ivu-poptip {
  display: block;
}
</style>
