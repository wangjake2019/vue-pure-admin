<script setup lang="ts">
import {
  h,
  ref,
  computed,
  Transition,
  defineComponent,
  getCurrentInstance
} from "vue";
import { RouterView } from "vue-router";
import { usePermissionStoreHook } from "/@/store/modules/permission";

const props = defineProps({
  fixedHeader: Boolean
});
const keepAlive: Boolean = ref(
  getCurrentInstance().appContext.config.globalProperties.$config?.KeepAlive
);

const transitions = computed(() => {
  return route => {
    return route.meta.transition;
  };
});

const transitionMain = defineComponent({
  render() {
    return h(
      Transition,
      {
        name:
          transitions.value(this.route) &&
          this.route.meta.transition.enterTransition
            ? "pure-classes-transition"
            : (transitions.value(this.route) &&
                this.route.meta.transition.name) ||
              "fade-transform",
        enterActiveClass:
          transitions.value(this.route) &&
          `animate__animated ${this.route.meta.transition.enterTransition}`,
        leaveActiveClass:
          transitions.value(this.route) &&
          `animate__animated ${this.route.meta.transition.leaveTransition}`,
        mode: "out-in",
        appear: true
      },
      {
        default: () => [this.$slots.default()]
      }
    );
  },
  props: {
    route: {
      type: undefined,
      required: true
    }
  }
});
</script>

<template>
  <section
    :class="[props.fixedHeader ? 'app-main' : 'app-main-nofixed-header']"
  >
    <router-view>
      <template #default="{ Component, route }">
        <el-scrollbar v-if="props.fixedHeader">
          <el-backtop target=".app-main .el-scrollbar__wrap"></el-backtop>
          <transitionMain :route="route">
            <keep-alive
              v-if="keepAlive"
              :include="usePermissionStoreHook().cachePageList"
            >
              <component :is="Component" :key="route.fullPath" />
            </keep-alive>
            <component v-else :is="Component" :key="route.fullPath" />
          </transitionMain>
        </el-scrollbar>
        <div v-else>
          <transitionMain :route="route">
            <keep-alive
              v-if="keepAlive"
              :include="usePermissionStoreHook().cachePageList"
            >
              <component :is="Component" :key="route.fullPath" />
            </keep-alive>
            <component v-else :is="Component" :key="route.fullPath" />
          </transitionMain>
        </div>
      </template>
    </router-view>
  </section>
</template>

<style scoped>
.app-main {
  width: 100%;
  height: 100vh;
  position: relative;
  overflow-x: hidden;
}

.app-main-nofixed-header {
  width: 100%;
  min-height: 100vh;
  position: relative;
}
</style>
