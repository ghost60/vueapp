<template>
  <div class="panel">
    <div class="null"></div>
    <div v-for="(item,index) in dataList" :key=index>
    <Cell :title="item.location" :lable="item.cn_name" :value="item.num+item.unit"/>
    </div>
    <toast v-model="warnToast" type="warn" width="50%">服务端异常</toast>
  </div>
</template>

<script>
import Cell from "@/components/cell";
import axios from "axios";
import { Toast } from "vux";
import {url} from "../common/global";

export default {
  name: "Real",
  components: { Cell, Toast },
  data() {
    return {
      dataList: [],
      warnToast: false
    };
  },
  mounted() {
    this.getSensorsStationId();
  },
  methods: {
    getSensorsStationId() {
      const _vm = this;
      axios({
        method: "get",
        url: "http://120.78.180.96:8080/sensors-stationId/" + localStorage.getItem("stationId"),
        // url: "/cw/sensors-stationId/" + localStorage.getItem("stationId"),
        headers: {
          username: localStorage.getItem("username"),
          stationId: localStorage.getItem("stationId"),
          token: localStorage.getItem("token"),
          uri: "sensors-stationId",
          type: "API"
        }
      })
        .then(res => {
          if (res.data) {
            var oldlist = res.data.data;
            for (let i = 0; i < oldlist.length; i++) {
              if (oldlist[i].unit == "暂时没有单位") {
                oldlist[i].unit = "";
              }
            }
            this.getSensorData(oldlist, _vm);
          }
        })
        .catch(error => {
          this.warnToast = true;
        });
    },

    getSensorData(oldlist, _vm) {
      let ws = null;
      let list = oldlist;
      if ("WebSocket" in window) {
        ws = new WebSocket("ws://120.78.180.96:18888" + "/webSocket/recycledwater/" + localStorage.getItem('username') +"/" + localStorage.getItem('token'));
        ws.onopen = function() {
          let id = [];
          list.forEach(element => {
            id.push(element.equipId);
          });
          ws.send(JSON.stringify({ type: "recycledwater", data: id }));
        };
        ws.onmessage = function(evt) {
          let num = JSON.parse(evt.data);
          for (let i = list.length - 1; i >= 0; i--) {
            for (let j = num.length - 1; j >= 0; j--) {
              if (list[i].equipId == num[j].id) {
                list[i].num = num[j].value;
              }
            }
          }
          _vm.dataList = list;
        };
      }
    }
  }
};
</script>