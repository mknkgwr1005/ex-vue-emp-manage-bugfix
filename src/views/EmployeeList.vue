<template>
  <div class="container">
    <div>
      <input type="text" v-model="name" />
      <button type="button" @click="searchForText">
        検索
      </button>
    </div>
    <!-- パンくずリスト -->
    <nav>
      <div class="nav-wrapper">
        <div class="col s12 teal">
          <router-link :to="'/employeeList/'" class="breadcrumb"
            >従業員リスト</router-link
          >
        </div>
      </div>
    </nav>
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
        {{
          errorMessage
        }}
        <tbody>
          <tr v-for="employee of currentEmployeeList" v-bind:key="employee.id">
            <td>
              <router-link :to="'/employeeDetail/' + employee.id">{{
                employee.name
              }}</router-link>
            </td>
            <td>{{ employee.hireDate }}</td>
            <td>{{ employee.dependentsCount }}人</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { Employee } from "@/types/employee";
/**
 * 従業員一覧を表示する画面.
 */
@Component
export default class EmployeeList extends Vue {
  // 従業員一覧
  private currentEmployeeList: Array<Employee> = [];
  // 従業員数
  private employeeCount = 0;
  // 検索欄
  private name = "";
  //
  private errorMessage = "";

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
    await this.$store.dispatch("getEmployeeList");

    // 従業員一覧情報をVuexストアから取得
    // 非同期で外部APIから取得しているので、async/await使わないとGetterで取得できない
    // ページング機能実装のため最初の10件に絞り込み
    this.currentEmployeeList = this.$store.getters.getAllEmployees;
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
   * 検索欄に入力された従業員一覧を表示する.
   */
  searchForText(): void {
    this.errorMessage = "";
    this.currentEmployeeList = this.$store.getters.getSearchEmployeeByName(
      this.name
    );
    if (this.currentEmployeeList.length <= 0) {
      this.errorMessage =
        "検索結果が1件もありませんでしたので、全件表示します。";
      this.currentEmployeeList = this.$store.getters.getAllEmployees;
    }
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
