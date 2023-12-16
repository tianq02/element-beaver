<script setup>

//-----------------导入包--------------------

// ref用于存储动态数据
import {ref, onMounted} from 'vue'
// axios 发送请求
import axios from "axios";
import {HttpStatusCode} from "axios"


//-------------------全局变量------------------

const proxy_echo_status = ref('');

const jdbc_type = ref('');
const jdbc_url = ref('');
const jdbc_username = ref('');
const jdbc_password = ref('');
const jdbc_is_connected = ref(false);

const db_list = ref([]);
const db_name = ref('');

const table_list = ref([]);
const table_name = ref('');

const query_result = ref([]);
const query_sql = ref('');

const resetParams = function () {
  proxy_echo_status.value = '';

  jdbc_type.value = '';
  jdbc_url.value = '';
  jdbc_username.value = '';
  jdbc_password.value = '';
  jdbc_is_connected.value = false;

  db_list.value = [];
  db_name.value = '';

  table_list.value = [];
  table_name.value = '';

  query_result.value = [];
  query_sql.value = '';
};

const globalVariables = {
  proxy_echo_status,
  jdbc_type,
  jdbc_url,
  jdbc_username,
  jdbc_password,
  jdbc_is_connected,
  db_list,
  db_name,
  table_list,
  table_name,
  query_result,
  query_sql
};

//-------------------函数------------------


// axios测试
// 代理在vue.config.js中设置，注意url的path代理完不会变
const testProxyConnection = function () {
  axios.get("/echo/get")
      .then(function (response) {
        proxy_echo_status.value = response.statusText;
        console.log("testProxyConnection: ",response);
      })
      .catch(function (error) {
        proxy_echo_status.value = error;
        console.error("testProxyConnection: ",error);
      })
      .finally(function () {
        console.log("testProxyConnection: axios done")
      })
}

//获取当前数据库连接
const getJdbcConnection = function () {
  console.log("getJdbcConnection: called")
  axios.get("/api/config/get")
      .then(function (response) {
        // 能到达此处，HTTP的状态码已经有axios校验过，仅需处理若依封装的状态码

        // 从响应中取若依封装的AjaxResult
        const result = response.data
        if (
            !Object.hasOwn(result, "code") ||
            !Object.hasOwn(result, "data") ||
            result.code !== HttpStatusCode.Ok
        ) {
          if (Object.hasOwn(result, "msg"))
            window.alert("result.msg" + result.msg)
          console.error("getJdbcConnection: result code error", result);
          // 状态错误，连接未建立
          resetParams()
          return;
        }

        const jdbc_prop = result.data;

        if (
            !Object.hasOwn(jdbc_prop, "db") ||
            !Object.hasOwn(jdbc_prop, "address") ||
            !Object.hasOwn(jdbc_prop, "username") ||
            !Object.hasOwn(jdbc_prop, "password")
        ) {
          resetParams()
          console.error("getJdbcConnection: response jdbc prop format illegal", jdbc_prop);
          return;
        }

        jdbc_type.value = jdbc_prop.db;
        jdbc_url.value = jdbc_prop.address;
        jdbc_username.value = jdbc_prop.username;
        jdbc_password.value = jdbc_prop.password;
        jdbc_is_connected.value = true

        console.log("getJdbcConnection: success");

      })
      .catch(function (error) {
        console.error("getJdbcConnection: axios error", error);
      })
      .finally(function () {
        console.log("getJdbcConnection: axios done")
      })
}

//设置连接
const setJdbcConnection = function (
    new_jdbc_type,
    new_jdbc_url,
    new_jdbc_username,
    new_jdbc_password
) {
  console.log(
      "setJdbcConnection: called",
      new_jdbc_type,
      new_jdbc_url,
      new_jdbc_username,
      new_jdbc_password
  )
  axios.post("/api/config/set", null, {
    params: {
      db: new_jdbc_type,
      address: new_jdbc_url,
      username: new_jdbc_username,
      password: new_jdbc_password
    }
  })
      .then(function (response) {
        const result = response.data
        if (
            !Object.hasOwn(result, "code") ||
            result.code !== HttpStatusCode.Ok
        ) {
          if (Object.hasOwn(result, "msg"))
            window.alert("response.msg" + result.msg)
          resetParams()
          console.error("setJdbcConnection: response code error", result);
          return;
        }

        jdbc_type.value = new_jdbc_type;
        jdbc_url.value = new_jdbc_url;
        jdbc_username.value = new_jdbc_username;
        jdbc_password.value = new_jdbc_password;
        jdbc_is_connected.value = true;

        console.log(
            "setConnection:success",
            new_jdbc_type,
            new_jdbc_url,
            new_jdbc_username,
            new_jdbc_password
        );
      })
      .catch(function (error) {
        resetParams()
        console.error("setJdbcConnection: axios error", error);
      })
}


//------------------初始化语句--------------------

onMounted(() => {
  resetParams();
  testProxyConnection();
  // setJdbcConnection(
  //     "MySQL",
  //     "localhost:3306",
  //     "paste_u",
  //     "TX0A13fA_paste_u"
  // )
  getJdbcConnection()
  console.log(`the component is now mounted.`)
})

</script>

<template>
  <div id="cloud-cat">

    <div id="fun-call">
      <el-button @click="getJdbcConnection">
        getJdbcConnection
      </el-button>
    </div>

    <div id="var-show">
<!--      <el-table :data="globalVariables">-->
<!--        <el-table-column prop="key" label="变量名" width="180" />-->
<!--        <el-table-column prop="value" label="值" width="180" />-->
<!--      </el-table>-->
      <table>
        <tr>
          <th>Variable</th>
          <th>Value</th>
        </tr>
        <tr v-for="(value, key) in globalVariables" :key="key">
          <td>{{ key }}</td>
          <td>{{ value.value }}</td>
        </tr>
      </table>
    </div>

  </div>

</template>