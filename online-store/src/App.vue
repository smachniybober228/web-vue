<script setup lang="ts">
import { computed, ref, watch } from 'vue'

type Product = {
  id: number
  name: string
  price: number
  category: string
  image: string
}

type CartItem = Product & { quantity: number }

const products: Product[] = [
  { id: 1, name: 'Ноутбук Pro 14', price: 89990, category: 'Электроника', image: '💻' },
  { id: 2, name: 'Механическая клавиатура', price: 7500, category: 'Электроника', image: '⌨️' },
  { id: 3, name: 'Кружка разработчика', price: 690, category: 'Аксессуары', image: '☕️' },
  { id: 4, name: 'Худи "Vue Master"', price: 3200, category: 'Одежда', image: '👕' },
  { id: 5, name: 'Мышь беспроводная', price: 2400, category: 'Электроника', image: '🖱' },
  { id: 6, name: 'Стикерпак с логотипом', price: 150, category: 'Аксессуары', image: '🏷' },
  { id: 7, name: 'Монитор 27"', price: 24990, category: 'Электроника', image: '🖥' },
  { id: 8, name: 'Кепка "Frontend"', price: 1100, category: 'Одежда', image: '🧢' },
]

const search = ref('')
const selectedCategory = ref('Все')
const sort = ref('default')
const promo = ref('')
const promoApplied = ref(false)
const cart = ref<CartItem[]>(loadCart())

const categories = computed(() => ['Все', ...new Set(products.map((product) => product.category))])

const filteredProducts = computed(() => {
  const searchText = search.value.trim().toLowerCase()
  const result = products.filter((product) => {
    const matchesSearch = product.name.toLowerCase().includes(searchText)
    const matchesCategory = selectedCategory.value === 'Все' || product.category === selectedCategory.value
    return matchesSearch && matchesCategory
  })

  if (sort.value === 'price-asc') return [...result].sort((a, b) => a.price - b.price)
  if (sort.value === 'price-desc') return [...result].sort((a, b) => b.price - a.price)
  return result
})

const cartCount = computed(() => cart.value.reduce((sum, item) => sum + item.quantity, 0))
const discount = computed(() => (promoApplied.value ? 0.1 : 0))
const total = computed(() => cart.value.reduce((sum, item) => sum + item.price * item.quantity * (1 - discount.value), 0))

watch(cart, (newCart) => {
  localStorage.setItem('shop-cart', JSON.stringify(newCart))
}, { deep: true })

function loadCart(): CartItem[] {
  const savedCart = localStorage.getItem('shop-cart')
  if (!savedCart) return []
  try {
    return JSON.parse(savedCart)
  } catch {
    return []
  }
}

function addToCart(product: Product) {
  const item = cart.value.find((cartItem) => cartItem.id === product.id)
  if (item) item.quantity++
  else cart.value.push({ ...product, quantity: 1 })
}

function removeFromCart(id: number) {
  cart.value = cart.value.filter((item) => item.id !== id)
}

function clearCart() {
  cart.value = []
}

function applyPromo() {
  promoApplied.value = promo.value.trim().toUpperCase() === 'WEB'
}

function formatPrice(price: number) {
  return `${price.toLocaleString('ru-RU')} ₽`
}
</script>

<template>
  <main>
    <h1>Интернет-магазин</h1>

    <section>
      <h2>Каталог</h2>
      <label>Поиск: <input v-model="search" type="search" placeholder="Название товара" /></label>
      <label>Категория:
        <select v-model="selectedCategory">
          <option v-for="category in categories" :key="category" :value="category">{{ category }}</option>
        </select>
      </label>
      <label>Сортировка:
        <select v-model="sort">
          <option value="default">По умолчанию</option>
          <option value="price-asc">Сначала дешёвые</option>
          <option value="price-desc">Сначала дорогие</option>
        </select>
      </label>

      <p v-if="filteredProducts.length === 0">Товары не найдены.</p>
      <ul v-else>
        <li v-for="product in filteredProducts" :key="product.id">
          <span>{{ product.image }} {{ product.name }}</span>
          <span>{{ product.category }} — {{ formatPrice(product.price) }}</span>
          <button type="button" @click="addToCart(product)">Добавить в корзину</button>
        </li>
      </ul>
    </section>
    <section>
      <h2>Корзина ({{ cartCount }})</h2>
      <p v-if="cart.length === 0">Корзина пуста.</p>
      <ul v-else>
        <li v-for="item in cart" :key="item.id">
          {{ item.image }} {{ item.name }} — {{ item.quantity }} шт. —
          {{ formatPrice(item.price * item.quantity * (1 - discount)) }}
          <button type="button" @click="removeFromCart(item.id)">Удалить</button>
        </li>
      </ul>

      <div>
        <input v-model="promo" placeholder="Промокод" />
        <button type="button" @click="applyPromo">Применить</button>
        <span v-if="promoApplied"> Промокод применён: скидка 10%</span>
        <span v-else-if="promo"> Промокод не найден</span>
      </div>
      <p>Итого: {{ formatPrice(total) }}</p>
      <button type="button" :disabled="cart.length === 0" @click="clearCart">Очистить корзину</button>
    </section>
  </main>
</template>