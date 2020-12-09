<template>
  <div>
    <p v-for="(item, index) in recordings" :key="index">
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
        <!-- 持续伤害展示的地方 -->
        <template v-if="item.hinjuredConmbatList">
          <p
            v-for="(continued, idx) in item.hinjuredConmbatList"
            :key="idx"
          >
            <span>{{ continued.attacker }}</span>
            使用{{ continued.skillName }} 造成了
            <span
              :style="{
                color: '#5b00ff',
                fontSize: continued.isCritical == 0 ? '14px' : '20px',
              }"
            >
              {{ continued.hurt }}
            </span>
            剩余
            <span style="color: red">
              {{ continued.surplusHealth }}
            </span>
            血量
          </p>
        </template>
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
          用{{ item.skillName }}
          帮
          <span>{{ item.hinjured }}</span>
          回复
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
  </div>
</template>
       
<script>
export default {
  name: "battleProcess",
  props: ["recordings"],
  data() {
    return {
    };
  },
  watch: {
    recordings(newValue, oldValue) {
        this.recordings = newValue
    }
  },
  methods: {
  
  },
};
</script>

<style scoped>

</style>
