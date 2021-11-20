<template>
  <div class="container">
    <div class="row register-page">
      <div class="error">{{ errorMessage }}</div>
      <form class="col s12" id="reg-form">
        <div class="row">
          <div class="error">{{ errorOfName }}</div>
          <div class="input-field col s6">
            <input
              id="last_name"
              type="text"
              class="validate"
              v-model="lastName"
              required
            />
            <label for="last_name">姓</label>
          </div>
          <div class="input-field col s6">
            <input
              id="first_name"
              type="text"
              class="validate"
              v-model="firstName"
              required
            />
            <label for="first_name">名</label>
          </div>
        </div>
        <div class="row">
          <div class="error">{{ errorOfMailAddress }}</div>
          <div class="input-field col s12">
            <input
              id="email"
              type="email"
              class="validate"
              v-model="mailAddress"
              required
            />
            <label for="email">メールアドレス</label>
          </div>
        </div>
        <div class="row">
          <div class="error">{{ errorOfPassword }}</div>
          <div class="input-field col s12">
            <input
              id="password"
              type="password"
              class="validate"
              minlength="8"
              v-model="password"
              required
            />
            <label for="password">パスワード</label>
          </div>
        </div>
        <div class="row">
          <div class="input-field col s12">
            <input
              id="confirmPassword"
              type="password"
              class="validate"
              minlength="8"
              v-model="confirmPassword"
              required
            />
            <label for="confirmPassword">確認用パスワード</label>
          </div>
        </div>
        <div class="row">
          <div class="input-field col s6">
            <input
              id="zipCode"
              type="text"
              class="validate"
              minlength="8"
              v-model="zipCode"
              required
            />
            <label for="zipCode">郵便番号(ハイフンあり)</label>
          </div>
          <div class="input-field col s6">
            <button
              class="btn btn-small btn-register waves-effect waves-light"
              type="button"
              v-on:click="searchAddress"
            >
              住所検索
            </button>
          </div>
        </div>
        <div class="row">
          <div class="input-field col s12">
            <input
              placeholder="住所が自動で入ります"
              id="address"
              type="text"
              class="validate"
              minlength="8"
              v-model="address"
              required
            />
            <label for="address" class="active">住所</label>
          </div>
        </div>
        <div class="row">
          <div class="input-field col s6">
            <button
              class="btn btn-large btn-register waves-effect waves-light"
              type="button"
              v-on:click="registerAdmin"
            >
              登録
              <i class="material-icons right">done</i>
            </button>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import config from "@/const/const";
import axios from "axios";
// [npm install axios-jsonp]が必要
// import axiosJsonpAdapter from "axios-jsonp";

/**
 * 管理者情報を登録する画面.
 */
@Component
export default class RegisterAdmin extends Vue {
  // 姓名エラーメッセージ
  private errorOfName = "";
  // メールアドレスエラーメッセージ
  private errorOfMailAddress = "";
  // パスワードエラーメッセージ
  private errorOfPassword = "";
  // 登録失敗時のエラーメッセージ
  private errorMessage = "";
  // 姓
  private lastName = "";
  // 名
  private firstName = "";
  // メールアドレス
  private mailAddress = "";
  // パスワード
  private password = "";
  // 確認用パスワード
  private confirmPassword = "";
  // 郵便番号
  private zipCode = "";
  // 住所
  private address = "";

  /**
   * 管理者情報を登録する.
   *
   * @remarks
   * 本メソッドは非同期でWebAPIを呼び出し管理者登録をするためasync/await axiosを利用しています。これらを利用する場合は明示的に戻り値にPromiseオブジェクト型を指定する必要があります。
   * @returns Promiseオブジェクト
   */
  async registerAdmin(): Promise<void> {
    // 入力値エラーチェックし、エラーが１つ以上あれば処理を止める
    if (this.hasErrors()) {
      return;
    }

    // 管理者登録処理
    const response = await axios.post(`${config.EMP_WEBAPI_URL}/insert`, {
      name: this.lastName + " " + this.firstName,
      mailAddress: this.mailAddress,
      password: this.password,
      address: this.address,
    });
    console.dir("response:" + JSON.stringify(response));
    if (response.data.status === "success") {
      // 成功ならログイン画面に遷移する
      this.$router.push("/loginAdmin");
    } else {
      // 失敗ならエラーメッセージを表示する
      this.errorMessage = "登録できませんでした(" + response.data.message + ")";
    }
  }

  /**
   * 郵便番号から住所を検索する.
   */
  async searchAddress(): Promise<void> {
    // eslint-disable-next-line @typescript-eslint/no-var-requires
    const axiosJsonpAdapter = require("axios-jsonp");

    const response = await axios.get("https://zipcoda.net/api", {
      adapter: axiosJsonpAdapter,
      params: {
        zipcode: this.zipCode,
      },
    });
    console.dir(JSON.stringify(response));
    this.address = response.data.items[0].address;
  }

  /**
   * エラーチェック処理.
   *
   * @returns エラーがある:true / エラーがない:false
   */
  private hasErrors(): boolean {
    let hasError = false;
    this.errorOfName = "";
    this.errorOfMailAddress = "";
    this.errorOfPassword = "";

    if (this.lastName === "" || this.firstName === "") {
      this.errorOfName = "姓または名が入力されていません";
      hasError = true;
    }
    if (this.mailAddress === "") {
      this.errorOfMailAddress = "メールアドレスが入力されていません";
      hasError = true;
    }
    if (this.password === "") {
      this.errorOfPassword = "パスワードが入力されていません";
      hasError = true;
    } else if (this.password !== this.confirmPassword) {
      // パスワード一致チェック
      this.errorOfPassword = "パスワードが不一致です";
      hasError = true;
    }

    return hasError;
  }
}
</script>

<style scoped>
.register-page {
  width: 600px;
}
</style>
