<template>
  <div>ログアウト中・・・</div>
</template>
<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import config from "@/const/const";
import axios from "axios";

/**
 * ログアウトをする画面.
 */
@Component
export default class LogoutAdmin extends Vue {
  /**
   * ログアウトする.
   *
   * @remarks
   * 本メソッドは非同期でWebAPIを呼び出しログアウトをするためasync/await axiosを利用しています。これらを利用する場合は明示的に戻り値にPromiseオブジェクト型を指定する必要があります。
   * @returns Promiseオブジェクト
   */
  async created(): Promise<void> {
    console.log("logout!");

    const response = await axios.get(`${config.EMP_WEBAPI_URL}/logout`);
    console.dir("response:" + JSON.stringify(response));
    // ログアウト状態にする(stateのsessionStorage内のisLoginをfalseにする)
    this.$store.commit("logouted");
    // ログイン画面に遷移する
    this.$router.push("/loginAdmin");
  }
}
</script>
