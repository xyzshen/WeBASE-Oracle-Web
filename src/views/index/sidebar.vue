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
    <div style="height: 100%;">
        <div style="height: 100%;background-color: #2c2e2f" class="sidebar-content" ref='sidebarContent'>
            <div class="image-flex justify-center center font-color-fff" style="height: 54px;position:relative;" v-if="menuShowC">
                <span class="font-28">
                    WeBASE-Oracle-Web
                </span>
            </div>
            <div class="mini-sidebar-contract-icon" v-if="!menuShowC" style="padding-bottom:40px">
                <i class="el-icon-caret-right font-color-aeb1b5" @click="hideMune(false)" style="font-size: 18px;"></i>
            </div>
            <el-menu default-active="999" router class="el-menu-vertical-demo" text-color="#9da2ab" active-text-color="#1f83e7" active-background-color="#2c2e2f" background-color="#2c2e2f" @select="select" :collapse="!menuShowC" @open="handleOpen" @close="handleClose">
                <template v-for="(item,index) in routesListC" v-if="item.menuShow">
                    <el-submenu v-if="!item.leaf" :index="`${index}`" ref="ele" class="">
                        <template slot="title">
                            <div :style="{'padding-left':  menuShowC ? '13px':''}">
                                <i :class="item.iconCls" :style="{'color': activeIndex == index ? '#1f83e7':''}"></i>
                                <span :class="{'font-color-37eef2': activeIndex == index}">{{item.name}}</span>
                            </div>
                        </template>

                        <el-menu-item v-for="term in item.children" :key="term.path" :index="term.path" v-if="term.menuShow" style="padding-left: 58px" :style="{
                                    'padding-left':term.path == activeRoute ? '55px': '58px',
                                    'background-color':term.path == activeRoute ? '#1e293e': '#2c2e2f',}">
                            <span>{{term.name}}</span>
                        </el-menu-item>
                    </el-submenu>
                    <el-menu-item v-else-if="item.leaf&&item.children&&item.children.length" :index="item.children[0].path" :style="{
                                'padding-left':item.children[0].path == activeRoute ? '30px': '33px',
                                'background-color':item.children[0].path == activeRoute ? '#1e293e': '#2c2e2f',}">
                        <i :class="item.iconCls"></i>
                        <span slot="title">{{item.children[0].name}}</span>
                    </el-menu-item>
                </template>
            </el-menu>
        </div>
    </div>
</template>

<script>
import router from "@/router";
import {
    queryGroup,
    queryClientVersion
} from "@/util/api";
import Bus from "@/bus"
export default {
    name: "sidebar",
    props: ["minMenu"],
    data() {
        return {
            activeIndex: 0,
            activeRoute: "",
            routesList: [],
            buttomNone: true,
        };
    },
    computed: {
        menuShowC() {
            if (this.minMenu) {
                return this.minMenu;
            } else {
                return false;
            }
        },
        routesListC() {
            var list = this.routesList
            list.forEach((item => {
                switch (item.enName) {
                    case 'contractManagement':
                        item.name = this.$t('route.contractManagement')
                        break;
                    case 'systemMonitoring':
                        item.name = this.$t('route.systemMonitoring')
                        break;

                }
                if (item.children) {
                    item.children.forEach((it) => {
                        switch (it.enName) {
                            case 'statistics':
                                it.name = this.$t('route.statistics')
                                break;
                            case 'nodeManagement':
                                it.name = this.$t('route.nodeManagement')
                                break;
                            case 'contractIDE':
                                it.name = this.$t('route.contractIDE')
                                break;
                            case 'contractList':
                                it.name = this.$t('route.contractList')
                                break;
                            case 'hostMetrics':
                                it.name = this.$t('route.hostMetrics')
                                break;
                            case 'nodeMetrics':
                                it.name = this.$t('route.nodeMetrics')
                                break;
                            case 'privateKeyManagement':
                                it.name = this.$t('route.privateKeyManagement')
                                break;
                        }
                    })
                }

            }))
            return list;
        }
    },
    mounted() {
        this.$nextTick(function () {
            localStorage.setItem("sidebarHide", false);
            this.changeRouter();
            if (this.$refs.sidebarContent.offsetHeight > document.body.clientHeight) {
                this.buttomNone = false
            } else {
                this.buttomNone = true
            }
        });
        let _this = this
        window.onresize = () => {
            return (() => {
                if (_this.$refs.sidebarContent.offsetHeight > document.body.clientHeight) {
                    _this.buttomNone = false
                } else {
                    _this.buttomNone = true
                }
            })()
        }
    },
    methods: {
        changeRouter() {
            let list = this.$router.options.routes;

            list.forEach(item => {
                if (item.name === "帐号管理") {
                    item.menuShow = false;
                }
            });
            list.forEach(item => {
                if (this.userRole === "admin" && item.name === "帐号管理") {
                    item.menuShow = true;
                }
            });
            this.routesList = list;
        },
        select(index, indexPath) {
            this.activeIndex = indexPath[0];
            this.activeRoute = index;
        },
        handleOpen(key, keyPath) {
        },
        handleClose(key, keyPath) {
        },
        hideMune(val) {
            this.$emit("sidebarChange", val);
            if (this.menuShow) {
                this.menuShow = false;
            } else {
                this.menuShow = true;
            }
            if (this.$route.path === "contract" && val) {
                localStorage.setItem("sidebarHide", true);
            } else {
                localStorage.setItem("sidebarHide", false);
            }
        }
    }
};
</script>

<style scoped>
.sidebar-content {
    position: relative;
    overflow-y: auto;
    overflow-x: hidden;
}
.sidebar-version {
    width: 100%;
    padding: 20px 0;
    text-align: center;
    color: #92a1b3;
    border-top: 2px solid #20293c;
    background-color: #2c2e2f;
}
.group-content {
    position: relative;
    cursor: pointer;
}
.group-content ul {
    position: absolute;
    left: 20px;
    top: 35px;
    color: #666;
    z-index: 2;
    background-color: #fff;
    border: 1px solid #ebeef5;
    border-radius: 4px;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}
.group-content ul li {
    height: 32px;
    line-height: 32px;
    cursor: pointer;
    padding: 0 20px;
}
.group-content ul li:hover {
    background-color: #ecf5ff;
    color: #66b1ff;
}
.sidebar-check-group {
    color: #92a1b3;
    padding: 20px 0 20px 0px;
    border-top: 2px solid #2c2e2a;
    border-bottom: 2px solid #2c2e2a;
    text-align: center;
}
.sidebar-check-group >>> .el-dropdown {
    color: #fff;
}
.select-network {
    cursor: default;
}
.el-menu-vertical-demo >>> .is-active {
    color: #fff !important;
    background-color: #2c2e2f !important;
}
.el-menu-vertical-demo {
    padding-top: 14px;
    border: none;
}
.sidebar-content >>> .el-menu-item {
    font-size: 14px;
    color: #9da2ab;
    text-align: left;
    border-bottom: 1px solid #313437 !important;
}
.el-menu-vertical-demo >>> .el-submenu__title {
    padding-left: 33px;
}
.el-menu-item-group > ul > .el-menu-item {
    font-size: 14px;
    color: #9da2ab;
    text-align: left;
    padding-left: 57px !important;
    height: 44px;
    line-height: 44px;
    border-bottom: 1px solid #313437 !important;
}
/* .el-menu-vertical-demo>>> .el-icon-arrow-down:before {
    content: "\e60b"
} */
.sidebar-content >>> .el-menu--collapse {
    width: 56px;
}
.sidebar-content >>> .el-menu--collapse .is-active .el-tooltip {
    padding-left: 17px !important;
    color: #fff !important;
    background-color: #2c2e2f;
}
.mini-sidebar-contract-icon {
    position: relative;
    text-align: center;
}
.mini-sidebar-contract-icon i {
    position: absolute;
    top: 20px;
    right: 10px;
    z-index: 9999;
    cursor: pointer;
}

.image-flex {
    display: -webkit-box !important;
    display: -webkit-flex !important;
    display: -ms-flexbox !important;
    display: flex !important;
    -webkit-flex-wrap: wrap;
    -ms-flex-wrap: wrap;
    flex-wrap: wrap;
}
.image-flex,
.image-flex *,
.image-flex :after,
.image-flex :before {
}
.image-flex.justify-center {
    -webkit-box-pack: center;
    -webkit-justify-content: center;
    -ms-flex-pack: center;
    justify-content: center;
}
.image-flex.center {
    -webkit-box-pack: center;
    -webkit-justify-content: center;
    -ms-flex-pack: center;
    justify-content: center;
    -webkit-box-align: center;
    -webkit-align-items: center;
    -ms-flex-align: center;
    align-items: center;
}
.sidebar-icon {
    font-size: 15px;
    padding-right: 5px;
}
.sidebar-contract-icon {
    position: absolute;
    display: inline-block;
    left: 180px;
    top: 18px;
    font-size: 12px;
    letter-spacing: 0;
    text-align: right;
    cursor: pointer;
    z-index: 6666;
}
.sidebar-contract-icon i {
    cursor: pointer;
}
</style>
