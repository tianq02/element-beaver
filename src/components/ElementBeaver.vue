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

const globalVariables = ref([
  { key: 'proxy_echo_status', value: proxy_echo_status },
  { key: 'jdbc_type', value: jdbc_type },
  { key: 'jdbc_url', value: jdbc_url },
  { key: 'jdbc_username', value: jdbc_username },
  { key: 'jdbc_password', value: jdbc_password },
  { key: 'jdbc_is_connected', value: jdbc_is_connected },
  { key: 'db_list', value: db_list },
  { key: 'db_name', value: db_name },
  { key: 'table_list', value: table_list },
  { key: 'table_name', value: table_name },
  // { key: 'query_result', value: query_result },
  { key: 'query_sql', value: query_sql }
]);

const isEditable = function (variable) {
  const valueType = typeof variable.value;
  return valueType === 'string' || valueType === 'number' || valueType === 'object';
}

//-------------------函数------------------


// axios测试
// 代理在vue.config.js中设置，注意url的path代理完不会变
const testProxyConnection = function () {
  axios.get("/echo/get")
      .then(function (response) {
        proxy_echo_status.value = response.statusText;
        console.log("testProxyConnection: ", response);
      })
      .catch(function (error) {
        proxy_echo_status.value = error;
        console.error("testProxyConnection: ", error);
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
            window.alert("getJdbcConnection: " + result.msg)
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
        window.alert("getJdbcConnection: axios error" + error)
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
            window.alert("setJdbcConnection: " + result.msg)
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

//获取可用数据库列表
const getDbList = function () {
  console.log("getDbList: called")
  axios.get("/api/database/getAll")
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
            window.alert("getDbList: " + result.msg)
          console.error("getDbList: result code error", result);
          // 状态错误，连接未建立
          resetParams()
          return;
        }

        db_list.value = result.data;

        console.log("getDbList: success");

      })
      .catch(function (error) {
        console.error("getDbList: axios error", error);
      })
      .finally(function () {
        console.log("getDbList: axios done")
      })
}

//获取可用数据库列表
const getDbUsed = function () {
  console.log("getDbUsed: called")
  axios.get("/api/database/get")
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
            window.alert("getDbUsed: " + result.msg)
          console.error("getDbUsed: result code error", result);
          // 状态错误，未指定table
          db_name.value = "";
          return;
        }

        const db_prop = result.data;

        if (
            !Object.hasOwn(db_prop, "dbname")
        ) {
          db_name.value = "";
          console.error("getDbUsed: response format illegal", db_prop);
          return;
        }

        db_name.value = db_prop.dbname;

        console.log("getDbUsed: success");

      })
      .catch(function (error) {
        console.error("getDbUsed: axios error", error);
      })
      .finally(function () {
        console.log("getDbUsed: axios done")
      })
}

//设置连接
const setDbUsed = function (new_db_name) {

  console.log("setDbUsed: called", new_db_name)

  axios.post("/api/database/set", null, {
    params: {
      dbname: new_db_name
    }
  })
      .then(function (response) {
        const result = response.data
        if (
            !Object.hasOwn(result, "code") ||
            result.code !== HttpStatusCode.Ok
        ) {
          if (Object.hasOwn(result, "msg"))
            window.alert("setDbUsed: " + result.msg)
          resetParams()
          console.error("setDbUsed: response code error", result);
          return;
        }

        db_name.value = new_db_name;

        console.log("setDbUsed:success", new_db_name);
      })
      .catch(function (error) {
        resetParams()
        console.error("setDbUsed: axios error", error);
      })
}

//获取可用数据库列表
const getTableList = function () {
  console.log("getTableList: called")
  axios.get("/api/tables")
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
            window.alert("getTableList: " + result.msg)
          console.error("getTableList: result code error", result);
          // 状态错误，数据库未指定模式
          resetParams()
          return;
        }

        table_list.value = response.data.data

        console.log("getTableList: success");

      })
      .catch(function (error) {
        console.error("getTableList: axios error", error);
      })
      .finally(function () {
        console.log("getTableList: axios done")
      })
}

const queryByTable = function (query_table_name, page_num, page_size) {
  if (query_table_name === "" || page_num < 1) {
    console.log("queryByTable: invalid params")
    window.alert("queryByTable: invalid params")
    return
  }
  axios.get("/api/table/get", {
    params: {
      tbname: query_table_name,
      pagenum: page_num,
      pagesize: page_size
    }
  })
      .then(function (response) {
        const result = response.data
        if (
            !Object.hasOwn(result, "code") ||
            !Object.hasOwn(result, "data") ||
            result.code !== HttpStatusCode.Ok
        ) {
          if (Object.hasOwn(result, "msg"))
            window.alert("queryByTable: " + result.msg)
          console.error("queryByTable: result code error", result);
          query_result.value = [];
          table_name.value = "";
          return;
        }
        query_result.value = result.data
        console.log("queryByTable: success");

      })
      .catch(function (error) {
        console.error("queryByTable: axios error", error);
      })
      .finally(function () {
        console.log("queryByTable: axios done")
      })
}


//------------------初始化语句--------------------

onMounted(function () {
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
  <div id="cloud-cat-jdbc">

    <div id="jdbc-ui">

    </div>

    <div id="fun-call">
      <el-button @click="getJdbcConnection">
        getJdbcConnection
      </el-button>
      <el-button @click="getDbList">
        getDbList
      </el-button>
      <el-button @click="getDbUsed">
        getDbUsed
      </el-button>
      <el-button @click="getTableList">
        getTableList
      </el-button>
      <br>

      <el-button
          @click='
             setJdbcConnection(
                 "MySQL",
                 "localhost:3306",
                 "paste_u",
                 "TX0A13fA_paste_u"
                 )
          '
      >
        setJdbcConnection
      </el-button>

      <el-button @click='setDbUsed("pastedb")'>
        setDbUsed
      </el-button>

      <el-button @click="queryByTable(table_name,1,10)">
        queryByTable({{ table_name }},1,10)
      </el-button>

    </div>




<!--    <div id="var-show">-->
<!--      <table>-->
<!--        <tr>-->
<!--          <th>Variable</th>-->
<!--          <th>Value</th>-->
<!--        </tr>-->
<!--        <tr v-for="(variable, key) in globalVariables" :key="key">-->
<!--          <td>{{ key }}</td>-->
<!--          <td>{{ variable.value }}</td>-->
<!--          <td>-->
<!--            &lt;!&ndash;            v-if="(typeof variable.value) in ['string','number','object']"&ndash;&gt;-->
<!--            <input v-model="variable.value" placeholder="new"/>-->
<!--          </td>-->
<!--        </tr>-->
<!--      </table>-->
<!--    </div>-->

    <div id="jdbc-settings">
      <el-collapse>

      </el-collapse>
    </div>

    <div>
      <el-table :data="globalVariables">
        <el-table-column prop="key" label="Variable"></el-table-column>
        <el-table-column prop="value" label="Value"></el-table-column>
        <el-table-column label="New Value">
          <template v-slot="{ row }">
            <el-input v-if="isEditable(row)" v-model="row.value" placeholder="new"></el-input>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div id="result-show">
      <el-table :data="query_result">
        <template v-if="query_result.length > 0">
          <el-table-column v-for="(value, key) in query_result[0]" :key="key" :prop="key" :label="key" />
        </template>
      </el-table>
    </div>

  </div>

</template>

<style scoped>

#var-show td {
  border: solid #589ef8;
  border-collapse: collapse;
  border-spacing: 0;
  overflow: hidden;
}
</style>