# Uge 18 — Vue.js Komponenter & Routing

Denne uge bygger vi rigtige Single Page Applications (SPA) med Vue.js. Vi lærer om komponent-arkitektur (den grundlæggende byggesten i moderne frontend), Vue Router til navigation uden sideopdateringer, og Vue CLI til professionel projektopsætning.

---

## Komponent-baseret Arkitektur

### Hvad er en komponent?

En komponent er en selvstændig, genbrugelig del af UI der indkapsler sin egen HTML, CSS og JavaScript.

**Tænk på det som LEGO:** Du bygger din app af komponenter — ligesom du bygger af LEGO-klodser. Hver klods er genbrugelig og uafhængig.

```
App
├── NavBar
├── ProductList
│   ├── ProductCard
│   ├── ProductCard
│   └── ProductCard
└── Footer
```

### Single File Component (.vue fil)

En Vue Single File Component har tre sektioner:

```vue
<!-- ProductCard.vue -->
<template>
  <!-- HTML for komponenten -->
  <div class="product-card">
    <img :src="product.imageUrl" :alt="product.name" />
    <h3>{{ product.name }}</h3>
    <p class="price">{{ product.price }} kr.</p>
    <button @click="addToCart">Tilføj til kurv</button>
  </div>
</template>

<script>
export default {
  name: 'ProductCard',

  // Props: data der sendes IND i komponenten fra forælderen
  props: {
    product: {
      type: Object,
      required: true
    }
  },

  // Emits: events der sendes UD til forælderen
  emits: ['add-to-cart'],

  methods: {
    addToCart() {
      this.$emit('add-to-cart', this.product);
    }
  }
}
</script>

<style scoped>
/* scoped = CSS gælder KUN denne komponent */
.product-card {
  border: 1px solid #ddd;
  padding: 16px;
  border-radius: 8px;
}
.price { color: #2ecc71; font-weight: bold; }
</style>
```

---

## Props — data ned i komponenten

**Props** er den måde en forælder-komponent sender data til en barn-komponent.

```vue
<!-- Forælder: ProductList.vue -->
<template>
  <div>
    <ProductCard
      v-for="product in products"
      :key="product.id"
      :product="product"
      @add-to-cart="handleAddToCart"
    />
  </div>
</template>

<script>
import ProductCard from './ProductCard.vue';

export default {
  components: { ProductCard },
  data() {
    return {
      products: [
        { id: 1, name: "Bog", price: 199, imageUrl: "..." },
        { id: 2, name: "Pen", price: 29, imageUrl: "..." }
      ]
    };
  },
  methods: {
    handleAddToCart(product) {
      console.log("Tilføj til kurv:", product.name);
    }
  }
}
</script>
```

**Dataflow:**
```
Forælder → Props → Barn
Forælder ← Events (emit) ← Barn
```

Props er **read-only** i barnet — barnet må ikke ændre props direkte!

---

## Emits — events op til forælderen

Når en barn-komponent skal kommunikere noget til sin forælder, bruges `$emit`:

```vue
<!-- Barn: SearchBar.vue -->
<template>
  <input
    v-model="searchText"
    @input="$emit('search', searchText)"
    placeholder="Søg..."
  />
</template>

<script>
export default {
  emits: ['search'],
  data() {
    return { searchText: '' };
  }
}
</script>
```

```vue
<!-- Forælder: lytter på 'search' eventet -->
<SearchBar @search="handleSearch" />
```

---

## Vue Router

Vue Router giver Single Page Applications navigation uden at siden genindlæses.

### Installation

```bash
npm install vue-router
```

### Opsætning

```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router';
import HomeView from '../views/HomeView.vue';
import ProductsView from '../views/ProductsView.vue';
import ProductDetailView from '../views/ProductDetailView.vue';

const routes = [
  { path: '/', component: HomeView },
  { path: '/products', component: ProductsView },
  { path: '/products/:id', component: ProductDetailView }, // :id = parameter
  { path: '/:pathMatch(.*)*', component: NotFoundView }   // 404 side
];

const router = createRouter({
  history: createWebHistory(),
  routes
});

export default router;
```

```javascript
// main.js
import { createApp } from 'vue';
import App from './App.vue';
import router from './router';

createApp(App).use(router).mount('#app');
```

### Navigation i templates

```vue
<template>
  <nav>
    <!-- RouterLink = <a> tag der ikke genindlæser siden -->
    <RouterLink to="/">Forside</RouterLink>
    <RouterLink to="/products">Produkter</RouterLink>
  </nav>

  <!-- RouterView viser den aktuelle side-komponent -->
  <RouterView />
</template>
```

### Route Parameters

```vue
<!-- ProductDetailView.vue -->
<template>
  <div v-if="product">
    <h1>{{ product.name }}</h1>
  </div>
</template>

<script>
export default {
  data() {
    return { product: null };
  },
  async created() {
    // Hent :id parameteret fra URL (fx /products/42)
    const id = this.$route.params.id;
    const response = await axios.get(`/api/products/${id}`);
    this.product = response.data;
  }
}
</script>
```

### Programmatisk navigation

```javascript
// Naviger fra kode (ikke fra et link)
this.$router.push('/products');
this.$router.push({ path: '/products/' + id });
this.$router.back(); // Gå tilbage
```

---

## Vue CLI — Professionelt Projektopsætning

Vue CLI (nu kaldet Vite + Vue) giver en komplet projektstruktur med:
- Hot Module Replacement (HMR) — siden opdateres automatisk når du gemmer
- ES modules / import
- Single File Components (.vue filer)
- Build system (minification, bundling)

### Opret nyt projekt

```bash
# Med Vite (moderne, anbefalet)
npm create vite@latest mit-projekt -- --template vue
cd mit-projekt
npm install
npm run dev
```

### Projektstruktur

```
mit-projekt/
├── src/
│   ├── main.js              ← Appens indgangspunkt
│   ├── App.vue              ← Rod-komponent
│   ├── components/          ← Genbrugelige komponenter
│   │   ├── NavBar.vue
│   │   └── ProductCard.vue
│   ├── views/               ← Side-komponenter (én per route)
│   │   ├── HomeView.vue
│   │   └── ProductsView.vue
│   └── router/
│       └── index.js         ← Vue Router konfiguration
├── public/                  ← Statiske filer
├── index.html
├── package.json
└── vite.config.js
```

---

## Composition API (moderne Vue)

Det vi har lært er Options API. Vue 3 har også Composition API med `setup()`:

```vue
<script setup>
// Composition API (Vue 3) — mere fleksibelt
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';

const products = ref([]);
const searchText = ref('');

const filteredProducts = computed(() =>
  products.value.filter(p =>
    p.name.toLowerCase().includes(searchText.value.toLowerCase())
  )
);

onMounted(async () => {
  const response = await axios.get('/api/products');
  products.value = response.data;
});
</script>

<template>
  <input v-model="searchText" placeholder="Søg..." />
  <div v-for="product in filteredProducts" :key="product.id">
    {{ product.name }}
  </div>
</template>
```

**Fordele ved Composition API:**
- Logik kan genbruges på tværs af komponenter (composables)
- Bedre TypeScript support
- Nemmere at organisere kompleks logik

---

## Komplet Eksempel — Mini-app

```vue
<!-- App.vue — Todo liste med komponenter og router -->
<template>
  <nav>
    <RouterLink to="/">Hjem</RouterLink>
    <RouterLink to="/todos">Todos</RouterLink>
  </nav>
  <RouterView />
</template>
```

```vue
<!-- views/TodosView.vue -->
<template>
  <h1>Mine todos</h1>
  <AddTodoForm @add="addTodo" />
  <TodoList :todos="todos" @delete="deleteTodo" />
</template>

<script>
import AddTodoForm from '../components/AddTodoForm.vue';
import TodoList from '../components/TodoList.vue';

export default {
  components: { AddTodoForm, TodoList },
  data() {
    return { todos: [] };
  },
  methods: {
    addTodo(text) {
      this.todos.push({ id: Date.now(), text, done: false });
    },
    deleteTodo(id) {
      this.todos = this.todos.filter(t => t.id !== id);
    }
  }
}
</script>
```

---

## Opsummering

| Begreb / Koncept         | Beskrivelse                                                                   |
|--------------------------|-------------------------------------------------------------------------------|
| Komponent                | Selvstændig, genbrugelig del af UI med HTML, CSS og JavaScript samlet         |
| Single File Component    | `.vue`-fil med tre sektioner: `<template>`, `<script>`, `<style>`             |
| Props                    | Data der sendes **ned** fra forælder-komponent til barn-komponent             |
| Emits / `$emit`          | Events der sendes **op** fra barn til forælder for at kommunikere ændringer   |
| `<style scoped>`         | CSS der kun gælder den aktuelle komponent — ingen utilsigtede sideeffekter    |
| Vue Router               | Håndterer navigation i en SPA uden at siden genindlæses                       |
| `<RouterLink>`           | Erstatning for `<a>` der navigerer uden fuld sideopdatering                   |
| `<RouterView>`           | Placeholder der viser den aktuelle side-komponent for den aktive route        |
| Route Parameters         | Del af URL'en (fx `:id`) der kan aflæses med `this.$route.params.id`          |
| `$router.push()`         | Programmatisk navigation fra JavaScript-kode                                  |
| Vite                     | Moderne build-tool til Vue-projekter med hot module replacement (HMR)         |
| Options API              | Den klassiske måde at skrive Vue-komponenter på (data, methods, computed)     |
| Composition API          | Moderne alternativ med `setup()`, `ref()` og `computed()` — mere fleksibelt  |

---

## Opgaver til selvstudium

1. Opret et nyt Vue-projekt med Vite og inspicer projektstrukturen (`src/`, `App.vue`, `main.js`)
2. Byg en `ProductCard`-komponent der modtager `product`-props (navn, pris, beskrivelse) og viser dem med Bootstrap-styling
3. Brug `$emit` til at sende et `add-to-cart`-event fra `ProductCard` op til forælderen — vis en tæller i forælderen
4. Opsæt Vue Router med mindst to routes: `/` (forside) og `/products` (produktliste)
5. Tilføj en route med parameter: `/products/:id` — hent og vis det specifikke produkt via Axios
6. Omskriv én af dine komponenter fra Options API til Composition API (`<script setup>`) og sammenlign de to stile
