<template>
  <n-h1 prefix="bar" style="margin-top: 30px;">
    <i class="twa twa-bookmark-tabs"></i>
    <n-text type="primary">
      配置文件
    </n-text>
  </n-h1>
  <n-grid cols="1" y-gap="1" item-responsive>
    <n-grid-item span="1">
      <n-card title="配置文件">
        <n-select v-model:value="node" :options="ServerList" @update:value="UpdateValue"/>
        <n-button strong secondary type="info" style="margin-top: 30px"
                  @click="copy(code, $event)">复制配置文件
        </n-button>
        <n-code :code="code" language="ini" show-line-numbers
                style="overflow:auto;margin-top: 30px;width: 100%;"></n-code>
      </n-card>
      <br/>
    </n-grid-item>

    <n-grid-item span="1">
      <n-card title="建站必看，不看导致的后果自负">
        <p>由于我们在新的配置文件生成机制中加入了</p>
        <n-code code="proxy_protocol_version = v2" language="ini"></n-code>
        <p>这一参数，导致网站如果不正确配置NGINX就无法正常访问</p>
        <n-divider/>
        <p>请根据下面的教程正确配置！</p>
        <p></p>
        <p>打开网站的配置文件（针对Nginx的网站，Apache或其他形式的网站自行百度或删除这行配置即可）</p>
        <a>在</a>
        <n-code code="listen 80" language="nginx" word-wrap></n-code>
        <a>和</a>
        <n-code
            code="listen 443 ssl http2" language="nginx" word-wrap></n-code>
        <a>两行的最后加上</a>
        <n-code
            code="proxy_protocol" language="nginx" word-wrap></n-code>
        <p></p>
        <p>例如：</p>
        <n-code code="listen 80 proxy_protocol
    listen 443 ssl http2 proxy_protocol
                    " language="nginx" word-wrap></n-code>
        <p>随后再在root下方放入这三行代码</p>
        <n-code code="real_ip_header proxy_protocol;
    real_ip_recursive on;
    set_real_ip_from 127.0.0.1;" language="nginx" word-wrap></n-code>
        <p>最后一个参数中的127.0.0.1请你改为创建隧道时填写的本地IP（Local_Ip）</p>
        <p>随后整个配置文件长得差不多像这样</p>
        <n-code code="server {
        listen 80 proxy_protocol
        listen 443 ssl http2 proxy_protocol
        server_name = xxx.com
        index index.html index.php
        root /ww/wwwroot/xxx.com
        real_ip_header proxy_protocol;
        real_ip_recursive on;
        set_real_ip_from 127.0.0.1;
    }" language="nginx" word-wrap></n-code>
        <p>保存，重启Nginx，你就可以获取用户的真实IP了！</p>
      </n-card>
    </n-grid-item>
  </n-grid>
</template>

<script>
</script>

<script setup>
import {ref} from 'vue';
import store from "../utils/stores/store.js";
import {get} from '../utils/request';
import {sendSuccessMessage, sendErrorMessage} from '../utils/message';
import clipboard from '..//utils/clipboard'


const node = ref("");
// 选择框数据
const ServerList = ref([]);
const code = ref("");

const rs = get("https://api.locyanfrp.cn/Proxies/GetServerList")
rs.then(res => {
  var i = 0;
  res.forEach(s => {
    if (i === 0) {
      node.value = s.id;
      UpdateValue(s.id);
    }
    ServerList.value[i] = {
      "label": s.name,
      "value": s.id
    };
    i = i + 1;
  });
})

function copy(data, event) {
  clipboard(data, event)
}

function UpdateValue(value) {
  const rs = get("https://api.locyanfrp.cn/Proxies/GetConfigFile?username=" + store.getters.get_username + "&token=" + store.getters.get_token + "&node=" + value, []);
  rs.then(res => {
    if (res.status) {
      sendSuccessMessage(res.message);
      code.value = res.config;
    } else {
      sendErrorMessage(res.message);
      code.value = "该节点下没有任何隧道捏~";
    }
  })
}

</script>