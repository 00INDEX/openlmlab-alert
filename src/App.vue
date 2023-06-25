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
        <el-input v-model="form.username"></el-input>
      </el-form-item>
      <el-form-item label="路径">
        <el-input v-model="form.path"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="insertRow">插入</el-button>
      </el-form-item>
    </el-form>

    <el-table :data="tableData" style="width: 100%">
      <el-table-column prop="cluster" label="集群"> </el-table-column>
      <el-table-column prop="username" label="用户名"> </el-table-column>
      <el-table-column prop="path" label="路径"> </el-table-column>
      <el-table-column prop="status" label="状态"> </el-table-column>
      <el-table-column label="操作" width="180">
        <template v-slot:default="scope">
          <el-button disabled>忽略</el-button>
          <el-button type="danger" @click="deleteRow(scope.$index)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  data() {
    return {
      clusters: ['S', 'P', '阿里云'],
      form: {
        cluster: '',
        username: '',
        path: '',
      },
      tableData: [],
    };
  },
  methods: {
    insertRow() {
      this.tableData.push({ ...this.form });
      this.form = { cluster: '', username: '', path: '' };
    },
    deleteRow(index) {
      this.tableData.splice(index, 1);
    },
  },
  mounted: function () {},
};
</script>
