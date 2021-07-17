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
    const entry = ref<Entry>({} as Entry)

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

      entry.value = entries.value[~~(Math.random() * entries.value.length)]
    }

    return { count, entries, getEntries, entry }
  },
  mounted() {
    this.getEntries()
  },
})
</script>

<template>
  <h1 class="text-6xl">คำคมเฉียบๆ</h1>
  <!-- <p>{{ entries[0] }}</p> -->
  <p>{{ entry }}</p>
</template>

<style scoped>
a {
  color: #42b983;
}

label {
  margin: 0 0.5em;
  font-weight: bold;
}

code {
  background-color: #eee;
  padding: 2px 4px;
  border-radius: 4px;
  color: #304455;
}
</style>
