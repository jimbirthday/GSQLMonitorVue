<template>
  <div id="components-layout-demo-basic">
    <a-layout>
      <a-layout-header>Welcome To Explain SQL</a-layout-header>
      <a-layout>
        <a-layout-content>
          <div id="components-form-demo-advanced-search">
            <a-form class="ant-advanced-search-form" :form="form" @submit="handleSearch">
              <a-row :gutter="24">
                <a-form-item>
                  <a-textarea placeholder="Please enter sql" :rows="4" v-model="sql" />
                </a-form-item>
              </a-row>
              <a-row>
                <a-col :span="24" :style="{ textAlign: 'right' }">
                  <a-button type="primary" html-type="submit">Search</a-button>
                  <a-button :style="{ marginLeft: '8px' }" @click="handleReset">Clear</a-button>
                </a-col>
              </a-row>
            </a-form>
            <div>
              <a-table :columns="columns" :data-source="slowRes" :scroll="{ x: 1500, y: 300 }">
                <a slot="action">action</a>
              </a-table>
            </div>
          </div>
        </a-layout-content>
      </a-layout>
      <a-layout-footer>
        <div v-if="showerr">
          <a-alert
            message="Error Tips"
            :description="result"
            type="error"
            closable
            @close="onClose"
          />
        </div>
      </a-layout-footer>
    </a-layout>
  </div>
</template>


<script>
import axios from "axios";
import website from "../config/webstie";
const columns = [
  {
    title: "table",
    width: 200,
    dataIndex: "table",
    key: "table",
    fixed: "left",
  },

  { title: "id", width: 100, dataIndex: "id", key: "id", fixed: "left" },
  { title: "filtered", dataIndex: "filtered", key: "1", width: 150 },
  { title: "key", dataIndex: "key", key: "key", width: 150 },
  { title: "key_len", dataIndex: "key_len", key: "key_len", width: 150 },
  {
    title: "partitions",
    dataIndex: "partitions",
    key: "partitions",
    width: 150,
  },
  {
    title: "possible_keys",
    dataIndex: "possible_keys",
    key: "possible_keys",
    width: 150,
  },
  { title: "ref", dataIndex: "ref", key: "ref", width: 150 },
  { title: "rows", dataIndex: "rows", key: "7", width: 150 },
  {
    title: "select_type",
    dataIndex: "select_type",
    key: "select_type",
    width: 150,
  },
  { title: "type", dataIndex: "type", key: "type", width: 150 },
  { title: "Extra", dataIndex: "Extra", key: "Extra", width: 150 },

  {
    title: "Action",
    key: "operation",
    fixed: "right",
    width: 100,
    scopedSlots: { customRender: "action" },
  },
];

export default {
  data() {
    return {
      showerr: false,
      slowRes: [],
      columns,
      size: "default",
      expand: false,
      result: "",
      sql: "",
      form: this.$form.createForm(this, { name: "advanced_search" }),
    };
  },
  computed: {
    count() {
      return this.expand ? 11 : 7;
    },
  },
  updated() {},
  methods: {
    onClose(e) {
      e.preventDefault();
      this.showerr = false;
    },
    onChange(e) {
      console.log("size checked", e.target.value);
      this.size = e.target.value;
    },
    explainSql() {
      let from = new FormData();
      from.append("sql", this.sql);
      axios.post(website.URL + "action/explainSql", from).then((res) => {
        if (res.data.code == 200) {
          this.slowRes = res.data.data;
          this.showerr = false;
        } else {
          this.result = res.data.data.Message;
          this.showerr = true;
        }
      });
    },
    handleSearch(e) {
      e.preventDefault();
      this.explainSql();
    },

    handleReset() {
      this.sql = "";
    },

    toggle() {
      this.expand = !this.expand;
    },
  },
};
</script>
<style>
.ant-advanced-search-form {
  padding: 24px;
  background: #fbfbfb;
  border: 1px solid #d9d9d9;
  border-radius: 6px;
}

.ant-advanced-search-form .ant-form-item {
  display: flex;
}

.ant-advanced-search-form .ant-form-item-control-wrapper {
  flex: 1;
}

#components-form-demo-advanced-search .ant-form {
  max-width: none;
}
#components-form-demo-advanced-search .search-result-list {
  margin-top: 16px;
  border: 1px dashed #e9e9e9;
  border-radius: 6px;
  background-color: #fafafa;
  min-height: 200px;
  text-align: center;
  padding-top: 80px;
}

#components-layout-demo-basic {
  text-align: center;
}
#components-layout-demo-basic .ant-layout-header,
#components-layout-demo-basic .ant-layout-footer {
  color: #fff;
}
#components-layout-demo-basic .ant-layout-footer {
  line-height: 1.5;
}
#components-layout-demo-basic .ant-layout-sider {
  color: #fff;
  line-height: 120px;
}
#components-layout-demo-basic .ant-layout-content {
}
#components-layout-demo-basic > .ant-layout {
  margin-bottom: 48px;
}
#components-layout-demo-basic > .ant-layout:last-child {
  margin: 0;
}
</style>

