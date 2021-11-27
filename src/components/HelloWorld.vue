<script lang="ts">
import { ref, defineComponent } from "vue";

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

    const getEntries = async () => {
      const response = await fetch(
        "https://raw.githubusercontent.com/narze/awesome-cheab-quotes/main/README.md"
      );

      const readme = await response.text();

      entries.value = readme
        .split("\n")
        .filter((line) => line.startsWith("- "))
        .map((l) => l.slice(2))
        .map((l, idx) => {
          const tagsTmp = l.match(/(\[(\w+(,\s)?)+\])/g);
          const body = l.replace(/(\[(\w+(,\s)?)+\])/g, "").trimEnd();
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
    };
    const randomEntryByTag = (tag: string) => {
      const index = ~~(Math.random() * entriesByTag.value[tag].length);
      result.value = entriesByTag.value[tag][index].body;
      resultTags.value = entriesByTag.value[tag][index].tags;
    };
    return {
      count,
      entries,
      getEntries,
      randomEntry,
      result,
      resultTags,
      randomEntryByTag,
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
    <a
      class="fixed top-10 left-10 text-white"
      href="https://github.com/narze/awesome-cheab-quotes"
      target="_blank"
      >github
    </a>
    <section class="flex flex-col justify-center items-center flex-grow h-full">
      <h1 class="text-5xl textWeight-500 text-white">คำคมเฉียบๆ</h1>
      <p class="mt-8 text-2xl textWeight-400 text-white text-opacity-90">
        {{ result }}
      </p>
      <div class="flex flex-row justify-center gap-1">
        <button
          type="button"
          class="
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
          v-bind:key="idx"
          v-for="(resultTag, idx) in resultTags"
          :title="`ค้นหาคำคม ${resultTag}`"
          @click="randomEntryByTag(resultTag)"
        >
          {{ `#${resultTag}` }}
        </button>
      </div>
      <button
        v-on:click="randomEntry()"
        class="
          btnCustom
          text-lg
          mt-4
          bg-green-400
          px-3
          py-1
          rounded
          border border-green-500
          hover:bg-green-500 hover:border-green-600
        "
      >
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
