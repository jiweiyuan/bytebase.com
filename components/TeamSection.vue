<!-- This example requires Tailwind CSS v2.0+ -->
<template>
  <div class="bg-white">
    <div class="max-w-7xl mx-auto py-12 px-4 text-center sm:px-6 lg:px-8">
      <div class="space-y-12">
        <div class="space-y-5 sm:mx-auto sm:max-w-xl sm:space-y-4 lg:max-w-5xl">
          <a
            id="team"
            href="#team"
            class="text-3xl font-extrabold tracking-tight sm:text-4xl hover:underline"
            >{{ $t("team.meet-our-crew") }}</a
          >
        </div>
        <div class="flex justify-center">
          <PastCompanyBar />
        </div>
        <div class="mx-auto max-w-5xl">
          <ul role="list" class="space-y-6">
            <li v-for="person in founder" :key="person.name">
              <div class="space-y-4 grid grid-cols-4 gap-6 space-y-0">
                <img
                  class="mx-auto sm:h-40 sm:w-40 rounded-full xl:w-56 xl:h-56"
                  :src="require(`~/assets/people/${person.imageUrl}`)"
                  alt=""
                />
                <div class="col-span-3 text-left pt-4">
                  <div class="space-y-4">
                    <div class="text-lg leading-6 font-medium space-y-1">
                      <h3>{{ person.name }}</h3>
                      <p class="text-indigo-600">{{ $t(person.role) }}</p>
                    </div>
                    <div class="text-lg">
                      <p class="text-gray-500">{{ $t(person.bio) }}</p>
                    </div>
                  </div>
                </div>
              </div>
            </li>
          </ul>
        </div>
        <ul
          role="list"
          class="mx-auto space-y-8 sm:grid grid-cols-2 sm:gap-8 sm:space-y-0 lg:grid-cols-4 lg:max-w-5xl"
        >
          <li v-for="person in shuffleList" :key="person.name">
            <img
              class="mx-auto h-40 w-40 rounded-full xl:w-56 xl:h-56"
              :src="require(`~/assets/people/${person.imageUrl}`)"
              alt=""
            />
            <div class="space-y-2">
              <div class="text-lg leading-6 font-medium space-y-1">
                <template v-if="person.name == 'You'">
                  <nuxt-link
                    :to="localePath('/jobs#jobs')"
                    class="text-2xl font-semibold text-indigo-600 hover:text-indigo-500 hover:underline whitespace-nowrap"
                    >{{ person.role }}🎢</nuxt-link
                  >
                </template>
                <template v-else>
                  <h3>{{ person.name }}</h3>
                  <p class="text-indigo-600">{{ $t(person.role) }}</p>
                </template>
              </div>
            </div>
          </li>
        </ul>
      </div>

      <div class="mt-16 space-y-12">
        <div class="space-y-5 sm:mx-auto sm:max-w-xl sm:space-y-4 lg:max-w-5xl">
          <a
            id="backer"
            href="#backer"
            class="text-3xl font-extrabold tracking-tight sm:text-4xl hover:underline"
            >{{ $t("team.backed-by-the-best") }}</a
          >
        </div>
        <ul
          role="list"
          class="mx-auto space-y-8 sm:grid grid-cols-2 sm:gap-8 sm:space-y-0 lg:grid-cols-2 lg:max-w-5xl"
        >
          <li v-for="person in backer" :key="person.name">
            <img
              class="mx-auto h-40 w-40 rounded-full xl:w-56 xl:h-56"
              :src="require(`~/assets/people/${person.imageUrl}`)"
              alt=""
            />
            <div class="space-y-2">
              <div class="text-lg leading-6 font-medium space-y-1">
                <h3>{{ person.name }}</h3>
                <p class="text-indigo-600">{{ person.role }}</p>
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import {
  defineComponent,
  useContext,
  ref,
  onMounted,
} from "@nuxtjs/composition-api";
import { shuffle } from "lodash";
import teammateList from "~/common/teammate";

const founder = [
  {
    name: "Tianzhou",
    fullname: "Tianzhou Chen",
    role: "team.roles.cofounder-ceo",
    imageUrl: "tianzhou.webp",
    bio: "team.bio.tianzhou",
  },
  {
    name: "Danny",
    fullname: "Danny Xu",
    role: "team.roles.cofounder-cto",
    imageUrl: "danny.webp",
    bio: "team.bio.danny",
  },
];

const backer = [
  {
    name: "Matrix Partners China",
    role: "",
    imageUrl: "matrix.webp",
  },
  {
    name: "Dongxu Huang",
    role: "Co-Founder & CTO - PingCAP",
    imageUrl: "dongxu.webp",
  },
];

export default defineComponent({
  setup() {
    const { app } = useContext();
    const shuffleList = ref<any[]>([]);

    const founderNameList = founder.map((person) => person.name);
    const people = teammateList
      .filter((t) => {
        return !t.emeritus && !founderNameList.includes(t.name);
      })
      .map((t) => {
        return {
          ...t,
          role: app.i18n.t(t.role),
          imageUrl: t.name.toLowerCase() + ".webp",
        };
      });

    onMounted(() => {
      const YOU = {
        name: "You",
        role: app.i18n.t("team.join-us"),
        imageUrl: "wantyou.webp",
      };

      shuffleList.value = shuffle(people).concat(YOU);
    });

    return { shuffleList, founder, backer };
  },
});
</script>
