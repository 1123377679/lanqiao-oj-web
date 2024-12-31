<template>
  <a-row id="globalHeader" align="center" :wrap="false">
    <a-col flex="auto">
      <a-menu
        mode="horizontal"
        :selected-keys="selectedKeys"
        @menu-item-click="doMenuClick"
      >
        <a-menu-item
          key="0"
          :style="{ padding: 0, marginRight: '38px' }"
          disabled
        >
          <div class="title-bar">
            <img
              class="logo"
              src="../assets/云课.png"
              style="background-color: #00a8eb"
            />
            <div class="title">蓝桥 OJ</div>
          </div>
        </a-menu-item>
        <a-menu-item v-for="item in visibleRoutes" :key="item.path">
          {{ item.name }}
        </a-menu-item>
      </a-menu>
    </a-col>
    <a-col flex="160px">
      <div class="user-info">
        <span>{{ store.state.user?.loginUser?.userName ?? "未登录" }}</span>
        <!-- 添加退出按钮，仅在用户登录时显示 -->
        <a-button
          v-if="store.state.user?.loginUser"
          type="text"
          size="small"
          @click="showLogoutConfirm"
          style="top: -5px"
        >
          <icon-export />
          退出
        </a-button>
      </div>
    </a-col>
  </a-row>
</template>

<script setup lang="ts">
import { routes } from "../router/routes";
import { useRoute, useRouter } from "vue-router";
import { computed, ref } from "vue";
import { useStore } from "vuex";
import checkAccess from "@/access/checkAccess";
import ACCESS_ENUM from "@/access/accessEnum";
import { Message, Modal } from "@arco-design/web-vue";
import axios from "axios";

const router = useRouter();
const store = useStore();

// 添加退出确认方法
const showLogoutConfirm = () => {
  Modal.warning({
    title: "确认退出",
    content: "是否确认退出登录？",
    okText: "确认",
    cancelText: "取消",
    async onOk() {
      try {
        const res = await axios.post(
          "http://localhost:8121/api/user/logout",
          {},
          {
            headers: {
              "Content-Type": "application/json",
            },
          }
        );

        if (res.data.code === 0 && res.data.data) {
          Message.success("退出成功");
          await store.dispatch("user/logout");
          router.push({
            path: "/user/login",
            replace: true,
          });
        } else {
          Message.error(res.data.message || "退出失败");
        }
      } catch (error: any) {
        Message.error(error.response?.data?.message || "退出失败，请重试");
      }
    },
  });
};

// 展示在菜单的路由数组
const visibleRoutes = computed(() => {
  return routes.filter((item, index) => {
    if (item.meta?.hideInMenu) {
      return false;
    }
    // 根据权限过滤菜单
    if (
      !checkAccess(store.state.user.loginUser, item?.meta?.access as string)
    ) {
      return false;
    }
    return true;
  });
});

// 默认主页
const selectedKeys = ref(["/"]);

// 路由跳转后，更新选中的菜单项
router.afterEach((to, from, failure) => {
  selectedKeys.value = [to.path];
});

console.log();

setTimeout(() => {
  store.dispatch("user/getLoginUser", {
    userName: "李某人管理员",
    userRole: ACCESS_ENUM.ADMIN,
  });
}, 3000);

const doMenuClick = (key: string) => {
  router.push({
    path: key,
  });
};
</script>

<style scoped>
.title-bar {
  display: flex;
  align-items: center;
}

.title {
  color: #444;
  margin-left: 16px;
}

.logo {
  height: 48px;
}
.user-info {
  top: -5px;
}

/* .logout-btn {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
} */
</style>
