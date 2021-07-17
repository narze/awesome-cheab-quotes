<script lang="ts">
import { ref, defineComponent } from "vue"

interface Entry {
  body: string
  tags: string[]
  id: number
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
    const count = ref(0)
    const entries = ref<Entry[]>([])
    const result = ref("")
    const resultTags = ref<string[]>([])

    const getEntries = async () => {
      const response = await fetch(
        "https://raw.githubusercontent.com/narze/awesome-cheab-quotes/main/README.md"
      )

      const readme = await response.text()

      entries.value = readme
        .split("\n")
        .filter((line) => line.startsWith("- "))
        .map((l) => l.slice(2))
        .map((l, idx) => {
          const tagsTmp = l.match(/(\[(\w+(,\s)?)+\])/g)
          const body = l.replace(/(\[(\w+(,\s)?)+\])/g, "").trimEnd()
          const tags = tagsTmp![0].slice(1, -1).split(/,\s?/)

          return { tags, body, id: idx + 1 }
        })
    }

    const randomEntry = () => {
      const index = ~~(Math.random() * entries.value.length)

      result.value = entries.value[index].body
      resultTags.value = entries.value[index].tags
    }

    return { count, entries, getEntries, randomEntry, result, resultTags }
  },
  async mounted() {
    await this.getEntries()
    this.randomEntry()
  },
})
</script>

<template>
  <main class="min-h-screen h-screen w-screen m-0 p-20">
    <section class="flex flex-col justify-center items-center flex-grow h-full">
      <h1 class="text-6xl">คำคมเฉียบๆ</h1>

      <p class="mt-8 text-3xl">
        {{ result }}
      </p>
      <p class="text-lg">{{ resultTags.map((t) => `#${t}`).join(", ") }}</p>

      <button
        v-on:click="randomEntry()"
        class="
          text-2xl
          mt-8
          bg-green-400
          px-2
          py-1
          rounded
          border border-green-700
          hover:bg-green-600 hover:border-green-900
        "
      >
        สุ่มใหม่
      </button>
    </section>
  </main>
</template>

<style scoped></style>
