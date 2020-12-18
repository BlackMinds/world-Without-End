<template>
  <div class="hello">
    <Card class="record" shadow v-if="cut">
      <p slot="title">登陆</p>
      <Form :model="formlogin" :label-width="68">
        <FormItem label="账号">
          <Input
            placeholder="请输入 6~20 位的帐号"
            v-model="formlogin.username"
          />
        </FormItem>
        <FormItem label="密码">
          <Input
            type="password"
            placeholder="请输入 6~20 位的密码"
            v-model="formlogin.password"
          />
        </FormItem>
      </Form>
      <Button type="primary" long @click="enroll">登陆</Button>
      <div class="go-register" @click="cut = false">
        或
        <a href="#">注册</a>
      </div>
    </Card>
    <Card class="record" shadow v-else>
      <p slot="title">注册</p>
      <Form :model="formRegister" :label-width="68">
        <FormItem label="账号">
          <Input
            placeholder="请输入 6~20 位的帐号"
            v-model="formRegister.username"
          />
        </FormItem>
        <FormItem label="密码">
          <Input
            placeholder="请输入 6~20 位的密码"
            v-model="formRegister.password"
            type="password"
          />
        </FormItem>
        <FormItem label="确认密码">
          <Input
            placeholder="请再输入一次密码"
            v-model="formRegister.confirmPassword"
            type="password"
          />
        </FormItem>
      </Form>
      <Button type="primary" long @click="register">注册</Button>
      <div class="go-register" @click="cut = true">
        或
        <a href="#">登录</a>
      </div>
    </Card>
  </div>
</template>

<script>
export default {
  name: "Login",
  data() {
    return {
      cut: true,
      ruleValidate: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
        ],
        password: [
          { required: true, message: "请输入密码", trigger: "blur" },
          {
            type: "string",
            min: 6,
            message: "请输入6位数以上的密码",
            trigger: "blur",
          },
        ],
      },
      formRegister: {
        username: undefined,
        password: undefined,
      },
      formlogin: {
        username: undefined,
        password: undefined,
        confirmPassword: undefined,
      },
    };
  },
  methods: {
    enroll() {
      this.$http.post("gamepassport/login", this.formlogin).then((res) => {
        if (res.data.msg != "OK") {
          console.log("看起来是账号密码错误了");
          this.$Message.warning(res.data.msg);
          return;
        }

        if (res.data.data.userId) {
          this.$Message.warning("登录成功");
          this.setCookie("userId", res.data.data.userId, 3);
          this.setCookie("charaId", res.data.data.charaId, 3);
          // console.log(res.data.data);

          this.$router.push({ name: "index" });
        }
      });
    },
    register() {
      if (this.formRegister.confirmPassword != this.formRegister.password) {
        this.$Message.warning("2次输入密码不同");
        return;
      }

      this.$http.post("gamepassport/regist", this.formRegister).then((res) => {
        if (res.data.msg != "ok") {
          this.$Message.warning(res.data.msg);
          return;
        }
      });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.hello {
  height: 100vh;
  background-image: url(https://s3.ax1x.com/2020/12/10/rFnfDU.jpg);
  background-repeat: no-repeat;
  background-size: cover;
  background-position: center center;
}
.record {
  width: 400px;
  position: absolute;
  left: 50%;
  top: 50%;
  margin-left: -200px;
  margin-top: -175px;
  backdrop-filter: blur(20px);
}
.record :global(.ivu-card-body) {
  padding: 24px 32px;
}
.go-register {
  margin-top: 8px;
}
.go-register a {
  color: #1890ff;
}
.form input {
  margin: 15px 0;
  height: 20px;
}
.form input:focus {
  box-shadow: 0 0 5px skyblue;
  border: 2px solid skyblue;
}
.button {
  width: 104px;
  margin: 20px auto 0;
}
.button button {
  border: 1px solid #333;
  background: #fff;
  outline: none;
  padding: 3px 6px;
  font-size: 14px;
}
.button button:hover {
  cursor: pointer;
  box-shadow: 0 0 5px skyblue;
  border: 1px solid skyblue;
  transition: all 0.4s;
}
.center1 {
  text-align: center;
}
</style>
