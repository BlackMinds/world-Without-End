<template>
  <div class="headers">
    <div class="head-cen">
      <div class="heade-left">
        <ul>
          <li>角色选择(暂时不可用)</li>
          <li>设置(暂时不可用)</li>
        </ul>
      </div>
      <div class="heade-right">
        <ul>
          <li @click="out">退出登录</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "headers",
  data() {
    return {
      
    };
  },
  methods: {
   out () {
     this.$router.push({name: "Login"});
   }
  },
};
</script>

<style scoped>
.headers {
  height: 60px;
  line-height: 60px;
  min-width: 1140px;
  color: #fff;
  /* backdrop-filter: blur(20px); */
  background: rgba(95, 95, 95, 0.3);
  box-shadow: 3px 3px 6px 3px rgba(0, 0, 0, 0.3);
  margin-bottom: 10px;
}

.head-cen {
  width: 1140px;
  margin: 0 auto;
  overflow: hidden;
}
.heade-left {
  float: left;
}
.heade-left ul,.heade-right ul {
  overflow: hidden;
}
.heade-left ul li,.heade-right ul li {
  float: left;
  height: 60px;
  padding: 0 10px;
  cursor: pointer;
}
.heade-left ul li:hover,.heade-right ul li:hover{
  background: #666;
  transition: all .6s;
}
.heade-right {
  float: right;
}
</style>
