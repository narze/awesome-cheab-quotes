<script lang="ts">
import { defineComponent, reactive, ref } from "vue";

interface Entry {
  body: string;
  tags: string[];
  id: number;
}

interface EntryMap {
  [key: string]: Entry[];
}

export default defineComponent({
  name: "HelloWorld",
  props: {
    msg: {
      type: String,
      required: true,
    },
  },
  setup: () => {
    const count = ref(0);
    const entries = ref<Entry[]>([]);
    const result = ref("");
    const resultTags = ref<string[]>([]);
    const entriesByTag = ref<EntryMap>({});
    const typingState = reactive({
      result: "",
      speed: 50,
      charIndex: 0,
    });

    const getEntries = async () => {
      const response = await fetch(
        "https://raw.githubusercontent.com/narze/awesome-cheab-quotes/main/README.md"
      );

      const readme = await response.text();
      const skipIndex = readme.indexOf("## คำคมเฉียบๆ")

      entries.value = readme
        .slice(skipIndex)
        .split("\n")
        .filter((line) => line.startsWith("- "))
        .map((l) => l.slice(2))
        .map((l, idx) => {
          const tagsTmp = l.match(/(\[([\w+\s](,\s)?)+\])/g);
          const body = l.replace(/(\[([\w+\s](,\s)?)+\])/g, "").trimEnd();
          const tags = tagsTmp![0].slice(1, -1).split(/,\s?/);
          
          return { tags, body, id: idx + 1 };
        });

      entries.value.forEach((entry) => {
        entry.tags.forEach((tag: String) => {
          if (entriesByTag.value[tag.toString()] === undefined) {
            entriesByTag.value[tag.toString()] = [];
          }
          entriesByTag.value[tag.toString()].push(entry);
        });
      });
    };

    const randomEntry = () => {
      const index = ~~(Math.random() * entries.value.length);

      result.value = entries.value[index].body;
      resultTags.value = entries.value[index].tags;

      typingState.result = ""
      typeText();
    };
    const randomEntryByTag = (tag: string) => {
      const index = ~~(Math.random() * entriesByTag.value[tag].length);
      result.value = entriesByTag.value[tag][index].body;
      resultTags.value = entriesByTag.value[tag][index].tags;

      typingState.result = ""
      typeText();
    };
    const typeText = () => {
      if (typingState.charIndex < result.value.length) {
        typingState.result += result.value.charAt(
          typingState.charIndex
        );
        typingState.charIndex += 1;
        setTimeout(typeText, typingState.speed);
      } else {
        typingState.charIndex = 0
      }
    };
    return {
      count,
      entries,
      getEntries,
      randomEntry,
      result,
      resultTags,
      randomEntryByTag,
      typingState
    };
  },
  async mounted() {
    await this.getEntries();
    this.randomEntry();
  },
});
</script>

<template>
  <main class="min-h-screen h-screen w-screen m-0 p-8 bg-gray-800">
    <a class="fixed bottom-10 text-white hover:text-gray-300" href="https://github.com/narze/awesome-cheab-quotes" target="_blank"
      aria-label="github">
      <svg aria-hidden="true" focusable="false" data-prefix="fab" data-icon="github"
        class="svg-inline--fa fa-github fa-w-16 h-8 w-8" role="img" xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 496 512">
        <path fill="currentColor"
          d="M165.9 397.4c0 2-2.3 3.6-5.2 3.6-3.3.3-5.6-1.3-5.6-3.6 0-2 2.3-3.6 5.2-3.6 3-.3 5.6 1.3 5.6 3.6zm-31.1-4.5c-.7 2 1.3 4.3 4.3 4.9 2.6 1 5.6 0 6.2-2s-1.3-4.3-4.3-5.2c-2.6-.7-5.5.3-6.2 2.3zm44.2-1.7c-2.9.7-4.9 2.6-4.6 4.9.3 2 2.9 3.3 5.9 2.6 2.9-.7 4.9-2.6 4.6-4.6-.3-1.9-3-3.2-5.9-2.9zM244.8 8C106.1 8 0 113.3 0 252c0 110.9 69.8 205.8 169.5 239.2 12.8 2.3 17.3-5.6 17.3-12.1 0-6.2-.3-40.4-.3-61.4 0 0-70 15-84.7-29.8 0 0-11.4-29.1-27.8-36.6 0 0-22.9-15.7 1.6-15.4 0 0 24.9 2 38.6 25.8 21.9 38.6 58.6 27.5 72.9 20.9 2.3-16 8.8-27.1 16-33.7-55.9-6.2-112.3-14.3-112.3-110.5 0-27.5 7.6-41.3 23.6-58.9-2.6-6.5-11.1-33.3 2.6-67.9 20.9-6.5 69 27 69 27 20-5.6 41.5-8.5 62.8-8.5s42.8 2.9 62.8 8.5c0 0 48.1-33.6 69-27 13.7 34.7 5.2 61.4 2.6 67.9 16 17.7 25.8 31.5 25.8 58.9 0 96.5-58.9 104.2-114.8 110.5 9.2 7.9 17 22.9 17 46.4 0 33.7-.3 75.4-.3 83.6 0 6.5 4.6 14.4 17.3 12.1C428.2 457.8 496 362.9 496 252 496 113.3 383.5 8 244.8 8zM97.2 352.9c-1.3 1-1 3.3.7 5.2 1.6 1.6 3.9 2.3 5.2 1 1.3-1 1-3.3-.7-5.2-1.6-1.6-3.9-2.3-5.2-1zm-10.8-8.1c-.7 1.3.3 2.9 2.3 3.9 1.6 1 3.6.7 4.3-.7.7-1.3-.3-2.9-2.3-3.9-2-.6-3.6-.3-4.3.7zm32.4 35.6c-1.6 1.3-1 4.3 1.3 6.2 2.3 2.3 5.2 2.6 6.5 1 1.3-1.3.7-4.3-1.3-6.2-2.2-2.3-5.2-2.6-6.5-1zm-11.4-14.7c-1.6 1-1.6 3.6 0 5.9 1.6 2.3 4.3 3.3 5.6 2.3 1.6-1.3 1.6-3.9 0-6.2-1.4-2.3-4-3.3-5.6-2z">
        </path>
      </svg>
    </a>
    <section class="flex flex-col justify-center items-center flex-grow h-full">
      <h1 class="fixed top-10 text-5xl font-thin text-white">คำคมเฉียบ ๆ</h1>
      <p class="mt-8 text-2xl textWeight-400 text-white text-opacity-90">
        {{ typingState.result }}
      </p>
      <div class="flex flex-row justify-center">
        <button type="button" class="
            text-white
            text-opacity-90
            textWeight-400
            text-xs
            mt-4
            px-1
            py-0.5
            rounded
            border border-green-700
            hover:bg-green-500 hover:border-green-500 hover:text-gray-800
          " 
          :class="{'mr-2': resultTags.length > 1}"
          v-bind:key="idx" v-for="(resultTag, idx) in resultTags" :title="`ค้นหาคำคม ${resultTag}`"
          @click="randomEntryByTag(resultTag)">
          {{ `#${resultTag}` }}
        </button>
      </div>
      <button v-on:click="randomEntry()" class="
          btnCustom
          text-lg
          mt-12
          bg-green-400
          px-3
          py-1
          rounded
          border border-green-500
          hover:bg-green-500 hover:border-green-600
        ">
        สุ่มใหม่
      </button>
    </section>
  </main>
</template>

<style>
.textWeight-400 {
  font-weight: 400;
}

.textWeight-500 {
  font-weight: 500;
}
</style>
