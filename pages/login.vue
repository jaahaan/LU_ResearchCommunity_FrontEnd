<template>
  <div class="login-section">
    <div class="left">
      <img :src="'/assets/images/lurc.png'" alt="img" />
    </div>

    <!-- Two Factor Form -->
    <div class="right" v-if="!isLoggingBlock">
      <div class="container-fluid">
        <div class="container-fluid rt col-10 p-2">
          <h1 class="text-center mb-2">Two Factor Authentication</h1>
          <div class="alert alert-dark" v-if="msg">
            {{ msg }}
          </div>

          <div class="mb-2">
            Enter OTP
            <input
              type="number"
              class="form-control"
              v-model="data.twoFactorCode"
            />
            <span class="text-danger" v-if="errors.twoFactorCode">{{
              errors.twoFactorCode[0]
            }}</span
            ><span class="text-danger" v-if="errmsg">{{ errmsg }}</span>
          </div>

          <div class="mb-2">
            <button
              :class="[
                data.twoFactorCode
                  ? 'main-btn-change col-12'
                  : 'main-btn col-12',
                ' main-btn col-12',
              ]"
              @click="submit"
              :disabled="isSubmitting"
              :loading="isSubmitting"
            >
              {{ isSubmitting ? "Submitting..." : "Submit OTP" }}
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Login Form -->
    <div class="right" v-else-if="isLoggingBlock">
      <div class="form">
        <h1>Login</h1>
        <div class="mb-2">
          Email
          <input type="email" v-model="data.email" placeholder="Email" />
          <span class="text-danger" v-if="errors.email">{{
            errors.email[0]
          }}</span>
        </div>

        <div class="mb-2">
          Password
          <input
            type="password"
            v-model="data.password"
            placeholder="Password"
          />
          <div class="d-block">
            <span class="w-full text-danger float-start" v-if="errors.password"
              >{{ errors.password }}
            </span>

            <nuxt-link class="float-end" to="/auth/forgot_password"
              >Forgot Password</nuxt-link
            >
          </div>
        </div>

        <div class="mb-2">
          <button
            type="button"
            :class="[
              data.email && data.password
                ? ' main-btn-change col-12'
                : ' main-btn col-12',
              ' main-btn  col-12',
            ]"
            @click="login"
            :disabled="isLogging"
            :loading="isLogging"
          >
            {{ isLogging ? "Logging In.." : "Login" }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "login",
  middleware: "guest",

  data() {
    return {
      data: {
        email: "",
        password: "",
        twoFactorCode: "",
      },
      msg: null,
      errmsg: null,

      isLoggingBlock: true,
      isLogging: false,
      isSubmitting: false,
      errors: [],
      props: ["msg"],
    };
  },
  methods: {
    async login() {
      // if (this.data.email.trim() == "")
      //     return this.e("Email is required");
      // if (this.data.password.trim() == "")
      //     return this.e("Password is required");
      this.isLogging = true;
      this.errors = [];
      //if we put await in front of it, it will be a response! The await keyword causes code to wait for that Promise to resolve. And whatever data is normally passed to your callback as an argument is instead returned. There is still an asynchronous AJAX call happening, but our code reads a bit more like synchronous code.
      const res = await this.callApi("post", "/api/login", this.data);

      if (res.status == 200) {
        console.log(res.data.user);
        console.log(res.data.token);

        this.$store.dispatch("setAuthInfo", res.data.user);
        this.$store.dispatch("setToken", res.data.token);
        this.setCookie("token", res.data.token);
        // this.$router.push(`/home`);
        this.$router.go(`/home`);

        this.s("You are logged In");
        this.getNotificationItemsServer();
        if (this.callNotificationOb) {
          let notification = this.callNotificationOb;

          return;
        }
        if (this.$route.query.callback)
          return this.$router.push(this.$route.query.callback);
        // window.location = "/";
      } else {
        if (res.status == 400) {
          // this.msg = res.data.msg;
          this.e(res.data.error);
        } else if (res.status == 401) {
          this.errors.password = res.data.msg;
          // this.e(res.data.msg);
        } else if (res.status == 422) {
          if (res.data.email) {
            this.errors.email = res.data.email[0];
            // this.e(res.data.email[0]);
          }
          if (res.data.password) {
            this.errors.password = res.data.password[0];
            // this.e(res.data.password[0]);
          }
        } else if (res.status == 402) {
          let emailPassword = {
            email: this.data.email,
            password: this.data.password,
          };
          this.$store.commit("setUnauthorizedCredential", emailPassword);
          this.e(res.data.msg);
          this.$router.push(`/auth/account-activation`);
        } else {
          this.swr();
        }
      }
      this.isLogging = false;
    },

    async submit() {
      this.isSubmitting = true;
      const res = await this.callApi(
        "post",
        "/api/submit_twoFactor_otp",
        this.data
      );

      if (res.status == 200) {
        this.s(res.data.msg);
        window.location = "/";
        this.$router.go();
        // window.location.reload();
        this.errors = [];

        //this.data.otp = "";
      } else {
        if (res.status == 401) {
          this.errmsg = res.data.msg;
          this.isLoggingBlock = false;
        } else if (res.status == 402) {
          this.errmsg = res.data.msg;
          this.isLoggingBlock = true;
        } else if (res.status == 422) {
          for (let i in res.data.errors) {
            this.errors = res.data.errors;
            // this.e(res.data.errors[i][0]);
          }
        } else {
          this.swr();
        }
      }
      this.isSubmitting = false;
    },
  },

  // created(){
  //     if(this.authInfo) this.$router.push('/')
  // }
};
</script>
<style scoped>
input {
  display: block;
  padding: 6px 15px;
  width: 100%;
  box-sizing: none;
  border: 1px solid #fff;
  border-radius: 5px;
  color: #555;
}
</style>
