<template>
  <div>
    <el-form ref="form" :model="form" inline>
      <el-form-item label="集群">
        <el-select v-model="form.cluster" placeholder="请选择">
          <el-option
            v-for="item in clusters"
            :key="item"
            :label="item"
            :value="item"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="用户名">
        <el-input v-model="form.user"></el-input>
      </el-form-item>
      <el-form-item label="路径">
        <el-input v-model="form.path"></el-input>
      </el-form-item>
      <el-form-item label="飞书webhook">
        <el-input v-model="form.webhook"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="add_job">插入</el-button>
      </el-form-item>
    </el-form>

    <el-table :data="tableData" style="width: 100%">
      <el-table-column prop="cluster" label="集群"> </el-table-column>
      <el-table-column prop="user" label="用户名"> </el-table-column>
      <el-table-column prop="path" label="路径"> </el-table-column>
      <el-table-column label="状态">
        <template v-slot:default="scope">
          <el-icon
            :style="{
              color:
                scope.row.status === 0
                  ? 'green'
                  : scope.row.status === 1
                  ? 'yellow'
                  : 'red',
            }"
            :name="statusIcon(scope.row.status)"
          ></el-icon>
        </template>
      </el-table-column>
      <el-table-column
        :style="{
          color:
            scope.row.log_file_status === 0
              ? 'green'
              : scope.row.log_file_status === 1
              ? 'yellow'
              : 'red',
        }"
        prop="log_file_msg"
        label="log_file信息"
      >
      </el-table-column>
      <el-table-column
        :style="{
          color:
            scope.tensorboard_status === 0
              ? 'green'
              : scope.row.tensorboard_status === 1
              ? 'yellow'
              : 'red',
        }"
        prop="tensorboard_msg"
        label="tensorboard信息"
      >
      </el-table-column>
      <el-table-column label="操作" width="180">
        <template v-slot:default="scope">
          <el-button @click="pause_monitor(scope.$index)" v-if="!scope.paused"
            >暂停监控</el-button
          >
          <el-button @click="resume_monitor(scope.$index)" v-if="scope.paused"
            >继续监控</el-button
          >
          <el-button type="danger" @click="remove_job(scope.$index)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import axios from 'axios';
import { ElNotification } from 'element-plus';
export default {
  data() {
    return {
      clusters: ['S', 'P', '阿里云'],
      form: {
        cluster: '',
        user: '',
        path: '',
        webhook: '',
      },
      tableData: [],
    };
  },
  methods: {
    async add_job() {
      await axios
        .post(
          '/add',
          (data = {
            cluster: this.form.cluster,
            user: this.form.user,
            path: this.form.path,
          })
        )
        .then((res) => {
          this.get_data();
        })
        .catch((e) => {
          ElNotification({
            title: 'Error',
            message: e.message,
            type: 'error',
          });
        });
    },
    statusIcon(status) {
      switch (status) {
        case 0:
          return 'el-icon-check';
        case 1:
          return 'el-icon-warning';
        case 2:
          return 'el-icon-error';
        default:
          return 'el-icon-question-filled';
      }
    },
    async reomve_job(index) {
      await axios
        .post(
          '/remove',
          (data = {
            cluster: this.tableData[index].cluster,
            user: this.tableData[index].user,
            path: this.tableData[index].path,
          })
        )
        .then((res) => {
          this.get_data();
        })
        .catch((e) => {
          ElNotification({
            title: 'Error',
            message: e.message,
            type: 'error',
          });
        });
    },
    async get_data() {
      await axios
        .get('/api/data')
        .then((res) => {
          this.tableData.clear();
          Array.prototype.push.apply(this.tableData, res.data.data);
        })
        .catch((e) => {
          ElNotification({
            title: 'Error',
            message: e.message,
            type: 'error',
          });
        });
    },
    async pause_monitor(index) {
      await axios
        .post(
          '/api/pause',
          (data = {
            cluster: this.tableData[index].cluster,
            user: this.tableData[index].user,
            path: this.tableData[index].path,
          })
        )
        .then((res) => {
          this.get_data();
        })
        .catch((e) => {
          ElNotification({
            title: 'Error',
            message: e.message,
            type: 'error',
          });
        });
    },
    async resume_monitor(index) {
      await axios
        .post(
          '/api/resume',
          (data = {
            cluster: this.tableData[index].cluster,
            user: this.tableData[index].user,
            path: this.tableData[index].path,
          })
        )
        .then((res) => {
          this.get_data();
        })
        .catch((e) => {
          ElNotification({
            title: 'Error',
            message: e.message,
            type: 'error',
          });
        });
    },
  },
  mounted: function () {},
};
</script>
