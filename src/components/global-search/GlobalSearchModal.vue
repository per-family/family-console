<script lang="ts" setup>
import { useRoute, useRouter, type RouteLocationRaw } from "vue-router";
import {
  VModal,
  VEntity,
  VEntityField,
  IconLink,
  IconBookRead,
  IconFolder,
  IconSettings,
  IconPalette,
  IconPages,
  IconUserSettings,
} from "@halo-dev/components";
import { computed, markRaw, onMounted, ref, watch, type Component } from "vue";
import Fuse from "fuse.js";
import { apiClient } from "@/utils/api-client";
import { usePermission } from "@/utils/permission";

const router = useRouter();
const route = useRoute();

const { currentUserHasPermission } = usePermission();

const props = withDefaults(
  defineProps<{
    visible: boolean;
  }>(),
  {
    visible: false,
  }
);

const emit = defineEmits<{
  (e: "update:visible", visible: boolean): void;
}>();

const globalSearchInput = ref<HTMLInputElement | null>(null);
const keyword = ref("");

interface SearchableItem {
  title: string;
  icon: { component: Component } | { src: string };
  group?: string;
  route: RouteLocationRaw;
}

const searchableItem: SearchableItem[] = [];
const selectedIndex = ref(0);

const fuse = new Fuse(searchableItem, {
  keys: ["title", "group", "route.path", "route.name"],
  useExtendedSearch: true,
});

const searchResults = computed((): SearchableItem[] => {
  return fuse
    .search(keyword.value, {
      limit: 20,
    })
    .map((result) => result.item);
});

const handleBuildSearchIndex = () => {
  const routes = router.getRoutes().filter((route) => {
    return !!route.meta?.title && route.meta?.searchable;
  });

  routes.forEach((route) => {
    fuse.add({
      title: route.meta?.title as string,
      icon: {
        component: markRaw(IconLink),
      },
      group: "后台页面",
      route,
    });
  });

  if (currentUserHasPermission(["system:users:view"])) {
    apiClient.extension.user.listv1alpha1User().then((response) => {
      response.data.items.forEach((user) => {
        fuse.add({
          title: user.spec.displayName,
          icon: {
            component: markRaw(IconUserSettings),
          },
          group: "用户",
          route: {
            name: "UserDetail",
            params: {
              name: user.metadata.name,
            },
          },
        });
      });
    });
  }

  if (currentUserHasPermission(["system:plugins:view"])) {
    apiClient.extension.plugin
      .listpluginHaloRunV1alpha1Plugin()
      .then((response) => {
        response.data.items.forEach((plugin) => {
          fuse.add({
            title: plugin.spec.displayName as string,
            icon: {
              src: plugin.spec.logo as string,
            },
            group: "插件",
            route: {
              name: "PluginDetail",
              params: {
                name: plugin.metadata.name,
              },
            },
          });
        });
      });
  }

  if (currentUserHasPermission(["system:posts:view"])) {
    apiClient.extension.post
      .listcontentHaloRunV1alpha1Post()
      .then((response) => {
        response.data.items.forEach((post) => {
          fuse.add({
            title: post.spec.title,
            icon: {
              component: markRaw(IconBookRead),
            },
            group: "文章",
            route: {
              name: "PostEditor",
              query: {
                name: post.metadata.name,
              },
            },
          });
        });
      });

    apiClient.extension.category
      .listcontentHaloRunV1alpha1Category()
      .then((response) => {
        response.data.items.forEach((category) => {
          fuse.add({
            title: category.spec.displayName,
            icon: {
              component: markRaw(IconBookRead),
            },
            group: "分类",
            route: {
              name: "Categories",
              query: {
                name: category.metadata.name,
              },
            },
          });
        });
      });

    apiClient.extension.tag.listcontentHaloRunV1alpha1Tag().then((response) => {
      response.data.items.forEach((tag) => {
        fuse.add({
          title: tag.spec.displayName,
          icon: {
            component: markRaw(IconBookRead),
          },
          group: "标签",
          route: {
            name: "Tags",
            query: {
              name: tag.metadata.name,
            },
          },
        });
      });
    });
  }

  if (currentUserHasPermission(["system:singlepages:view"])) {
    apiClient.extension.singlePage
      .listcontentHaloRunV1alpha1SinglePage()
      .then((response) => {
        response.data.items.forEach((singlePage) => {
          fuse.add({
            title: singlePage.spec.title,
            icon: {
              component: markRaw(IconPages),
            },
            group: "自定义页面",
            route: {
              name: "SinglePageEditor",
              query: {
                name: singlePage.metadata.name,
              },
            },
          });
        });
      });
  }

  if (currentUserHasPermission(["system:attachments:view"])) {
    apiClient.extension.storage.attachment
      .liststorageHaloRunV1alpha1Attachment()
      .then((response) => {
        response.data.items.forEach((attachment) => {
          fuse.add({
            title: attachment.spec.displayName as string,
            icon: {
              component: markRaw(IconFolder),
            },
            group: "附件",
            route: {
              name: "Attachments",
              query: {
                name: attachment.metadata.name,
              },
            },
          });
        });
      });
  }

  if (
    currentUserHasPermission(["system:settings:view"]) &&
    currentUserHasPermission(["system:configmaps:view"])
  ) {
    apiClient.extension.setting
      .getv1alpha1Setting({ name: "system" })
      .then((response) => {
        response.data.spec.forms.forEach((form) => {
          fuse.add({
            title: form.label as string,
            icon: {
              component: markRaw(IconSettings),
            },
            group: "设置",
            route: {
              name: "SystemSetting",
              params: {
                group: form.group,
              },
            },
          });
        });
      });

    // get theme settings
    apiClient.extension.configMap
      .getv1alpha1ConfigMap({
        name: "system",
      })
      .then(({ data: systemConfigMap }) => {
        if (systemConfigMap.data?.theme) {
          const themeConfig = JSON.parse(systemConfigMap.data.theme);

          apiClient.extension.theme
            .getthemeHaloRunV1alpha1Theme({
              name: themeConfig.active,
            })
            .then(({ data: theme }) => {
              if (theme && theme.spec.settingName) {
                apiClient.extension.setting
                  .getv1alpha1Setting({
                    name: theme.spec.settingName,
                  })
                  .then(({ data: themeSettings }) => {
                    themeSettings.spec.forms.forEach((form) => {
                      fuse.add({
                        title: `${theme.spec.displayName} / ${form.label}`,
                        icon: {
                          component: markRaw(IconPalette),
                        },
                        group: "主题设置",
                        route: {
                          name: "ThemeSetting",
                          params: {
                            group: form.group,
                          },
                        },
                      });
                    });
                  });
              }
            });
        }
      });
  }
};

const handleKeydown = (e: KeyboardEvent) => {
  if (!props.visible) {
    return;
  }

  const { key, ctrlKey } = e;

  if (key === "ArrowUp" || (key === "k" && ctrlKey)) {
    selectedIndex.value = Math.max(0, selectedIndex.value - 1);
    e.preventDefault();
  }

  if (key === "ArrowDown" || (key === "j" && ctrlKey)) {
    selectedIndex.value = Math.min(
      searchResults.value.length - 1,
      selectedIndex.value + 1
    );
    e.preventDefault();
  }

  if (key === "Enter") {
    const searchResult = searchResults.value[selectedIndex.value];
    handleRoute(searchResult);
  }

  if (key === "Escape") {
    onVisibleChange(false);
    e.preventDefault();
  }
};

const handleRoute = async (item: SearchableItem) => {
  // if route has query params, need router.go(0)
  if (typeof item.route !== "string" && "query" in item.route) {
    if ("name" in item.route && route.name === item.route.name) {
      await router.push(item.route);
      router.go(0);
      return;
    }
  }
  router.push(item.route);
  emit("update:visible", false);
};

watch(
  () => selectedIndex.value,
  (index) => {
    if (index > 0) {
      document.getElementById(`search-item-${index}`)?.scrollIntoView();
      return;
    }
    document.getElementById("search-input")?.scrollIntoView();
  }
);

watch(
  () => keyword.value,
  () => {
    selectedIndex.value = 0;
  }
);

watch(
  () => props.visible,
  (visible) => {
    if (visible) {
      setTimeout(() => {
        globalSearchInput.value?.focus();
      }, 100);

      document.addEventListener("keydown", handleKeydown);
    } else {
      document.removeEventListener("keydown", handleKeydown);
      keyword.value = "";
      selectedIndex.value = 0;
    }
  }
);

onMounted(() => {
  handleBuildSearchIndex();
});

const onVisibleChange = (visible: boolean) => {
  emit("update:visible", visible);
};
</script>

<template>
  <VModal
    :visible="visible"
    :body-class="['!p-0']"
    :mount-to-body="true"
    :width="650"
    :centered="false"
    @update:visible="onVisibleChange"
  >
    <div id="search-input" class="border-b border-gray-100 px-4 py-2.5">
      <input
        ref="globalSearchInput"
        v-model="keyword"
        placeholder="输入关键词以搜索"
        class="w-full py-1 text-base outline-none"
        autocomplete="off"
        autocorrect="off"
        spellcheck="false"
      />
    </div>
    <div class="px-2 py-2.5">
      <div
        v-if="!searchResults.length"
        class="flex items-center justify-center text-sm text-gray-500"
      >
        <span>没有搜索结果</span>
      </div>
      <ul
        v-if="searchResults.length > 0"
        class="box-border flex h-full w-full flex-col gap-0.5"
        role="list"
      >
        <li
          v-for="(item, itemIndex) in searchResults"
          :id="`search-item-${itemIndex}`"
          :key="itemIndex"
          @click="handleRoute(item)"
        >
          <VEntity
            class="rounded-md px-2 py-2.5 hover:bg-gray-100"
            :class="{ 'bg-gray-100': selectedIndex === itemIndex }"
          >
            <template #start>
              <VEntityField>
                <template #description>
                  <div class="h-5 w-5 rounded border p-0.5">
                    <component
                      :is="item.icon.component"
                      v-if="'component' in item.icon"
                      class="h-full w-full"
                    />
                    <img
                      v-if="'src' in item.icon"
                      :src="item.icon.src"
                      class="h-full w-full object-cover"
                    />
                  </div>
                </template>
              </VEntityField>
              <VEntityField :title="item.title"></VEntityField>
            </template>
            <template #end>
              <VEntityField :description="item.group"></VEntityField>
            </template>
          </VEntity>
        </li>
      </ul>
    </div>
    <div class="border-t border-gray-100 px-4 py-2.5">
      <div class="flex items-center justify-end">
        <span class="mr-1 text-xs text-gray-600">选择</span>
        <kbd
          class="mr-1 w-5 rounded border p-0.5 text-center text-[10px] text-gray-600 shadow-sm"
        >
          ↑
        </kbd>
        <kbd
          class="mr-5 w-5 rounded border p-0.5 text-center text-[10px] text-gray-600 shadow-sm"
        >
          ↓
        </kbd>
        <span class="mr-1 text-xs text-gray-600">确认</span>
        <kbd
          class="mr-5 rounded border p-0.5 text-[10px] text-gray-600 shadow-sm"
        >
          Enter
        </kbd>
        <span class="mr-1 text-xs text-gray-600">关闭</span>
        <kbd class="rounded border p-0.5 text-[10px] text-gray-600 shadow-sm">
          Esc
        </kbd>
      </div>
    </div>
  </VModal>
</template>
