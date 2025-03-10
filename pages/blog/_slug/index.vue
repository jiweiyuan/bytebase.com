<template>
  <div id="content" class="space-y-8">
    <div class="hidden sm:flex sm:h-96 w-full">
      <img
        v-if="blog.featureImage"
        class="w-full h-auto object-scale-down"
        :src="blog.featureImage"
      />
    </div>
    <div class="prose prose-xl mx-auto px-4">
      <div class="flex h-8 items-center justify-between">
        <div
          v-for="(tag, tagIndex) in blog.tags"
          :key="tagIndex"
          class="mb-4 inline-flex"
        >
          <span
            v-if="getTagStyle(tag)"
            class="items-center px-3 py-0.5 mr-2 rounded-full text-base font-medium"
            :class="getTagStyle(tag)"
          >
            {{ tag }}
          </span>
        </div>
        <div class="flex space-x-2">
          <img
            v-for="(integration, integrationIndex) in blog.integrations"
            :key="integrationIndex"
            :src="require(`~/assets/logo/${getIntegrationLogo(integration)}`)"
            class="h-8 w-auto"
          />
        </div>
      </div>
      <h1>{{ blog.title }}</h1>
    </div>
    <div
      class="flex flex-row items-center justify-center text-base text-gray-900 font-semibold tracking-wide uppercase"
    >
      <img
        v-if="blog.author.avatar"
        class="h-10 w-10 rounded-full mr-2"
        :src="blog.author.avatar"
        alt=""
      />{{ blog.author.name }}
      <div class="ml-2 flex space-x-1 text-gray-500">
        <time :datetime="blog.publishedAt">
          {{ blog.formatedPublishedAt }}
        </time>
        <span aria-hidden="true">&middot;</span>
        <span>{{ blog.readingTime }}</span>
      </div>
    </div>
    <div class="flex flex-row">
      <!-- An empty block on the left side for layout -->
      <aside class="w-52 hidden xl:flex" />
      <nuxt-content
        class="w-full px-4 py-6 prose xl:prose-lg prose-indigo mx-auto"
        :document="blog"
      />
      <Toc :content="blog" :scroll-offset="200" />
    </div>
  </div>
</template>

<script lang="ts">
import Toc from "~/components/Toc.vue";
import { startsWith } from "lodash";
import { getTeammateByName } from "~/common/teammate";
import { calcReadingTime } from "~/common/utils";
import {
  PostTag,
  postTagStyle,
  Integration,
  integrationLogo,
} from "~/common/type";

export default {
  components: { Toc },
  layout: "blog",
  async asyncData({ params, $content }: any) {
    const data = await $content("blog", params.slug, {
      deep: true,
    }).fetch();
    const author: any = {
      name: data.author,
    };
    const teammate = getTeammateByName(author.name) as any;
    if (teammate) {
      author.avatar = require(`~/assets/people/${teammate?.name.toLowerCase()}.webp`);
    }

    const blog = {
      ...data,
      author: author,
      formatedPublishedAt: new Date(data.publishedAt).toLocaleString(
        "default",
        {
          year: "numeric",
          month: "short",
          day: "numeric",
        }
      ),
      readingTime: calcReadingTime(data.bodyPlainText),
    };

    return {
      blog,
    };
  },
  head() {
    const blog = (this as any).blog;
    const route = (this as any).$route;
    const link = process.env.hostname + route.fullPath;
    let featureImage = blog.featureImage;
    if (startsWith(featureImage, "/")) {
      featureImage = process.env.hostname + featureImage;
    }

    return {
      title: blog.title,
      meta: [
        {
          hid: "twitter:card",
          name: "twitter:card",
          content: "summary_large_image",
        },
        {
          hid: "title",
          name: "og:title",
          property: "og:title",
          content: blog.title,
        },
        {
          hid: "description",
          name: "og:description",
          property: "og:description",
          content: blog.description,
        },
        {
          hid: "image",
          name: "og:image",
          property: "og:image",
          content: featureImage,
        },
        {
          hid: "type",
          name: "og:type",
          property: "og:type",
          content: "article",
        },
        {
          hid: "url",
          property: "og:url",
          content: link,
        },
      ],
    };
  },
  methods: {
    getTagStyle(tag: PostTag): string {
      return postTagStyle(tag);
    },
    getIntegrationLogo(integration: Integration): string {
      return integrationLogo(integration);
    },
  },
};
</script>

<style scoped>
.nuxt-content table pre {
  white-space: pre-wrap;
  margin: 0;
}
</style>
