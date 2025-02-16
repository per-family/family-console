<script lang="ts" setup>
// core libs
import { inject } from "vue";
import { RouterLink } from "vue-router";

// components
import {
  VAlert,
  VSpace,
  VTag,
  IconMore,
  VButton,
  Dialog,
} from "@halo-dev/components";

// types
import type { ComputedRef, Ref } from "vue";
import type { Theme } from "@halo-dev/api-client";

import { apiClient } from "@/utils/api-client";

const selectedTheme = inject<Ref<Theme | undefined>>("selectedTheme");
const isActivated = inject<ComputedRef<boolean>>("isActivated");

const handleReloadThemeSetting = async () => {
  Dialog.warning({
    title: "是否确认刷新主题的设置表单？",
    description: "此操作仅会刷新主题的设置表单，不会删除已有的配置。",
    onConfirm: async () => {
      try {
        if (!selectedTheme?.value) {
          return;
        }

        await apiClient.theme.reloadThemeSetting({
          name: selectedTheme.value.metadata.name,
        });

        window.location.reload();
      } catch (e) {
        console.error("Failed to reload theme setting", e);
      }
    },
  });
};
</script>

<template>
  <div class="bg-white px-4 py-4 sm:px-6">
    <div class="group flex items-center justify-between">
      <div class="flex flex-row gap-3">
        <div
          class="h-12 w-12 overflow-hidden rounded border bg-white hover:shadow-sm"
        >
          <img
            :key="selectedTheme?.metadata.name"
            :alt="selectedTheme?.spec.displayName"
            :src="selectedTheme?.spec.logo"
            class="h-full w-full"
          />
        </div>
        <div>
          <h3 class="text-lg font-medium leading-6 text-gray-900">
            {{ selectedTheme?.spec.displayName }}
          </h3>
          <p class="mt-1 flex max-w-2xl items-center gap-2">
            <span class="text-sm text-gray-500">
              {{ selectedTheme?.spec.version }}
            </span>
            <VTag>
              {{ isActivated ? "当前启用" : "未启用" }}
            </VTag>
          </p>
        </div>
      </div>
      <FloatingDropdown v-permission="['system:themes:manage']">
        <div
          class="cursor-pointer rounded p-1 transition-all hover:text-blue-600 group-hover:bg-gray-100"
        >
          <IconMore />
        </div>
        <template #popper>
          <div class="w-48 p-2">
            <VSpace class="w-full" direction="column">
              <VButton
                v-close-popper
                block
                type="secondary"
                @click="handleReloadThemeSetting"
              >
                刷新设置表单
              </VButton>
            </VSpace>
          </div>
        </template>
      </FloatingDropdown>
    </div>
  </div>
  <div class="border-t border-gray-200">
    <dl class="divide-y divide-gray-100">
      <div
        class="bg-white px-4 py-5 hover:bg-gray-50 sm:grid sm:grid-cols-6 sm:gap-4 sm:px-6"
      >
        <dt class="text-sm font-medium text-gray-900">ID</dt>
        <dd class="mt-1 text-sm text-gray-900 sm:col-span-3 sm:mt-0">
          {{ selectedTheme?.metadata.name }}
        </dd>
      </div>
      <div
        class="bg-white px-4 py-5 hover:bg-gray-50 sm:grid sm:grid-cols-6 sm:gap-4 sm:px-6"
      >
        <dt class="text-sm font-medium text-gray-900">作者</dt>
        <dd class="mt-1 text-sm text-gray-900 sm:col-span-3 sm:mt-0">
          {{ selectedTheme?.spec.author.name }}
        </dd>
      </div>
      <div
        class="bg-white px-4 py-5 hover:bg-gray-50 sm:grid sm:grid-cols-6 sm:gap-4 sm:px-6"
      >
        <dt class="text-sm font-medium text-gray-900">网站</dt>
        <dd class="mt-1 text-sm text-gray-900 sm:col-span-3 sm:mt-0">
          <a :href="selectedTheme?.spec.website" target="_blank">
            {{ selectedTheme?.spec.website }}
          </a>
        </dd>
      </div>
      <div
        class="bg-white px-4 py-5 hover:bg-gray-50 sm:grid sm:grid-cols-6 sm:gap-4 sm:px-6"
      >
        <dt class="text-sm font-medium text-gray-900">源码仓库</dt>
        <dd class="mt-1 text-sm text-gray-900 sm:col-span-3 sm:mt-0">
          <a :href="selectedTheme?.spec.repo" target="_blank">
            {{ selectedTheme?.spec.repo }}
          </a>
        </dd>
      </div>
      <div
        class="bg-white px-4 py-5 hover:bg-gray-50 sm:grid sm:grid-cols-6 sm:gap-4 sm:px-6"
      >
        <dt class="text-sm font-medium text-gray-900">当前版本</dt>
        <dd class="mt-1 text-sm text-gray-900 sm:col-span-3 sm:mt-0">
          {{ selectedTheme?.spec.version }}
        </dd>
      </div>
      <div
        class="bg-white px-4 py-5 hover:bg-gray-50 sm:grid sm:grid-cols-6 sm:gap-4 sm:px-6"
      >
        <dt class="text-sm font-medium text-gray-900">Halo 版本要求</dt>
        <dd class="mt-1 text-sm text-gray-900 sm:col-span-3 sm:mt-0">
          {{ selectedTheme?.spec.require }}
        </dd>
      </div>
      <div
        class="bg-white px-4 py-5 hover:bg-gray-50 sm:grid sm:grid-cols-6 sm:gap-4 sm:px-6"
      >
        <dt class="text-sm font-medium text-gray-900">存储位置</dt>
        <dd class="mt-1 text-sm text-gray-900 sm:col-span-3 sm:mt-0">
          {{ selectedTheme?.status?.location }}
        </dd>
      </div>
      <!-- TODO: add display required plugins support -->
      <div
        v-if="false"
        class="bg-white px-4 py-5 hover:bg-gray-50 sm:grid sm:grid-cols-6 sm:gap-4 sm:px-6"
      >
        <dt class="text-sm font-medium text-gray-900">插件依赖</dt>
        <dd class="mt-1 text-sm sm:col-span-3 sm:mt-0">
          <VAlert description="当前有 1 个插件还未安装" title="提示"></VAlert>
          <ul class="mt-2 space-y-2">
            <li>
              <div
                class="inline-flex w-96 cursor-pointer flex-col gap-y-3 rounded border p-5 hover:border-primary"
              >
                <RouterLink
                  :to="{
                    name: 'PluginDetail',
                    params: { name: 'PluginLinks' },
                  }"
                  class="font-medium text-gray-900 hover:text-blue-400"
                >
                  run.halo.plugins.links
                </RouterLink>
                <div class="text-xs">
                  <VSpace>
                    <VTag> 已安装</VTag>
                  </VSpace>
                </div>
              </div>
            </li>
            <li>
              <div
                class="inline-flex w-96 cursor-pointer flex-col gap-y-3 rounded border p-5 hover:border-primary"
              >
                <span class="font-medium hover:text-blue-400">
                  run.halo.plugins.photos
                </span>
                <div class="text-xs">
                  <VSpace>
                    <VTag>未安装</VTag>
                  </VSpace>
                </div>
              </div>
            </li>
          </ul>
        </dd>
      </div>
    </dl>
  </div>
</template>
