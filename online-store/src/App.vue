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
const promoAttempted = ref(false)
const cart = ref<CartItem[]>(loadCart())

const activeTab = ref<'catalog' | 'cart'>('catalog')

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

watch(promo, () => {
  promoAttempted.value = false
  promoApplied.value = false
})

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

function increaseQuantity(id: number) {
  const item = cart.value.find((cartItem) => cartItem.id === id)
  if (item) item.quantity++
}

function decreaseQuantity(id: number) {
  const item = cart.value.find((cartItem) => cartItem.id === id)
  if (!item) return
  if (item.quantity > 1) item.quantity--
  else removeFromCart(id)
}

function setQuantity(id: number, value: number | string) {
  const item = cart.value.find((cartItem) => cartItem.id === id)
  if (!item) return

  // Приводим к числу и отбрасываем всё, что не целое положительное
  const parsed = Math.floor(Number(value))
  if (!Number.isFinite(parsed) || parsed < 1) {
    item.quantity = 1
  } else {
    item.quantity = parsed
  }
}

function clearCart() {
  cart.value = []
}

function applyPromo() {
  promoAttempted.value = true
  promoApplied.value = promo.value.trim().toUpperCase() === 'WEB'
}

function formatPrice(price: number) {
  return `${price.toLocaleString('ru-RU')} ₽`
}
</script>

<template>
  <main>
    <h1>Интернет-магазин</h1>

    <nav class="tabs">
      <button
        type="button"
        :class="{ active: activeTab === 'catalog' }"
        @click="activeTab = 'catalog'"
      >
        Каталог
      </button>
      <button
        type="button"
        :class="{ active: activeTab === 'cart' }"
        @click="activeTab = 'cart'"
      >
        Корзина ({{ cartCount }})
      </button>
    </nav>

    <section v-show="activeTab === 'catalog'">
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

    <section v-show="activeTab === 'cart'">
      <h2>Корзина ({{ cartCount }})</h2>
      <p v-if="cart.length === 0">Корзина пуста.</p>
      <ul v-else>
        <li v-for="item in cart" :key="item.id">
          <span>{{ item.image }} {{ item.name }}</span>
          <span>{{ formatPrice(item.price * item.quantity * (1 - discount)) }}</span>

          <div class="quantity">
            <button type="button" class="step" @click="decreaseQuantity(item.id)">−</button>
            <input
              type="number"
              min="1"
              :value="item.quantity"
              @input="setQuantity(item.id, ($event.target as HTMLInputElement).value)"
            />
            <button type="button" class="step" @click="increaseQuantity(item.id)">+</button>
          </div>

          <button type="button" @clSWick="removeFromCart(item.id)">Удалить</button>
        </li>
      </ul>

      <div class="promo">
        <input v-model="promo" placeholder="Промокод" />
        <button type="button" @click="applyPromo">Применить</button>
        <span v-if="promoApplied" class="ok"> Промокод применён: скидка 10%</span>
        <span v-else-if="promoAttempted" class="fail"> Промокод не найден</span>
      </div>
      <p class="total">Итого: {{ formatPrice(total) }}</p>
      <button type="button" :disabled="cart.length === 0" @click="clearCart">Очистить корзину</button>
    </section>
  </main>
</template>

<style scoped>
main {
  max-width: 960px;
  margin: 0 auto;
  padding: 24px 16px 48px;
  font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
  color: #1f2933;
}

h1 {
  margin: 0 0 20px;
  font-size: 28px;
}

h2 {
  margin: 0 0 16px;
  font-size: 20px;
}

/* Вкладки */

.tabs {
  display: flex;
  gap: 8px;
  margin-bottom: 20px;
  border-bottom: 2px solid #e5e7eb;
}

.tabs button {
  background: transparent;
  color: #4b5563;
  border: none;
  border-bottom: 2px solid transparent;
  border-radius: 0;
  padding: 10px 16px;
  margin-bottom: -2px;
  font-weight: 500;
  cursor: pointer;
}

.tabs button:hover:not(.active) {
  color: #4f46e5;
  background: transparent;
}

.tabs button.active {
  color: #4f46e5;
  border-bottom-color: #4f46e5;
}

/* Секции */

section {
  background: #fff;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 24px;
}

label {
  display: block;
  margin-bottom: 12px;
  font-size: 14px;
  color: #4b5563;
}

input,
select {
  font: inherit;
  padding: 6px 10px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  background: #fff;
  color: inherit;
  outline: none;
}

input:focus,
select:focus {
  border-color: #4f46e5;
  box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.15);
}

label input,
label select {
  margin-left: 6px;
}

ul {
  list-style: none;
  margin: 16px 0 0;
  padding: 0;
  display: grid;
  gap: 10px;
}

li {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 14px;
  border: 1px solid #e5e7eb;
  border-radius: 10px;
  background: #fafafa;
}

li > span:first-child {
  font-weight: 600;
  flex: 1;
}

li > span:nth-child(2) {
  color: #6b7280;
  font-size: 14px;
}

button {
  font: inherit;
  padding: 6px 12px;
  border: 1px solid transparent;
  border-radius: 8px;
  background: #4f46e5;
  color: #fff;
  cursor: pointer;
  transition: background 0.15s ease;
}

button:hover:not(:disabled):not(.active) {
  background: #4338ca;
}

button:disabled {
  background: #c7d2fe;
  cursor: not-allowed;
}

/* Управление количеством */

.quantity {
  display: flex;
  align-items: center;
  gap: 4px;
}

.quantity input[type="number"] {
  width: 56px;
  text-align: center;
  padding: 6px 4px;
  -moz-appearance: textfield;
  appearance: textfield;
}

/* Прячем стрелки у number input в WebKit — у нас свои кнопки */
.quantity input[type="number"]::-webkit-outer-spin-button,
.quantity input[type="number"]::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

button.step {
  width: 32px;
  height: 32px;
  padding: 0;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  line-height: 1;
  background: #eef2ff;
  color: #4f46e5;
}

button.step:hover:not(:disabled) {
  background: #e0e7ff;
}

/* Промокод */

.promo {
  margin-top: 16px;
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
}

.promo span {
  font-size: 14px;
}

.promo span.ok {
  color: #16a34a;
}

.promo span.fail {
  color: #dc2626;
}

.total {
  margin: 16px 0 12px;
  font-size: 18px;
  font-weight: 600;
}

p {
  color: #6b7280;
}

@media (max-width: 600px) {
  li {
    flex-direction: column;
    align-items: flex-start;
  }

  li > span:first-child {
    flex: initial;
  }
}
</style>