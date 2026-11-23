<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import { useBooks } from '@/composables/useBooks'
import BookList from '@/components/BookList.vue'

const { books, loading, error, load } = useBooks()
onMounted(load)

const query = ref('')
const category = ref('')
const sort = ref('title')
const page = ref(1)
const PER_PAGE = 8

const categories = computed(() => [...new Set(books.value.map((b) => b.category))])

const filtered = computed(() =>
  books.value
    .filter((b) => !category.value || b.category === category.value)
    .filter((b) =>
      (b.title + ' ' + b.author).toLowerCase().includes(query.value.trim().toLowerCase()),
    )
    .sort((a, b) => (a[sort.value] > b[sort.value] ? 1 : -1)),
)

const pageCount = computed(() => Math.max(1, Math.ceil(filtered.value.length / PER_PAGE)))

const visible = computed(() =>
  filtered.value.slice((page.value - 1) * PER_PAGE, page.value * PER_PAGE),
)

// retour à la page 1 quand un filtre change
watch([query, category, sort], () => (page.value = 1))
</script>

<template>
  <h1>Catalogue</h1>

  <p v-if="loading" role="status">Chargement…</p>

  <div v-else-if="error" role="alert">
    <p>{{ error }}</p>
    <button type="button" @click="load">Réessayer</button>
  </div>

  <template v-else>
    <input v-model="query" type="search" aria-label="Rechercher" placeholder="Rechercher…" />

    <select v-model="category" aria-label="Catégorie">
      <option value="">Toutes</option>
      <option v-for="c in categories" :key="c" :value="c">{{ c }}</option>
    </select>

    <select v-model="sort" aria-label="Trier par">
      <option value="title">Titre</option>
      <option value="year">Année</option>
    </select>

    <p v-if="!filtered.length">Aucun résultat.</p>
    <BookList v-else :books="visible" />

    <nav aria-label="Pagination">
      <button type="button" :disabled="page === 1" @click="page--">Précédent</button>
      <span>Page {{ page }} / {{ pageCount }}</span>
      <button type="button" :disabled="page === pageCount" @click="page++">Suivant</button>
    </nav>
  </template>
</template>
