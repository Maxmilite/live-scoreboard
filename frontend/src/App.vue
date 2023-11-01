<script setup lang="ts">
import { onMounted, ref } from "vue";
import { ElMessage } from 'element-plus'
import { i18nAssets } from './assets/constants'

const serverAddress = "/api"
const state = ref(0);
const loggedIn = ref(false);
const userName = ref("");

const queryContestID = ref(0);

function stateHandler(e: number) {
  state.value = -1;
  queryContestID.value = e;
  setTimeout(() => {
    state.value = 1;
    if (e === 0) state.value = 0;
    if (!loggedIn.value) {
      state.value = 0;
      ElMessage.error(i18nAssets.signinRequired)
    }
  }, 10);
}

const sleep = (delay: number) => new Promise((resolve) => setTimeout(resolve, delay));

function login(username: string, password: string) {
  let HTTPRequest = new XMLHttpRequest();
  HTTPRequest.open("POST", serverAddress + "/login", false);
  HTTPRequest.withCredentials = true;
  HTTPRequest.setRequestHeader("Content-Type", "application/json;charset=UTF-8");
  try {
    HTTPRequest.send(JSON.stringify({ "username": username, "password": password }));
    let returnMessage = JSON.parse(HTTPRequest.responseText);
    if (returnMessage.code == 200) {
      ElMessage.success(returnMessage.msg);
      loggedIn.value = true;
      userName.value = getSession();
      return 1;
    } else {
      ElMessage.error("Error: " + returnMessage.msg);
      return 0;
    }
  } catch (error) {
    ElMessage.error("Error: Network Error");
    return 0;
  }
}

function logout() {
  let HTTPRequest = new XMLHttpRequest();
  HTTPRequest.open("GET", serverAddress + "/logout", false);
  HTTPRequest.withCredentials = true;
  try {
    HTTPRequest.send();
    let returnMessage = JSON.parse(HTTPRequest.responseText);
    ElMessage.success(returnMessage.msg);
    loggedIn.value = false;
    stateHandler(0);
    return 0;
  } catch (error) {
    ElMessage.error("Error: Network Error");
    return 0;
  }
}

function getSession() {
  let HTTPRequest = new XMLHttpRequest();
  HTTPRequest.open("GET", serverAddress + "/session", false);
  HTTPRequest.withCredentials = true;
  try {
    HTTPRequest.send();
    let returnMessage = JSON.parse(HTTPRequest.responseText);
    if (returnMessage.code == 200) {
      return returnMessage.msg;
    } else {
      return "";
    }
  } catch (error) {
    return "";
  }
}

function getContestList() {
  let returnMessage = "";
  let HTTPRequest = new XMLHttpRequest();
  HTTPRequest.open("GET", serverAddress + "/contest/list", false);
  try {
    HTTPRequest.send();
    returnMessage = HTTPRequest.responseText;
    return JSON.parse(returnMessage);
  } catch (error) {
    return i18nAssets.error;
  }
}

function getContestInfo(id: number) {
  let returnMessage = "";
  let HTTPRequest = new XMLHttpRequest();
  HTTPRequest.open("GET", serverAddress + "/contest/info?id=" + id, false);
  try {
    HTTPRequest.send();
    returnMessage = HTTPRequest.responseText;
    return JSON.parse(returnMessage);
  } catch (error) {
    return i18nAssets.error;
  }
}

function getRecords(id: number) {
  let returnMessage = "";
  let HTTPRequest = new XMLHttpRequest();
  HTTPRequest.open("GET", serverAddress + "/record?id=" + id, false);
  try {
    HTTPRequest.send();
    returnMessage = HTTPRequest.responseText;
    return JSON.parse(returnMessage);
  } catch (error) {
    return i18nAssets.error;
  }
}

function setCategory(id: number, category: string) {
  let HTTPRequest = new XMLHttpRequest();
  HTTPRequest.open("POST", serverAddress + "/history/set", false);
  HTTPRequest.withCredentials = true;
  HTTPRequest.setRequestHeader("Content-Type", "application/json;charset=UTF-8");
  try {
    HTTPRequest.send(JSON.stringify({ "id": id, "category": category }));
    let returnMessage = JSON.parse(HTTPRequest.responseText);
    if (returnMessage.code == 200) {
      ElMessage.success("Success");
      return 1;
    } else {
      ElMessage.error("Error: " + returnMessage.msg);
      return 0;
    }
  } catch (error) {
    ElMessage.error("Error: Network Error");
    return 0;
  }
}

function getHistory(category: string) {
  if (loggedIn.value == false) {
    ElMessage.error(i18nAssets.signinRequired);
    setTimeout(() => stateHandler(0), 10);
    return {};
  }
  let returnMessage = "";
  let HTTPRequest = new XMLHttpRequest();
  let address = "/history/get?username=" + userName.value;
  if (category) {
    address += "&category=" + category;
  }
  HTTPRequest.open("GET", serverAddress + address, false);
  try {
    HTTPRequest.send();
    returnMessage = HTTPRequest.responseText;
    return JSON.parse(returnMessage);
  } catch (error) {
    return "An error occurred while corresponding with the chatbot."
  }
}

function getCategories() {
  if (loggedIn.value == false) {
    return {};
  }
  let returnMessage = "";
  let HTTPRequest = new XMLHttpRequest();
  let address = "/history/categories";
  HTTPRequest.open("GET", serverAddress + address, false);
  try {
    HTTPRequest.send();
    returnMessage = HTTPRequest.responseText;
    return JSON.parse(returnMessage);
  } catch (error) {
    return "An error occurred while corresponding with the chatbot."
  }
}

onMounted(() => {
  let res = getSession();
  if (res !== "") {
    userName.value = res;
    loggedIn.value = true;
  }
});
</script>

<template>
  <el-config-provider namespace="ep">
    <BaseHeader @response="(e) => stateHandler(e)" :username="userName" :login-function="login" :logged-in="loggedIn"
      :logout-function="logout" />
    <div style="display: flex">
      <div style="width: 100%; height: calc(100vh - 120px);">
        <Welcome v-if="state === 0" :contest-list-function="getContestList" :logged-in="loggedIn"
          @response="(e) => stateHandler(e)" msg="Live Scoreboard" />
        <Scoreboard v-else-if="state === 1" :cid="queryContestID" :contest-info-function="getContestInfo"
          :record-function="getRecords" />
      </div>
    </div>
  </el-config-provider>
</template>
  
<style>
#app {
  text-align: center;
  color: var(--ep-text-color-primary);
}

.element-plus-logo {
  width: 50%;
}
</style>
