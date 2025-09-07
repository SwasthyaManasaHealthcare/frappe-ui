<template>
  <div class="flex flex-col gap-2 overflow-hidden">
    <div class="m-1">
      <TextInput
        ref="searchInput"
        :placeholder="'Search articles...'"
        v-model="search"
        :debounce="300"
      >
        <template #prefix>
          <FeatherIcon name="search" class="h-4 text-ink-gray-5" />
        </template>
      </TextInput>
    </div>
    <div
      class="flex justify-between items-center text-base text-ink-gray-5 mx-2"
    >
      <div>All articles</div>
      <div class="flex items-center gap-2">
        <div v-if="wikiAvailable" class="text-xs text-green-600 font-medium">
          📚 Wiki Available
        </div>
        <Button variant="ghost" @click="openDocs">
          <FeatherIcon name="arrow-up-right" class="h-4 text-ink-gray-5" />
        </Button>
      </div>
    </div>
    <div class="flex flex-col gap-1.5 overflow-y-auto">
      <div
        v-for="a in parsedArticles"
        :key="a.title"
        class="flex flex-col gap-1.5"
      >
        <div
          class="flex items-center justify-between p-1.5 hover:bg-surface-gray-1 rounded cursor-pointer"
          @click="a.opened = !a.opened"
        >
          <div class="flex items-center gap-2">
            <FeatherIcon
              :name="a.opened ? 'chevron-down' : 'chevron-right'"
              class="h-4 text-ink-gray-5"
            />
            <div class="text-base text-ink-gray-8">{{ a.title }}</div>
          </div>
        </div>
        <div v-show="a.opened" class="flex flex-col gap-1.5 ml-5">
          <div
            v-for="subArticle in a.subArticles"
            :key="subArticle.name"
            class="group flex items-center justify-between gap-2 p-1.5 hover:bg-surface-gray-1 rounded cursor-pointer"
            @click="() => openDoc(subArticle.name)"
          >
            <div class="flex items-center gap-2">
              <FeatherIcon name="file-text" class="h-4 text-ink-gray-5" />
              <div class="text-base text-ink-gray-8">
                {{ subArticle.title }}
              </div>
            </div>
            <FeatherIcon
              name="arrow-up-right"
              class="h-4 hidden group-hover:flex text-ink-gray-5"
            />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import Button from '../../src/components/Button/Button.vue'
import FeatherIcon from '../../src/components/FeatherIcon.vue'
import TextInput from '../../src/components/TextInput/TextInput.vue'
import { ref, computed, onMounted } from 'vue'

const props = defineProps({
  docsLink: {
    type: String,
    default: 'https://docs.frappe.io/healthcare',
  },
})

const searchInput = ref(null)
const search = ref('')
const articles = defineModel()
const wikiAvailable = ref(false)

const parsedArticles = computed(() => {
  if (!search.value) return articles.value

  return articles.value.filter((a) => {
    const filteredSubArticles = a.subArticles.filter((subArticle) => {
      return subArticle.title.toLowerCase().includes(search.value.toLowerCase())
    })

    if (
      a.title.toLowerCase().includes(search.value.toLowerCase()) ||
      filteredSubArticles.length > 0
    ) {
      return {
        ...a,
        subArticles: filteredSubArticles,
      }
    }

    return false
  })
})

async function checkWikiInstalled() {
  try {
    const response = await fetch('/api/method/emr_plus.api.wiki.check_wiki_availability', {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        'X-Frappe-CSRF-Token': window.csrf_token
      }
    })
    const data = await response.json()
    return data.message && data.message.available
  } catch (error) {
    console.error('Wiki availability check failed:', error)
    return false
  }
}

async function openDocs() {
  const wikiInstalled = await checkWikiInstalled()
  if (wikiInstalled) {
    window.open('/emr-plus-docs', '_blank')
  } else {
    window.open(props.docsLink, '_blank')
  }
}

async function openDoc(name) {
  const wikiInstalled = await checkWikiInstalled()
  if (wikiInstalled) {
    window.open(`/emr-plus-docs/${name}`, '_blank')
  } else {
    window.open(`${props.docsLink}/${name}`, '_blank')
  }
}

onMounted(async () => {
  searchInput.value?.el?.focus()
  wikiAvailable.value = await checkWikiInstalled()
})
</script>
