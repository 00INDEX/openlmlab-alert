<template>
  <div class="main">
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
      <el-form-item label="负责人">
        <el-input
          v-model="form.oncall"
          placeholder="预警信息通知目标，留空默认使用用户名"
        ></el-input>
      </el-form-item>
      <el-form-item label="勿扰时间">
        <el-time-picker
          v-model="form.no_disturbing_time"
          is-range
          range-separator="To"
          start-placeholder="Start time"
          end-placeholder="End time"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="add_job">插入</el-button>
      </el-form-item>
    </el-form>

    <el-table :data="tableData" style="width: 100%">
      <el-table-column prop="cluster" label="集群"> </el-table-column>
      <el-table-column prop="user" label="用户名"> </el-table-column>
      <el-table-column prop="oncall" label="负责人"> </el-table-column>
      <el-table-column prop="path" label="路径"> </el-table-column>
      <el-table-column prop="no_disturbing_time" label="勿扰时间">
      </el-table-column>
      <el-table-column label="状态">
        <template v-slot:default="scope">
          <el-icon style="color: green" :size="40" v-if="scope.row.status === 0"
            ><CircleCheckFilled
          /></el-icon>
          <el-icon
            style="color: #fa8c35"
            :size="40"
            v-if="scope.row.status === 1"
            ><InfoFilled
          /></el-icon>
          <el-icon
            style="color: #f00056"
            :size="40"
            v-if="scope.row.status === 2"
            ><WarnTriangleFilled
          /></el-icon>
        </template>
      </el-table-column>
      <el-table-column label="log_file信息">
        <template v-slot:default="scope">
          <span
            :style="{
              color:
                scope.row.log_file_status === 0
                  ? 'green'
                  : scope.row.log_file_status === 1
                  ? '#fa8c35'
                  : '#f00056',
            }"
            >{{ scope.row.log_file_msg }}</span
          >
        </template>
      </el-table-column>
      <el-table-column label="tensorboard信息">
        <template v-slot:default="scope">
          <span
            :style="{
              color:
                scope.row.tensorboard_status === 0
                  ? 'green'
                  : scope.row.tensorboard_status === 1
                  ? '#fa8c35'
                  : '#f00056',
            }"
            >{{ scope.row.tensorboard_msg }}</span
          >
        </template>
      </el-table-column>
      <el-table-column prop="next_run_time" label="距离下次查询">
      </el-table-column>
      <el-table-column label="操作" width="300">
        <template v-slot:default="scope">
          <div class="button-container">
            <el-button type="success" @click="resume_monitor(scope.$index)"
              >继续监控</el-button
            >
            <el-button type="warning" @click="pause_monitor(scope.$index)"
              >暂停监控</el-button
            >

            <el-button type="danger" @click="remove_job(scope.$index)"
              >删除</el-button
            >
          </div>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<style>
.button-container {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}
.main {
  padding-top: 50px;
  padding-left: 30px;
  padding-right: 30px;
}
</style>

<script>
import axios from 'axios';
import { ElNotification } from 'element-plus';
export default {
  data() {
    return {
      clusters: ['S', 'P', 'Ali', ''],
      form: {
        cluster: '',
        user: '',
        path: '',
        webhook: '',
        oncall: '',
        no_disturbing_time: [
          new Date(2016, 9, 10, 1, 0),
          new Date(2016, 9, 10, 8, 0),
        ],
      },
      tableData: [],
      interval: null,
    };
  },
  methods: {
    async add_job() {
      console.log('add_job');
      await axios
        .post('/monitor_system/alert/api/add', {
          cluster: this.form.cluster,
          user: this.form.user,
          path: this.form.path,
          webhook: this.form.webhook,
          oncall: this.form.oncall,
          no_disturbing_from: Math.floor(
            this.form.no_disturbing_time[0].getTime() / 1000
          ),
          no_disturbing_to: Math.floor(
            this.form.no_disturbing_time[1].getTime() / 1000
          ),
        })
        .then((res) => {})
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
    async remove_job(index) {
      await axios
        .post('/monitor_system/alert/api/remove', {
          cluster: this.tableData[index].cluster,
          user: this.tableData[index].user,
          path: this.tableData[index].path,
        })
        .then((res) => {
          if (res.data.code != 200) {
            ElNotification({
              title: 'Error',
              message: res.data.msg,
              type: 'error',
            });
          }
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
        .get('/monitor_system/alert/api/data')
        .then((res) => {
          if (res.data.code != 200) {
            ElNotification({
              title: 'Error',
              message: res.data.msg,
              type: 'error',
            });
          }
          this.tableData.length = 0;
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
        .post('/monitor_system/alert/api/pause', {
          cluster: this.tableData[index].cluster,
          user: this.tableData[index].user,
          path: this.tableData[index].path,
        })
        .then((res) => {
          if (res.data.code != 200) {
            ElNotification({
              title: 'Error',
              message: res.data.msg,
              type: 'error',
            });
          }
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
        .post('/monitor_system/alert/api/resume', {
          cluster: this.tableData[index].cluster,
          user: this.tableData[index].user,
          path: this.tableData[index].path,
        })
        .then((res) => {
          if (res.data.code != 200) {
            ElNotification({
              title: 'Error',
              message: res.data.msg,
              type: 'error',
            });
          }
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
  mounted: function () {
    this.get_data();
    this.interval = setInterval(this.get_data, 10000);
  },
};
</script>
