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
    <div class="row">
      <ul class="pagination">
        <span v-if="currentPageNum == 1">
          <li class="disabled">
            <router-link to="/employeeList">
              <i class="material-icons">chevron_left</i>
            </router-link>
          </li>
        </span>
        <span v-if="currentPageNum != 1">
          <li>
            <router-link
              to="/employeeList"
              v-on:click.native="pagingLinkClick(currentPageNum - 1)"
            >
              <i class="material-icons">chevron_left</i>
            </router-link>
          </li>
        </span>
        <span v-for="pageNum of pageNums" v-bind:key="pageNum">
          <li v-if="pageNum == currentPageNum" class="active">
            <router-link
              to="/employeeList"
              v-on:click.native="pagingLinkClick(pageNum)"
            >
              {{ pageNum }}
            </router-link>
          </li>
          <li v-if="pageNum != currentPageNum" class="waves-effect">
            <router-link
              to="/employeeList"
              v-on:click.native="pagingLinkClick(pageNum)"
            >
              {{ pageNum }}
            </router-link>
          </li>
        </span>
        <span v-if="currentPageNum == maxPageNum">
          <li class="disabled">
            <router-link to="/employeeList">
              <i class="material-icons">chevron_right</i>
            </router-link>
          </li>
        </span>
        <span v-if="currentPageNum != maxPageNum">
          <li>
            <router-link
              to="/employeeList"
              v-on:click.native="pagingLinkClick(currentPageNum + 1)"
            >
              <i class="material-icons">chevron_right</i>
            </router-link>
          </li>
        </span>
      </ul>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { Employee } from "@/types/employee";
@Component
export default class EmployeeList extends Vue {
  // 従業員一覧
  currentEmployeeList: Array<Employee> = [];
  // // 従業員数
  // employeeCount = 0;
  // 検索文字列
  searchName = "";
  // 検索文字列メッセージ
  searchNameMessage = "";
  // １ページに表示される最大の従業員の数
  readonly MAX_EMPLOYEE_COUNT = 5;
  // 画面下に表示されるページ番号
  pageNums: Array<number> = [];
  // 現在表示されているページ数
  currentPageNum = 1;
  // 最大のページ数
  maxPageNum = 0;

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

    // 従業員一覧情報をVuexストアから取得
    // 非同期で外部APIから取得しているので、async/await使わないとGetterで取得できない
    // ページング機能実装のため最初の10件に絞り込み
    this.currentEmployeeList = this["$store"].getters.getAllEmployees;

    // 初期表示
    this.initialDisplay();

    // 画面下のページ数番号を作成
    // 全従業員数を取得
    let allEmployeeCount = this["$store"].getters.getAllEmployeeCount;

    // 下のページング用のリンク数値を作成する(例：22人の場合だと3ページ)
    this.createLinkNumberForPaging(allEmployeeCount);
    // this.maxPageNum = Math.ceil(allEmployeeCount / this.MAX_EMPLOYEE_COUNT);
    // for (let i = 1; i <= this.maxPageNum; i++) {
    //   this.pageNums.push(i);
    // }
  }
  /**
   * 現在表示されている従業員一覧の数を返す.
   *
   * @returns 現在表示されている従業員一覧の数
   */
  get getEmployeeCount(): number {
    return this.currentEmployeeList.length;
  }

  /**
   * あいまい検索をする.
   *
   * @remarks
   * Vuexストアから検索文字列に一致したものを絞り込んで出力します。
   * １件も検索されなかった場合は「１件もありませんでしたので全件表示します」
   * と出力して全件検索を行います。
   */
  onSearchClick(): void {
    // 入力された文字列で絞り込みを行う
    this.currentEmployeeList = this["$store"].getters.getSearchEmployeeByName(
      this.searchName
    );

    // 検索されない場合は、メッセージを出して全件検索を行う
    if (this.currentEmployeeList.length == 0 || this.searchName === "") {
      this.searchNameMessage = "１件もありませんでしたので全件検索します";
      this.currentEmployeeList = this["$store"].getters.getAllEmployees;
      this.searchName = "";
    }
    // 下のページング用のリンク数値を作成する(例：22人の場合だと3ページ)
    this.createLinkNumberForPaging(this.currentEmployeeList.length);

    // 初期表示
    this.initialDisplay();
  }

  /**
   * ページネーション.
   *
   * @remarks
   * 1のリンクがクリックされたら、 0から10人の従業員で絞り込まれる
   * 2のリンクがクリックされたら、10から10人の従業員で絞り込まれる
   * 3のリンクがクリックされたら、20から10人の従業員で絞り込まれる...
   *
   * @param pageNum クリックされたページ番号
   */
  pagingLinkClick(pageNum: number): void {
    this.searchNameMessage = "";

    // 現在のページ数をクリックされたページ数に変更
    this.currentPageNum = pageNum;

    if (this.searchName === "") {
      // 検索入力欄に何も書かれていなかった場合、全従業員一覧情報をVuexストアから取得
      this.currentEmployeeList = this["$store"].getters.getAllEmployees;
    } else {
      // 検索入力欄に書かれていた場合、入力された文字列で絞り込みを行う
      this.currentEmployeeList = this["$store"].getters.getSearchEmployeeByName(
        this.searchName
      );
    }

    // 対象ページに出力する従業員一覧を作成する
    this.createEmployeeListForOnePage(pageNum, this.currentEmployeeList);
  }

  /**
   * 下のページング用のリンク数値を作成する(例：22人の場合だと3ページ).
   *
   * @param nowEmployeeCount 出力対象の従業員の数
   */
  private createLinkNumberForPaging(nowEmployeeCount: number): void {
    this.pageNums = [];
    this.maxPageNum = Math.ceil(nowEmployeeCount / this.MAX_EMPLOYEE_COUNT);
    for (let i = 1; i <= this.maxPageNum; i++) {
      this.pageNums.push(i);
    }
  }
  /*
   * 対象ページに出力する従業員一覧を作成する.
   *
   * @param targetPageNum 表示させる対象ページ番号
   * @param targetList 切り出す元の従業員一覧
   */
  private createEmployeeListForOnePage(
    targetPageNum: number,
    targetList: Array<Employee>
  ): void {
    // 開始位置を計算(1の場合は0、2の場合は10、3の場合は20・・・)
    let starNum = (targetPageNum - 1) * this.MAX_EMPLOYEE_COUNT;
    // 終了位置を計算(開始位置からMAX_EMPLOYEE_COUNT個取得)
    let endNum = starNum + this.MAX_EMPLOYEE_COUNT;
    console.log("startNum:" + starNum + "/endNum" + endNum);
    // 開始から終了までを切り出して表示用リストに格納
    this.currentEmployeeList = targetList.slice(starNum, endNum);
  }
  /*
   * 初期表示用共通メソッド.
   */
  private initialDisplay(): void {
    // 初期表示は0からMAX_EMPLOYEE_COUNT個表示
    this.currentEmployeeList = this.currentEmployeeList.slice(
      0,
      this.MAX_EMPLOYEE_COUNT
    );
    // 1ページに変更
    this.currentPageNum = 1;
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
