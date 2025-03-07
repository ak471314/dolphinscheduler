/*
* Licensed to the Apache Software Foundation (ASF) under one or more
* contributor license agreements.  See the NOTICE file distributed with
* this work for additional information regarding copyright ownership.
* The ASF licenses this file to You under the Apache License, Version 2.0
* (the "License"); you may not use this file except in compliance with
* the License.  You may obtain a copy of the License at
*
*    http://www.apache.org/licenses/LICENSE-2.0
*
* Unless required by applicable law or agreed to in writing, software
* distributed under the License is distributed on an "AS IS" BASIS,
* WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
* See the License for the specific language governing permissions and
* limitations under the License.
*/
<template>
  <div class="container">

    <div class="title-box">
      <span class="name">{{$t('Version Info')}}</span>
    </div>

    <div class="table-box" v-if="taskVersionData.taskDefinitionVersions.length > 0">
      <el-table :data="taskVersionData.taskDefinitionVersions" size="mini" style="width: 100%">
        <el-table-column type="index" :label="$t('#')" width="50"></el-table-column>
        <el-table-column prop="userName" :label="$t('Version')">
          <template slot-scope="scope">
            <span v-if="scope.row.version">
              <span v-if="scope.row.version === taskVersionData.taskDefinition.version" style="color: green"><strong>V{{scope.row.version}} {{$t('Current Version')}}</strong></span>
              <span v-else>V{{scope.row.version}}</span>
            </span>
            <span v-else>-</span>
          </template>
        </el-table-column>
        <el-table-column prop="description" :label="$t('Description')" min-width="240"></el-table-column>
        <el-table-column :label="$t('Create Time')" min-width="120">
          <template slot-scope="scope">
            <span>{{scope.row.updateTime | formatDate}}</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('Operation')" width="100">
          <template slot-scope="scope">
            <el-tooltip :content="$t('Code Diff With This Version')" placement="top">
              <el-popconfirm
                :confirmButtonText="$t('Confirm')"
                :cancelButtonText="$t('Cancel')"
                icon="el-icon-info"
                iconColor="red"
                :title="$t('Confirm Code Diff With This Version?')"
                @onConfirm="_mVersionDiffTaskDefinitionVersion(scope.row)"
              >
                <el-button :disabled=" scope.row.version === taskVersionData.taskDefinition.version || isInstance" type="primary" size="mini" icon="el-icon-warning" circle slot="reference"></el-button>
              </el-popconfirm>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div v-if="taskVersionData.taskDefinitionVersions.length === 0">
      <m-no-data><!----></m-no-data>
    </div>

    <div v-if="taskVersionData.taskDefinitionVersions.length > 0">
      <div class="bottom-box">
        <el-pagination
          style="float:right"
          background
          @current-change="_mVersionGetTaskDefinitionVersionsPage"
          layout="prev, pager, next"
          :total="taskVersionData.total">
        </el-pagination>
        <el-button type="text" size="mini" @click="_close()" style="float:right">{{$t('Cancel')}}</el-button>
      </div>
    </div>

  </div>
</template>

<script>
  import mNoData from '@/module/components/noData/noData'

  export default {
    name: 'taskVersions',
    data () {
      return {
        tableHeaders: [
          {
            label: 'version',
            prop: 'version'
          },
          {
            label: 'createTime',
            prop: 'createTime'
          }
        ]
      }
    },
    props: {
      isInstance: Boolean,
      taskVersionData: Object
    },
    methods: {
      /**
       * Diff version in task definition version list
       */
      _mVersionDiffTaskDefinitionVersion (item) {
        this.$emit('mVersionDiffTaskDefinitionVersion', {
          versionDataString: this.formatTaskDataString(item),
          taskDataString: this.formatTaskDataString(this.taskVersionData.taskDefinition.taskData),
          fromThis: this
        })
      },
      formatTaskDataString (task) {
        let versionStr = '\n********************\ntask version : ' + task.version + '\n********************\n'
        let taskStr = JSON.stringify(task.taskParams, null, '\t').toString()
        let sqlStr = ''
        let jsonData = JSON.parse(taskStr)
        if (Object.keys(jsonData).includes('sql')) {
          sqlStr = 'sql info : \n********************\n \n' + jsonData.sql + '\n'
        }
        return versionStr + sqlStr + '\n********************\ntask info: \n' + taskStr
      },
      /**
       * Paging event of task definition versions
       */
      _mVersionGetTaskDefinitionVersionsPage (val) {
        this.$emit('mVersionGetTaskDefinitionVersionsPage', {
          pageNo: val,
          pageSize: this.taskVersionData.pageSize,
          taskDefinitionCode: this.taskVersionData.taskDefinition.code,
          fromThis: this
        })
      },
      /**
       * Close and destroy component and component internal events
       */
      _close () {
        // flag Whether to delete a node this.$destroy()
        this.$emit('closeTaskVersion')
      }
    },
    created () {
    },
    mounted () {
    },
    components: { mNoData }
  }
</script>

<style lang="scss" rel="stylesheet/scss">
  .container {
    width: 800px;
    position: relative;

    .title-box {
      height: 61px;
      border-bottom: 1px solid #DCDEDC;
      position: relative;

      .name {
        position: absolute;
        left: 24px;
        top: 18px;
        font-size: 16px;
      }
    }

    .bottom-box {
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      text-align: right;
      height: 60px;
      line-height: 60px;
      border-top: 1px solid #DCDEDC;
      background: #fff;

      .ans-page {
        display: inline-block;
      }
    }

    .table-box {
      overflow-y: scroll;
      height: calc(100vh - 61px);
      padding-bottom: 60px;
    }
  }
</style>
