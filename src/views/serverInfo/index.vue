/*
 * Copyright 2014-2019 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
<template>
    <div class="rivate-key-management-wrapper">
        <v-contentHead :headTitle="`Oracle服务列表`"></v-contentHead>
        <div class="module-wrapper" style="padding-bottom: 20px;">
            <div class="search-part">
                <el-form :inline="true" :model="formInline" class="" :rules="rules" ref="ruleForm">
                    <el-form-item label="链" prop="chainId">
                        <el-select v-model="formInline.chainId" placeholder="链" @change="changeChain">
                            <el-option v-for="item in allChainAndGroup" :label="item.chainId" :value="item.chainId" :key="item.chainId"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="群组" prop="groupId">
                        <el-select v-model="formInline.groupId" placeholder="群组">
                            <el-option v-for="item in groupList" :label="item" :value="item" :key="item"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="onSubmit('ruleForm')">查询</el-button>
                    </el-form-item>
                </el-form>
            </div>
            <div class="search-table">
                <el-table :data="list" tooltip-effect="dark" v-loading="loading">
                    <el-table-column v-for="head in privateKeyHead" :label="head.name" :key="head.enName" show-overflow-tooltip :width="head.tdWidth" align="center">
                        <template slot-scope="scope">
                            <span v-if="head.enName ==='oracleServiceAddress'">
                                <i class="wbs-icon-copy font-12 copy-public-key" @click="copyPubilcKey(scope.row[head.enName])" title="复制"></i>
                                <span>
                                    {{scope.row[head.enName]}}
                                </span>
                            </span>
                            <span v-else-if="head.enName ==='keyHash'">
                                 <i class="wbs-icon-copy font-12 copy-public-key" @click="copyPubilcKey(scope.row[head.enName])" title="复制"></i>
                                <span>
                                    {{scope.row[head.enName]}}
                                </span>
                            </span>
                            <span v-else-if="head.enName ==='status'">
                                <span>
                                    {{scope.row[head.enName] | serverStatus}}
                                </span>
                            </span>
                            <span v-else>{{scope.row[head.enName]}}</span>
                        </template>

                    </el-table-column>

                </el-table>
                <!-- <el-pagination class="page" @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="currentPage" :page-sizes="[10, 20]" :page-size="pageSize" layout="total, sizes, prev, pager, next, jumper" :total="total">
                </el-pagination> -->
            </div>
        </div>
    </div>
</template>


<script>
import contentHead from "@/components/contentHead";
import { getChainAndGroup, getServer } from "@/util/api";

export default {
    name: "ServerInfo",
    components: {
        "v-contentHead": contentHead,
    },
    computed: {
        privateKeyHead() {
            var arr = [
                {
                    enName: "index",
                    name: '编号',
                    tdWidth: ''
                },
                {
                    enName: "oracleServiceAddress",
                    name: '服务地址',
                    tdWidth: ''
                },
               
                {
                    enName: "operator",
                    name: "运营方",
                    tdWidth: ""
                },
                {
                    enName: "url",
                    name: "访问地址",
                    tdWidth: ""
                },
                 {
                    enName: "keyHash",
                    name: "KeyHash",
                    tdWidth: ""
                },
                {
                    enName: "status",
                    name: "状态",
                    tdWidth: ""
                },
                {
                    enName: "latestRequstProcessedTime",
                    name: "请求时长（ms）",
                    tdWidth: ""
                },
                {
                    enName: "creatTime",
                    name: "创建时间",
                    tdWidth: ""
                },
            ]
            return arr
        },
        rules() {
            let data = {
                chainId: [
                    {
                        required: true,
                        message: '请选择链',
                        trigger: "change"
                    },
                ],
                groupId: [
                    {
                        required: true,
                        message: '请选择群组',
                        trigger: "change"
                    },
                ],
            }
            return data
        }
    },
    data() {
        return {
            loading: false,
            currentPage: 1,
            pageSize: 10,
            total: 0,
            list: [],
            allChainAndGroup: [],
            formInline: {
                chainId: '',
                groupId: ''
            },
            groupList: []
        };
    },
    mounted() {
        this.queryChainAndGroup()
    },
    methods: {
        queryChainAndGroup() {
            getChainAndGroup()
                .then(res => {
                    this.loading = false
                    const { status, data } = res
                    if (data.code === 0) {
                        this.allChainAndGroup = data.data
                        if (data.data.length > 0) {
                            this.formInline.chainId = data.data[0]['chainId']
                            this.groupList = data.data[0]['groupIdList']
                            if(this.groupList.length > 0){
                                this.formInline.groupId = data.data[0]['groupIdList'][0]
                            }
                            this.onSubmit('ruleForm')
                        }
                    } else {

                    }
                })
                .catch(err => {

                })
        },
        queryServer() {
            let reqData = {
                chainId: this.formInline.chainId,
                groupId: this.formInline.groupId
            }
            getServer(reqData)
                .then(res => {
                    this.loading = false
                    const { status, data } = res
                    if (data.code === 0) {
                        this.list = data.data
                        this.total = data.totalCount
                    } else {

                    }
                })
                .catch(err => {

                })
        },
        handleSizeChange(val) {
            this.pageSize = val;
            this.currentPage = 1;
            this.queryServer();
        },
        handleCurrentChange(val) {
            this.currentPage = val;
            this.queryServer();
        },
        link(reqId) {
            this.$router.push({
                path: '/newQuery',
                query: {
                    reqId: reqId
                }
            })
        },
        changeChain(val) {
            this.allChainAndGroup.forEach(item => {
                if (val == item.chainId) {
                    this.groupList = item.groupIdList
                }
            })
        },
        onSubmit(formName) {
            this.$refs[formName].validate(valid => {
                if (valid) {
                    this.loading = true;
                    this.queryServer()
                } else {
                    return false;
                }
            });
        },
        copyPubilcKey(val) {
            if (!val) {
                this.$message({
                    type: "fail",
                    showClose: true,
                    message: 'key为空，不复制。',
                    duration: 2000
                });
            } else {
                this.$copyText(val).then(e => {
                    this.$message({
                        type: "success",
                        showClose: true,
                        message: '复制成功',
                        duration: 2000
                    });
                });
            }
        },
    }
};
</script>
<style scoped>

</style>
