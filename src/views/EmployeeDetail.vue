<template>
  <div class="container">
    <!-- パンくずリスト -->
    <nav>
      <div class="nav-wrapper">
        <div class="col s12 teal">
          <router-link to="/employeeList" class="breadcrumb"
            >従業員リスト</router-link
          >
          <a class="breadcrumb">従業員詳細</a>
        </div>
      </div>
    </nav>
    <div class="row">
      <form>
        <fieldset>
          <legend>従業員情報</legend>
          <table>
            <tr>
              <th nowrap>従業員名</th>
              <td>
                <span v-text="currentEmployee.name"></span>
              </td>
            </tr>
            <tr>
              <th nowrap>写真</th>
              <td>
                <img v-bind:src="currentEmployeeImage" />
              </td>
            </tr>
            <tr>
              <th nowrap>性別</th>
              <td>
                <span v-text="currentEmployee.gender"></span>
              </td>
            </tr>
            <tr>
              <th nowrap>入社日</th>
              <td><span v-text="currentEmployee.formatHireDate"></span></td>
            </tr>
            <tr>
              <th nowrap>メールアドレス</th>
              <td>
                <span v-text="currentEmployee.mailAddress"></span>
              </td>
            </tr>
            <tr>
              <th nowrap>郵便番号</th>
              <td>
                <span v-text="currentEmployee.zipCode"></span>
              </td>
            </tr>
            <tr>
              <th nowrap>住所</th>
              <td>
                <span v-text="currentEmployee.address"></span>
              </td>
            </tr>
            <tr>
              <th nowrap>電話番号</th>
              <td>
                <span v-text="currentEmployee.telephone"></span>
              </td>
            </tr>
            <tr>
              <th nowrap>給料</th>
              <td><span v-text="currentEmployee.formatSalary"></span>円</td>
            </tr>
            <tr>
              <th nowrap>特性</th>
              <td>
                <span v-text="currentEmployee.characteristics"></span>
              </td>
            </tr>
            <tr>
              <th nowrap>扶養人数</th>
              <td>
                <div class="input-field col s12">
                  <div class="error">{{ errorMessage }}</div>
                  <input
                    id="dependentsCount"
                    type="text"
                    class="validate"
                    value="3"
                    v-model="currentDependentsCount"
                    required
                  />
                </div>
              </td>
            </tr>
          </table>

          <button
            class="btn btn-register waves-effect waves-light"
            type="button"
            v-on:click="update"
          >
            更新
          </button>
        </fieldset>
      </form>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import config from "@/const/const";
import { Employee } from "@/types/employee";
import axios from "axios";
import { parse } from "date-fns";

@Component
export default class EmployeeDetail extends Vue {
  // 従業員情報
  // 初期値で初期化しなければtemplateでidなどが存在しないということでエラーとなる
  private currentEmployee = new Employee(
    0,
    "XXXX",
    "/img/noImage.png",
    "XXXX",
    new Date(2020, 0, 1),
    "XXXX",
    "XXXX",
    "XXXX",
    "XXXX",
    0,
    "XXXX",
    0
  );
  // currentEmployee!: Employee;
  // エラーメッセージ
  private errorMessage = "";
  // 画像
  private currentEmployeeImage = "";
  // 扶養人数
  private currentDependentsCount = 0;

  /**
   * VuexストアのGetter経由で受け取ったリクエストパラメータのIDから１件の従業員情報を取得する.
   *
   * @remarks
   * Vueインスタンスが生成されたタイミングで
   * Vuexストア内のGetterを呼ぶ。
   * ライフサイクルフックのcreatedイベント利用
   */
  async created(): Promise<void> {
    // ログインしていなければログイン画面へ遷移
    if (this.$store.getters.getLoginStatus === false) {
      this.$router.push("/loginAdmin");
      return;
    }

    // 送られてきたリクエストパラメータのidをnumberに変換して取得する
    const employeeId = Number(this.$route.params.id);

    // VuexストアのGetter、getEmployeeById()メソッドに先ほど取得したIDを渡し、１件の従業員情報を取得し、戻り値をcurrentEmployee属性に代入する
    // this.currentEmployee = this.$store.getters.getEmployeeById(employeeId);

    const response = await axios.get(
      `${config.EMP_WEBAPI_URL}/employee/${employeeId}`
    );
    console.dir(JSON.stringify(response));
    let responseEmployee = response.data.employee;
    this.currentEmployee = new Employee(
      responseEmployee.id,
      responseEmployee.name,
      responseEmployee.image,
      responseEmployee.gender,
      new Date(responseEmployee.hireDate),
      responseEmployee.mailAddress,
      responseEmployee.zipCode,
      responseEmployee.address,
      responseEmployee.telephone,
      responseEmployee.salary,
      responseEmployee.characteristics,
      responseEmployee.dependentsCount
    );

    // 今取得した従業員情報から画像パスを取り出し、imgディレクトリの名前を前に付与(文字列連結)してcurrentEmployeeImage属性に代入する
    this.currentEmployeeImage = `${config.EMP_WEBAPI_URL}/img/${this.currentEmployee.image}`;

    // 今取得した従業員情報から扶養人数を取り出し、currentDependentsCount属性に代入する
    this.currentDependentsCount = this.currentEmployee.dependentsCount;
  }

  /**
   * 扶養人数を更新する.
   *
   * 本メソッドは非同期でWebAPIを呼び出し扶養人数を更新するためasync/await axiosを利用しています。これらを利用する場合は明示的に戻り値にPromiseオブジェクト型を指定する必要があります。
   * @returns Promiseオブジェクト
   */
  async update(): Promise<void> {
    const response = await axios.post(
      `${config.EMP_WEBAPI_URL}/employee/update`,
      {
        id: this.currentEmployee.id,
        dependentsCount: this.currentDependentsCount,
      }
    );
    console.dir("response:" + JSON.stringify(response));

    if (response.data.status === "success") {
      // 成功なら従業員一覧画面に遷移する
      this.$router.push("/employeeList");
    } else {
      // 失敗ならエラーメッセージを表示する
      this.errorMessage = "更新できませんでした(" + response.data.message + ")";
    }
  }
}
</script>
