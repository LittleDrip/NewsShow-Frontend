<script setup>
import {
    Management,
    Promotion,
    UserFilled,
    User,
    Crop,
    EditPen,
    SwitchButton,
    CaretBottom,
} from "@element-plus/icons-vue"
import avatar from "@/assets/default.png"
import { userInfoService } from "@/api/user.js"
import { useUserInfoStore } from "@/stores/userInfo.js";
import { useRouter } from "vue-router";
import { ElMessage, ElMessageBox } from "element-plus";
import { useTokenStore } from "@/stores/token";
const tokenStore = useTokenStore()
const userInfoStore = useUserInfoStore()

// 调用用户函数
const getUserInfo = async () => {
    let res = await userInfoService();
    userInfoStore.setInfo(res.data);
}
getUserInfo();
const router = useRouter();
const handleCommand = (command) => {
    // 判断指令
    if (command == 'logout') {
        ElMessageBox.confirm(
            '是否退出登录?',
            '温馨提示',
            {
                confirmButtonText: '是',
                cancelButtonText: '否',
                type: 'warning',
            }
        )
            .then(async () => {
                // 退出登录 清空token和个人信息
                tokenStore.removeToken();
                userInfoStore.removeInfo();
                router.push('/login')
                ElMessage({

                    type: 'success',
                    message: '退出登录成功',
                })

            })
            .catch(() => {
                ElMessage({
                    type: 'info',
                    message: '用户取消了退出登录',
                })
            })
    } else {
        router.push('/user/' + command)
    }
}
</script>

<template>
    <el-container class="layout-container">
        <!-- 左侧菜单 -->
        <el-aside width="200px">
            <div class="el-aside__logo"></div>
            <!-- el-menu 菜单标签 -->
            <el-menu active-text-color="#ffd04b" background-color="##cbe0b5" text-color="#111" router>
                <el-menu-item index="/article/category">
                    <el-icon>
                        <Management />
                    </el-icon>
                    <span>文章分类</span>
                </el-menu-item>
                <el-menu-item index="/article/manage">
                    <el-icon>
                        <Promotion />
                    </el-icon>
                    <span>文章管理</span>
                </el-menu-item>
                <el-sub-menu>
                    <template #title>
                        <el-icon>
                            <UserFilled />
                        </el-icon>
                        <span>个人中心</span>
                    </template>

                    <el-menu-item index="/user/info">
                        <el-icon>
                            <User />
                        </el-icon>
                        <span>基本资料</span>
                    </el-menu-item>
                    <el-menu-item index="/user/avatar">
                        <el-icon>
                            <Crop />
                        </el-icon>
                        <span>更换头像</span>
                    </el-menu-item>
                    <el-menu-item index="/user/resetPassword">
                        <el-icon>
                            <EditPen />
                        </el-icon>
                        <span>重置密码</span>
                    </el-menu-item>
                </el-sub-menu>
            </el-menu>
        </el-aside>
        <!-- 右侧主区域 -->
        <el-container>
            <!-- 头部区域 -->
            <el-header>
                <div>欢迎你：<strong>{{ userInfoStore.info.nickname }}</strong></div>
                <el-dropdown placement="bottom-end" @command="handleCommand">
                    <span class="el-dropdown__box">
                        <el-avatar :src="userInfoStore.info.userPic ? userInfoStore.info.userPic : avatar" />
                        <el-icon>
                            <CaretBottom />
                        </el-icon>
                    </span>
                    <template #dropdown>
                        <el-dropdown-menu>
                            <el-dropdown-item command="info" :icon="User">基本资料</el-dropdown-item>
                            <el-dropdown-item command="avatar" :icon="Crop">更换头像</el-dropdown-item>
                            <el-dropdown-item command="resetPassword" :icon="EditPen">重置密码</el-dropdown-item>
                            <el-dropdown-item command="logout" :icon="SwitchButton">退出登录</el-dropdown-item>
                        </el-dropdown-menu>
                    </template>
                </el-dropdown>
            </el-header>
            <!-- 中间区域 -->
            <el-main>
                <!-- <div style="width: 1290px; height: 570px; border: 1px solid red">内容展示区</div> -->
                <router-view></router-view>
            </el-main>
            <!-- 底部区域 -->

        </el-container>
    </el-container>
</template>

<style lang="scss" scoped>
.layout-container {
    height: 100vh;

    .el-aside {
        background-color: #cce1eb;

        &__logo {
            height: 120px;
            background: url("@/assets/logo.png") no-repeat center / 120px auto;
        }

        .el-menu {
            border-right: none;
        }
    }

    .el-header {
        background-color: #ffffff;
        display: flex;
        align-items: center;
        justify-content: space-between;

        .el-dropdown__box {
            display: flex;
            align-items: center;

            .el-icon {
                color: #999999;
                margin-left: 10px;
            }

            &:active,
            &:focus {
                outline: none;
            }
        }
    }

    .el-footer {
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 14px;
        color: #666;
    }
}

.el-menu-item.is-active {
    background-color: #3370ff !important;
    color: #fff;

    span {
        color: #fff !important;
    }
}


.el-menu-item:hover {
    outline: 0 !important;
    color: #1a5fa4 !important;
    background: linear-gradient(270deg, #F2F7FC 0%, #FEFEFE 100%) !important;
}
</style>
