<template>
  <div
    v-if="documentTreeRoot"
    ref="sidebarElRef"
    class="relative pb-6 w-full h-full flex flex-col flex-shrink-0 bg-gray-50 border-r border-gray-200 transition-all overflow-y-auto"
  >
    <a
      v-if="shouldShowBackToMainDocs"
      :href="localePath('/docs')"
      exact
      class="pl-6 pr-1 mt-3 py-2 text-gray-500 text-sm border border-transparent border-r-0 whitespace-pre-wrap break-all hover:text-accent"
    >
      <span>← back to main docs</span>
    </a>
    <!-- Render document tree. We only support 4 level folder. -->
    <div
      v-for="(node, index) in documentTreeRoot.children"
      :key="index"
      class="w-full flex flex-col justify-start items-start"
    >
      <!-- separator -->
      <hr v-if="node.type === 'hr'" class="w-full my-1 mt-4" />
      <!-- root node -->
      <div
        v-else
        class="pl-3 mt-3 w-full flex flex-row justify-between cursor-pointer items-start"
        @click="
          node.displayChildren = !node.displayChildren;
          handleLinkClick();
        "
      >
        <div
          v-if="node.type === 'text'"
          class="pl-3 py-2 w-full flex flex-row justify-between items-start text-gray-600 font-bold text-sm border border-transparent border-r-0 whitespace-pre-wrap break-all hover:text-gray-700"
        >
          <span class="pr-1 leading-6">{{ node.title }}</span>
          <span
            v-if="node.children.length !== 0"
            class="flex-shrink-0 h-6 mr-5 flex flex-row justify-center items-center cursor-pointer select-none"
            @click.prevent.stop="node.displayChildren = !node.displayChildren"
          >
            <img
              class="relative w-4 h-auto transition-all opacity-60"
              :class="node.displayChildren ? 'rotate-90-arrow' : ''"
              src="~/assets/svg/chevron-right.svg"
              alt="toggle"
            />
          </span>
        </div>
        <nuxt-link
          v-else
          :to="
            node.type === 'page-link'
              ? localePath(`/docs${node.path}`)
              : { hash: node.path }
          "
          exact
          class="pl-3 py-2 w-full flex flex-row justify-between items-start text-gray-600 font-bold text-sm border border-transparent border-r-0 whitespace-pre-wrap break-all hover:text-gray-700"
        >
          <span class="pr-1 leading-6">{{ node.title }}</span>
          <span
            v-if="node.children.length !== 0"
            class="flex-shrink-0 h-6 mr-5 flex flex-row justify-center items-center cursor-pointer select-none"
            @click.prevent.stop="node.displayChildren = !node.displayChildren"
          >
            <img
              class="relative w-4 h-auto transition-all opacity-60"
              :class="node.displayChildren ? 'rotate-90-arrow' : ''"
              src="~/assets/svg/chevron-right.svg"
              alt="toggle"
            />
          </span>
        </nuxt-link>
      </div>
      <div
        v-for="subNode in node.children"
        v-show="node.displayChildren"
        :key="subNode.title"
        class="w-full flex flex-col justify-start items-start"
      >
        <!-- subnode -->
        <div
          class="pl-6 w-full"
          @click="
            subNode.displayChildren = !subNode.displayChildren;
            handleLinkClick();
          "
        >
          <nuxt-link
            :to="
              subNode.type === 'page-link'
                ? localePath(`/docs${subNode.path}`)
                : { hash: subNode.path }
            "
            class="pl-3 pr-1 py-2 flex flex-row justify-between items-center flex-shrink-0 text-gray-500 w-full text-sm border border-transparent border-r-0 whitespace-pre-wrap hover:text-gray-700"
          >
            <span>{{ subNode.title }}</span>
            <span
              v-if="subNode.children.length !== 0"
              class="flex-shrink-0 mr-4"
              @click.prevent.stop="
                subNode.displayChildren = !subNode.displayChildren
              "
            >
              <img
                class="relative w-4 h-auto transition-all opacity-60"
                :class="subNode.displayChildren ? 'rotate-90-arrow' : ''"
                src="~/assets/svg/chevron-right.svg"
                alt="toggle"
              />
            </span>
          </nuxt-link>
        </div>
        <!-- childnode -->
        <div
          v-for="childNode in subNode.children"
          v-show="subNode.displayChildren"
          :key="childNode.title"
          class="w-full"
          @click="handleLinkClick"
        >
          <div
            class="pl-9 w-full"
            @click="
              childNode.displayChildren = !childNode.displayChildren;
              handleLinkClick();
            "
          >
            <nuxt-link
              :to="
                childNode.type === 'page-link'
                  ? localePath(`/docs${childNode.path}`)
                  : { hash: childNode.path }
              "
              class="pl-3 pr-1 py-2 flex flex-row justify-between items-center flex-shrink-0 text-gray-500 w-full text-sm border border-transparent border-r-0 whitespace-pre-wrap hover:text-gray-700"
            >
              <span>{{ childNode.title }}</span>
              <span
                v-if="childNode.children.length !== 0"
                class="flex-shrink-0 mr-4"
                @click.prevent.stop="
                  childNode.displayChildren = !childNode.displayChildren
                "
              >
                <img
                  class="relative w-4 h-auto transition-all opacity-60"
                  :class="childNode.displayChildren ? 'rotate-90-arrow' : ''"
                  src="~/assets/svg/chevron-right.svg"
                  alt="toggle"
                />
              </span>
            </nuxt-link>
          </div>
          <!-- leaf nodes -->
          <div
            v-for="leafNode in childNode.children"
            v-show="childNode.displayChildren"
            :key="leafNode.title"
            class="pl-12 w-full"
            @click="handleLinkClick"
          >
            <nuxt-link
              :to="
                leafNode.type === 'page-link'
                  ? localePath(`/docs${leafNode.path}`)
                  : { hash: leafNode.path }
              "
              class="pl-3 pr-1 py-2 block flex-shrink-0 text-gray-500 w-full text-sm border border-transparent border-r-0 whitespace-pre-wrap hover:text-gray-700"
            >
              <span>{{ leafNode.title }}</span>
            </nuxt-link>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import {
  computed,
  defineComponent,
  ref,
  onMounted,
  useContext,
  useFetch,
  watch,
} from "@nuxtjs/composition-api";
import { last } from "lodash";
import { ContentDocument, DocumentTreeNode } from "~/types/docs";
import { validDocsCategoryList } from "~/common/const";

const getDocumentTreeRoot = (documentList: any[]): DocumentTreeNode => {
  if (!documentList || !Array.isArray(documentList)) {
    return {
      path: "",
      title: "",
      type: "text",
      children: [],
      displayChildren: true,
    };
  }

  const documentTreeRoot = {
    path: "",
    title: "",
    type: "text",
    children: [],
    displayChildren: true,
  } as DocumentTreeNode;

  for (const document of documentList) {
    if (document.level === 1) {
      documentTreeRoot.children.push({
        title: document.title,
        path: document.path,
        type: document.type,
        children: [],
        displayChildren: document.displayChildren,
      });
    } else if (document.level === 2) {
      const parentNode = last(documentTreeRoot.children);
      if (parentNode) {
        parentNode.children.push({
          title: document.title,
          path: document.path,
          type: document.type,
          children: [],
          displayChildren: false,
        });
      }
    } else if (document.level === 3) {
      const parentNode = last(last(documentTreeRoot.children)?.children);
      if (parentNode) {
        parentNode.children.push({
          title: document.title,
          path: document.path,
          type: document.type,
          children: [],
          displayChildren: false,
        });
      }
    } else if (document.level === 4) {
      const parentNode = last(
        last(last(documentTreeRoot.children)?.children)?.children
      );
      if (parentNode) {
        parentNode.children.push({
          title: document.title,
          path: document.path,
          type: document.type,
          children: [],
          displayChildren: false,
        });
      }
    }
  }

  return documentTreeRoot;
};

export default defineComponent({
  emits: ["link-click"],
  setup(_, { emit }) {
    const { $content, route } = useContext();
    const sidebarElRef = ref<HTMLDivElement>();
    const documentTreeRoot = ref<DocumentTreeNode>();

    const category = route.value.params.category;
    const validDocsCategoryPathList = validDocsCategoryList.map(
      (t) => t.category
    );
    const shouldShowBackToMainDocs = computed(() =>
      validDocsCategoryPathList.includes(category)
    );

    useFetch(async () => {
      // Valid category for separate menu items. e.g. "cli"
      // Now we don't have a needed and finished submenus, so it's empty.
      const layout = (await $content(
        "docs",
        validDocsCategoryPathList.includes(category) ? category : "",
        "_layout",
        {
          deep: true,
        }
      ).fetch()) as any as ContentDocument;

      const nodes = layout.body.children
        .filter(
          (n) =>
            n.tag === "h2" ||
            n.tag === "h3" ||
            n.tag === "h4" ||
            n.tag === "h5" ||
            n.tag === "hr"
        )
        .map((n) => {
          let level = 1;
          if (n.tag === "h3") {
            level = 2;
          } else if (n.tag === "h4") {
            level = 3;
          } else if (n.tag === "h5") {
            level = 4;
          } else if (n.tag === "hr") {
            return {
              level: 1,
              type: n.tag,
              displayChildren: false,
            };
          }

          const child = n.children[1];
          let path = undefined;
          let title = "";
          let type = "text";
          let displayChildren = false;

          if (child.type !== "text") {
            if (child.props.to) {
              type = "page-link";
              path = child.props.to;
            } else {
              type = "hash-link";
              path = child.props.href;
            }
            title = child.children[0].value;
          } else {
            title = child.value;
          }

          if (
            Array.isArray(layout.expandSectionList) &&
            layout.expandSectionList.includes(title)
          ) {
            displayChildren = true;
          }

          return {
            level,
            title,
            path,
            type,
            displayChildren,
          };
        });

      documentTreeRoot.value = getDocumentTreeRoot(nodes) as DocumentTreeNode;
    });

    const relocateSidebar = () => {
      const pathname = window.location.pathname;

      if (documentTreeRoot.value) {
        for (const rootNode of documentTreeRoot.value.children) {
          for (const subNode of rootNode.children) {
            for (const childNode of subNode.children) {
              for (const leafNode of childNode.children) {
                if (pathname.includes(`/docs${leafNode.path}`)) {
                  rootNode.displayChildren = true;
                  subNode.displayChildren = true;
                  childNode.displayChildren = true;
                  break;
                }
              }
              if (pathname.includes(`/docs${childNode.path}`)) {
                rootNode.displayChildren = true;
                subNode.displayChildren = true;
                break;
              }
            }
            if (pathname.includes(`/docs${subNode.path}`)) {
              rootNode.displayChildren = true;
              break;
            }
          }
        }
      }

      if (!sidebarElRef.value) {
        return;
      }

      for (const anchorEl of Array.from(
        sidebarElRef.value.querySelectorAll("a")
      )) {
        let href = anchorEl.getAttribute("href");
        if (pathname.endsWith("/")) {
          href = href + "/";
        }

        if (pathname === href) {
          if (anchorEl.offsetTop > sidebarElRef.value.clientHeight) {
            sidebarElRef.value.scrollTo(0, anchorEl.offsetTop / 1.5);
          }
          break;
        }
      }
    };

    onMounted(() => {
      relocateSidebar();
    });

    watch(route, () => {
      relocateSidebar();
    });

    const handleLinkClick = () => {
      emit("link-click");
    };

    return {
      documentTreeRoot,
      sidebarElRef,
      shouldShowBackToMainDocs,
      handleLinkClick,
    };
  },
});
</script>

<style scoped>
.router-active-link {
  @apply bg-white text-accent no-underline border-gray-200;
}
.rotate-90-arrow {
  transform: rotate(90deg);
}
</style>
