<template>
    <div style="padding: 20px">
        <div v-if="this.devicesGroup.length == 0">
            暂无设备
        </div>
        <el-collapse accordion v-for="item in devicesGroup" v-bind:key="item.deviceCode">
            <el-collapse-item>
                <template slot="title">
                    <div style="display: inline-block; width: 35%">
                        {{item.deviceName}}{{alias(item)}}
                    </div>
                    <div style="display: inline-block; width: 35%">
                        设备唯一识别码：{{item.deviceCode}}
                    </div>
                    <div style="display: inline-block; width: 30%; text-align: right">
                        <el-link style="margin-right: 20px" type="primary" :underline="false" @click="setAlias(item.deviceCode, item.alias, $event)">设置别名</el-link>
                        <el-link style="margin-right: 20px" type="primary" :underline="false" @click="shock(item.deviceCode, $event)">震动反馈</el-link>
                    </div>
                </template>
                <div v-for="app in item.appInfoList" v-bind:key="app.applicationName">
                    <div style="display: inline-block; width: 35%">
                        <el-link type="primary" v-bind:href="'http://' + app.address + ':' + app.port" :underline="false" target="view_window">{{app.applicationName}}</el-link>
                    </div>
                    <div style="display: inline-block; width: 65%">
                        {{app.address}}:{{app.port}}
                    </div>
                </div>
            </el-collapse-item>
        </el-collapse>
    </div>
</template>

<script>
    import * as URL from './UrlConstant'
    export default {
        name: 'app',
        data() {
            return {
                devicesGroup: []
            }
        },
        methods: {
            getDevice() {
                this.axios.get(URL.GET_DEVICE).then((resp) => {
                    if (resp.data.success) {
                        this.devicesGroup = resp.data.data
                    } else {
                        this.$message({
                            showClose: true,
                            message: resp.data.message,
                            type: 'error',
                        })
                    }
                })
            },
            setAlias(code, alias, e) {
                this.$prompt('请输入别名', "设备唯一识别码：" + code, {
                    inputValue: alias
                }).then(({ value }) => {
                    this.axios.get(URL.SET_ALIAS, {
                        params: {
                            code: code,
                            alias: value
                        }
                    }).then((resp) => {
                        if (resp.data.success) {
                            this.getDevice()
                        } else {
                            this.$message({
                                showClose: true,
                                message: resp.data.message,
                                type: 'error',
                            })
                        }
                    })
                })
                e.cancelBubble = true;
            },
            shock(code, e) {
                this.axios.get(URL.SHOCK, {
                    params: {
                        code: code
                    }
                }).then((resp) => {
                    if (!resp.data.success) {
                        this.$message({
                            showClose: true,
                            message: resp.data.message,
                            type: 'error',
                        })
                    }
                })
                e.cancelBubble = true;
            },
            alias(item) {
                if ("" === item.alias) {
                    return ""
                }
                return "（" + item.alias + "）"
            },
        },
        mounted() {
            this.getDevice()
        }
    }
</script>

<style>

</style>
