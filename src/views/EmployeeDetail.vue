<template>
  <div class="container">
    <!-- パンくずリスト -->
    <nav>
      <div class="nav-wrapper">
        <div class="col s12 teal">
          <router-link :to="'/employeeList/'" class="breadcrumb"
            >従業員リスト</router-link
          >
          <router-link :to="'/employeeDetail/:id/'" class="breadcrumb"
            >従業員詳細</router-link
          >
          >
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
              <td><span v-text="currentEmployee.formatDate"></span></td>
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
                    v-model.number="currentDependentsCount"
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
import { format } from "date-fns";

/**
 * 従業員詳細を表示する画面.
 */
@Component
export default class EmployeeDetail extends Vue {
  // 従業員情報
  // 初期値で初期化しなければtemplateでidなどが存在しないということでエラーとなる
  private currentEmployee = new Employee(
    0,
    "XXXX",
    "/noImage.png",
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
   * 従業員詳細をWebAPIから取得する.
   *
   * @returns - Promiseオブジェクト
   *
   * @remarks リロードするとページが消えることの対策として作成されたWebAPIから
   * 渡されたIDの従業員詳細を表示させる
   */
  async created(): Promise<void> {
    // 送られてきたリクエストパラメータのidをnumberに変換して取得する
    const employeeId = parseInt(this.$route.params.id);
    const response = await axios.get(
      `http://153.127.48.168:8080/ex-emp-api/employee/${employeeId}`
    );
    console.dir("respone:" + JSON.stringify(response));
    this.currentEmployee = new Employee(
      response.data.employee.id,
      response.data.employee.name,
      response.data.employee.image,
      response.data.employee.gender,
      new Date(response.data.employee.hireDate),
      response.data.employee.mailAddress,
      response.data.employee.zipCode,
      response.data.employee.address,
      response.data.employee.telephone,
      response.data.employee.salary,
      response.data.employee.characteristics,
      response.data.employee.dependentsCount
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
