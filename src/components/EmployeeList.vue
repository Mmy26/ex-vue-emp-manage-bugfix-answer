<template>
  <div class="container">
    <!-- パンくずリスト -->
    <nav>
      <div class="nav-wrapper">
        <div class="col s12 teal">
          <a class="breadcrumb">従業員リスト</a>
        </div>
      </div>
    </nav>
    <div class="searchForm">
      <form>
        <div class="error">{{ searchNameMessage }}</div>
        <div class="input-field col s6">
          <input id="searchName" type="text" v-model="searchName" />
          <label for="searchName">検索したい名前(部分一致検索)</label>
        </div>
        <div class="searchBtn">
          <button
            class="btn btn-register waves-effect waves-light"
            type="button"
            v-on:click="onSearchClick"
          >
            検索
          </button>
        </div>
      </form>
    </div>
    <div>従業員数:{{ getEmployeeCount }}人</div>
    <div class="row">
      <table class="striped">
        <thead>
          <tr>
            <th>名前</th>
            <th>入社日</th>
            <th>扶養人数</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="employee of currentEmployeeList" v-bind:key="employee.id">
            <td>
              <router-link :to="'/employeeDetail/' + employee.id">{{
                employee.name
              }}</router-link>
            </td>
            <td>{{ employee.formatHireDate }}</td>
            <td>{{ employee.dependentsCount }}人</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
// import { Employee } from "../types/employee";
import { Employee } from "@/types/employee";

@Component
export default class EmployeeList extends Vue {
  // 従業員一覧
  currentEmployeeList = [];
  // 検索文字列
  searchName = "";
  // 検索文字列メッセージ
  searchNameMessage = "";

  /**
   * あいまい検索をする.
   *
   * @remarks
   * Vuexストアから検索文字列に一致したものを絞り込んで出力します。
   * １件も検索されなかった場合は「１件もありませんでしたので全件表示します」
   * と出力して全件検索を行います。
   */
  onSearchClick(): void {
    this.currentEmployeeList = this["$store"].getters.getSearchEmployeeByName(
      this.searchName
    );

    if (this.searchName === "") {
      this.searchNameMessage = "１件もありませんでしたので全件表示します";
    }
  }

  /**
   * Vuexストアのアクション経由で非同期でWebAPIから従業員一覧を取得する.
   *
   * @remarks
   * Vueインスタンスが生成されたタイミングで
   * Vuexストア内のアクションを呼ぶ(ディスパッチする)。
   * ライフサイクルフックのcreatedイベント利用。
   *
   * 取得してからゲットするため、async awaitを利用している。
   */
  async created(): Promise<void> {
    await this["$store"].dispatch("getEmployeeList");

    // 非同期で外部APIから取得しているので、async/await使わないとGetterで取得できない
    this.currentEmployeeList = this["$store"].getters.getEmployees;
  }
  /**
   * 非同期で取得したVuexストア内の従業員数を取得し返す.
   *
   * @returns 従業員数
   */
  get getEmployeeCount(): number {
    return this["$store"].getters.getEmployeeCount;
  }
  /**
   * 非同期で取得したVuexストア内の従業員一覧を取得し返す.
   *
   * @returns 従業員一覧情報
   */
  get getEmployees(): Array<Employee> {
    this.currentEmployeeList = this["$store"].getters.getEmployees;
    return this.currentEmployeeList;
  }
}
</script>

<style scoped>
.searchForm {
  margin-bottom: 20px;
  width: 450px;
  margin: 0 auto;
}

.searchBtn {
  display: block;
  width: 150px;
  margin: 0 auto;
}
</style>
