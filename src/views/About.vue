<template>
  <div>
    <a-divider orientation="left">DataBase Status</a-divider>
    <a-row type="flex">
      <a-col :flex="2">
        <a-descriptions bordered>
          <a-descriptions-item label="DataBase Name">{{DbInfo.DataBaseName}}</a-descriptions-item>
          <a-descriptions-item label="Engine">{{this.DbInfo.Engine}}</a-descriptions-item>
          <a-descriptions-item label="Port">{{this.DbInfo.Port}}</a-descriptions-item>
          <a-descriptions-item label="DataSize">{{this.DbInfo.DataSize}}</a-descriptions-item>
          <a-descriptions-item label="Uptime" :span="2">{{this.DbInfo.Uptime}}</a-descriptions-item>
          <a-descriptions-item label="Status" :span="3">
            <a-badge status="processing" v-text="this.DbInfo.Status" />
          </a-descriptions-item>
          <a-descriptions-item label="Processlist">{{this.ProcesslistSize}}</a-descriptions-item>
          <a-descriptions-item label="QPS">{{this.QPS}}m/s</a-descriptions-item>
          <a-descriptions-item label="TPS">{{this.TPS}}m/s</a-descriptions-item>
        </a-descriptions>
      </a-col>
      <a-col :flex="3">
        <div style="margin-left:10%">
          <a-row :gutter="14">
            <a-col :span="8">
              <a-card>
                <a-statistic
                  title="KeyBufferReadHits"
                  :value="keyBufferRead"
                  :precision="2"
                  suffix="%"
                  :value-style="{ color: '#cf1322' }"
                  style="margin-right: 50px"
                >
                  <template #prefix>
                  </template>
                </a-statistic>
              </a-card>
            </a-col>
            <a-col :span="8">
              <a-card>
                <a-statistic
                  title="KeyBufferWriteHits"
                  :value="keyBufferWrite"
                  :precision="2"
                  suffix="%"
                  class="demo-class"
                  :value-style="{ color: '#cf1322' }"
                >
                  <template #prefix>
                  </template>
                </a-statistic>
              </a-card>
            </a-col>
          </a-row>
        </div>
        <div style="margin-left:10%">
          <a-row :gutter="14">
            <a-col :span="8">
              <a-card>
                <a-statistic
                  title="QueryCacheHits"
                  :value="queryCache"
                  :precision="2"
                  suffix="%"
                  :value-style="{ color: '#cf1322' }"
                  style="margin-right: 50px"
                >
                  <template #prefix>
                  </template>
                </a-statistic>
              </a-card>
            </a-col>
            <a-col :span="8">
              <a-card>
                <a-statistic
                  title="ThreadCacheHits"
                  :value="threadCache"
                  :precision="2"
                  suffix="%"
                  class="demo-class"
                  :value-style="{ color: '#cf1322' }"
                >
                  <template #prefix>
                  </template>
                </a-statistic>
              </a-card>
            </a-col>
          </a-row>
        </div>
      </a-col>
    </a-row>
    <a-divider orientation="left">连接数</a-divider>
    <a-row type="flex">
      <a-col flex="100%">
        <a-table :columns="columns" :data-source="Processlist">
          <a slot="name" slot-scope="text">{{ text }}</a>
          <span slot="customTitle">
            <a-icon type="smile-o" />Name
          </span>
          <span slot="tags" slot-scope="tags">
            <a-tag
              v-for="tag in tags"
              :key="tag"
              :color="tag === 'loser' ? 'volcano' : tag.length > 5 ? 'geekblue' : 'green'"
            >{{ tag.toUpperCase() }}</a-tag>
          </span>
          <span slot="action" slot-scope="text, record">
            <a>Invite 一 {{ record.name }}</a>
            <a-divider type="vertical" />
            <a>Delete</a>
            <a-divider type="vertical" />
            <a class="ant-dropdown-link">
              More actions
              <a-icon type="down" />
            </a>
          </span>
        </a-table>
      </a-col>
    </a-row>
  </div>
</template>
<script>
const columns = [
  {
    dataIndex: "Id",
    key: "Id",
    slots: { title: "Id" },
    scopedSlots: { customRender: "Id" },
  },
  {
    title: "BaseName",
    dataIndex: "BaseName",
    key: "BaseName",
  },
  {
    title: "Command",
    dataIndex: "Command",
    key: "Command",
  },
  {
    title: "Host",
    key: "Host",
    dataIndex: "Host",
    scopedSlots: { customRender: "Host" },
  },
  {
    title: "User",
    key: "User",
    dataIndex: "User",
    scopedSlots: { customRender: "User" },
  },
  {
    title: "State",
    key: "State",
    dataIndex: "State",
    scopedSlots: { customRender: "State" },
  },
  {
    title: "Info",
    key: "Info",
    dataIndex: "Info",
    scopedSlots: { customRender: "Info" },
  },
  {
    title: "Time",
    key: "Time",
    dataIndex: "Time",
    scopedSlots: { customRender: "Time" },
  },
  {
    title: "Action",
    key: "action",
    scopedSlots: { customRender: "action" },
  },
];

import axios from "axios";
import website from '../config/webstie'
export default {
  data() {
    return {
      DbInfo: {},
      Processlist: [],
      ProcesslistSize: 0,
      TPS: 0,
      QPS: 0,
      keyBufferRead: "",
      keyBufferWrite: "",
      queryCache:"",
      threadCache:"",
      columns,
    };
  },
  mounted() {
    this.getDBInfo();
    this.getProcesslist();
    this.getQPS();
    this.getTPS();
    this.getKeyBuffer();
    this.getQueryCache();
    this.getThreadCache();
  },
  methods: {
    getDBInfo() {
      axios
        .get(website.URL+"base/dataBaseStatus", {
          params: {},
        })
        .then((res) => {
          this.DbInfo = res.data.data;
        });
    },
    getProcesslist() {
      axios
        .get(website.URL+"base/processlist", {
          params: {},
        })
        .then((res) => {
          this.Processlist = res.data.data;
          this.ProcesslistSize = this.Processlist.length;
        });
    },
    getQPS() {
      axios
        .get(website.URL+"base/qps", {
          params: {},
        })
        .then((res) => {
          let v = String(res.data.data[0].Value);
          this.QPS = v.substring(0, 5);
        });
    },
    getTPS() {
      axios
        .get(website.URL+"base/tps", {
          params: {},
        })
        .then((res) => {
          let v = String(res.data.data[0].Value);
          this.TPS = v.substring(0, 5);
        });
    },
    getKeyBuffer() {
      axios
        .get(website.URL+"base/keyBuffer", {
          params: {},
        })
        .then((res) => {
          this.keyBufferRead =  res.data.data[0].Value;
          this.keyBufferWrite =  res.data.data[1].Value;
        });
    },
    // getInnodbBuffer() {
    //   axios
    //     .get(website.URL+"base/innodbBuffer", {
    //       params: {},
    //     })
    //     .then((res) => {
    //     });
    // },
    getQueryCache() {
      axios
        .get(website.URL+"base/queryCache", {
          params: {},
        })
        .then((res) => {
          this.queryCache =  res.data.data.Value;
        });
    },
    getThreadCache() {
      axios
        .get(website.URL+"base/threadCache", {
          params: {},
        })
        .then((res) => {
                 this.threadCache =  res.data.data.Value;
        });
    },
  },
};
</script>
