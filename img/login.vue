<template>
  <b-container class="no-banner">
    <b-row align-h="center">
      <b-col xl="6" lg="8" sm="12">
        <b-card header="登录 - Admin">
          <b-form @submit="onSubmit" >
            <b-form-group class="mr-5" label-align-md="right" label-cols-md="2" label="邮箱:" label-for="input-email" description="示例: example@gmail.com">
              <b-form-input id="input-email" v-model="form.email" type="email" required placeholder="输入邮箱"></b-form-input>
            </b-form-group>

            <b-form-group class="mr-5" label-align-md="right" label-cols-md="2" label="密码:" label-for="input-password" description="密码区分大小写">
              <b-form-input id="input-password" v-model="form.password" type="password" required placeholder="输入密码"></b-form-input>
            </b-form-group>
            <div class="text-center">
              <b-button type="submit" variant="outline-success">登录</b-button>
              <b-button @click="onCancel" variant="outline-danger">取消</b-button>
            </div>
          </b-form>
          <!-- <b-card class="mt-3" header="Form Data Result">
            <pre class="m-0">{{ form }}</pre>
          </b-card> -->
        </b-card>
      </b-col>
    </b-row>

  </b-container>
</template>

<script>
import Cookies from "js-cookie";

export default {
  // 简单验证用户状态即可
  async asyncData({ app, store, redirect, req }) {
    const token = app.$getCookiesInServer(req).token;
    // 已经登录则跳转到后台
    if (token) {
      try {
        const user = await app.$api.auth();
        if (user) {
          store.commit("setUser", user);
          redirect("/admin/dashboard");
        }
      } catch (e) {}
    }
  },
  data: () => ({
    form: {
      email: "",
      password: ""
    }
  }),
  computed: {
    user() {
      return this.$store.state.user;
    }
  },
  methods: {
    makeToast(title, message, variant = null) {
      this.$bvToast.toast(message, {
        title: title,
        variant: variant,
        solid: true
      });
    },
    async onSubmit(evt) {
      evt.preventDefault();
      try {
        let result = await this.$api.login(this.form.email, this.form.password);
        if (result) {
          Cookies.set("token", result.token);
          this.$store.commit("setUser", result);
          this.$router.push("/admin/dashboard");
        }
      } catch (e) {
        this.makeToast("登录失败", "账户或密码有误", "danger");
        return;
      }
    },
    onCancel(evt) {
      this.$router.push("/post/1");
    }
  },
  mounted() {
    console.log(this.user);
  }
};
</script>