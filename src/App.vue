<script setup>
import { ref, computed, onMounted, watch } from 'vue'

const formName = ref('')
const formContact = ref('')

const nameError = computed(() => {
  if (!formName.value.trim()) return 'Введите имя'
  if (formName.value.trim().length < 2) return 'Минимум 2 символа'
  return ''
})

const contactError = computed(() => {
  const contact = formContact.value.trim()

  if (!contact) return 'Введите телефон или Telegram'

  const isPhone = /^\+?[\d\s()-]{10,}$/.test(contact)
  const isTelegram = /^@[a-zA-Z0-9_]{5,32}$/.test(contact)

  if (!isPhone && !isTelegram) {
    return 'Введите корректный телефон или @Telegram'
  }

  return ''
})

const formValid = computed(() => {
  return !nameError.value && !contactError.value
})
const categories = ['Все товары', 'Вазы', 'Чайная посуда', 'Тарелки']

const products = ref([
  {
    id: 1,
    name: 'Ваза «Лунный лотос»',
    category: 'Вазы',
    price: 4290,
    image: '/images/7C8L7cSfrTKXtBJaK3umLrU-hoMg-YZ2AoQ6yBMzS77c2BmvUY_1D1uDqBjl9xl7YX5NR3banrsmJ9CopAfyk7vp1qba0U_-Dunf9j6zlNQZWXf7na2p0xdAEW-6kv-ggKw_fUyimlVPeRkO73ypFVBkwIFZs3h2YtPbfIWVUGU.jpg',
    desc: 'Фарфор с традиционным синим орнаментом'
  },
  {
    id: 2,
    name: 'Чайный набор «Облака»',
    category: 'Чайная посуда',
    price: 6890,
    image: '/images/iKVIlo7UJeNklOO7-IxqnZ9O_DA49PC2DXkOnFFghcLZaP0KCS5sQXTfLg8VnjjSEUadORBFW39ZpKqHLTuPA1KA1qHPFhSHXQXtB6sr7mERUkQeIYSLwwDlK7c42P56B4_7fos6meuuyeKi35evenQlPvJN87b6_lbwP26RQPk.jpg',
    desc: 'Чайник и две пиалы ручной работы'
  },
  {
    id: 3,
    name: 'Тарелка «Синий дракон»',
    category: 'Тарелки',
    price: 2490,
    image: '/images/-io6C04pmmo-cYDoIu5Lw_NHYD7L1S5hff__t66S2F5hIrf3mCsw6YG-bgkZuQYLK1FQms0LMpR_t5l2cb4h9aU9MuaIaZEnYO6q_r705fsFtVEaWcFgcEMrxxQ3T3PBrdYwC2Jy8CWZmhr-1NJOdTUWM0lgn8t0CeLMktpPsP0.jpg',
    desc: 'Декоративная тарелка из тонкого фарфора'
  },
  {
    id: 4,
    name: 'Чашка «Журавли»',
    category: 'Чайная посуда',
    price: 1890,
    image: '/images/xFLvv8uheBfGuVsUQXLqMDSRfyZCVsqqk4EP48Z7vGH7YpLXpWAlsPF4_mDsdHCv-0guTaKQ9Z6EhsD7ZLZEKJ4yfJaXmcdbXKmMSrUwntAz4YrSMUTVmNZyktOlSXAaQ5IyXTmWxjnswkmPNm0UWLazPUY4Az6LChn5Q7xCEFA.jpg',
    desc: 'Изящная чашка с восточным мотивом'
  },
  {
    id: 5,
    name: 'Ваза «Белая хризантема»',
    category: 'Вазы',
    price: 590,
    image: '/images/PxO7ykIdii7utZlum2ntaokW-jRcZv1_XIIbUsUgFlLpSDg28gc3hI81OVloiqQEF1eukEfXHM3FsWlVGav99BAW66kKg84zQoP-Ie4_mWFTND1gfoxVehssCsEiZlDGfgrYaGaKgRKIg78AzbrUaBu0Vd03g0yvPDLC1fP1EDc.jpg',
    desc: 'Элегантная база для интерьера',
  },
  {
    id: 6,
    name: 'Блюдо «Небесный сад»',
    category: 'Тарелки',
    price: 3290,
    image: '/images/27RlDZzcZALk5dDKm6oIyNHD5ApNr1Lw1UmwQuzhqbSWQBQlFGoeniDwdGck-7MFk5ib2_br8kmVi4EkYbIwPCTRxIeOJwBpaAN_EWBJgQ01AQV8O6OXnV4272njJAoFK-0Dukf1PChNUxre16H6UhGUy_WCDn-qCh3xE1whtOc.jpg',
    desc: 'Фарфоровое блюдо с цветочным узором'
  }
])

const selectedCategory = ref('Все товары')
const search = ref('')
const cart = ref([])
const cartOpen = ref(false)
const toast = ref('')



const filteredProducts = computed(() => {
  return products.value.filter(product => {
    const matchesCategory =
      selectedCategory.value === 'Все товары' ||
      product.category === selectedCategory.value

    const matchesSearch = product.name
      .toLowerCase()
      .includes(search.value.toLowerCase().trim())

    return matchesCategory && matchesSearch
  })
})

const cartCount = computed(() =>
  cart.value.reduce((sum, item) => sum + item.quantity, 0)
)

const cartTotal = computed(() =>
  cart.value.reduce((sum, item) => sum + item.price * item.quantity, 0)
)

function formatPrice(price) {
  return price.toLocaleString('ru-RU') + ' ₽'
}

function scrollToSection(id) {
  document.getElementById(id)?.scrollIntoView({
    behavior: 'smooth'
  })
}

function goToTop() {
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

function addToCart(product) {
  const item = cart.value.find(item => item.id === product.id)

  if (item) {
    item.quantity++
  } else {
    cart.value.push({ ...product, quantity: 1 })
  }

  toast.value = 'Товар добавлен в корзину'

  setTimeout(() => {
    toast.value = ''
  }, 2200)
}

function changeQuantity(id, amount) {
  const item = cart.value.find(item => item.id === id)
  if (!item) return

  item.quantity += amount

  if (item.quantity <= 0) {
    removeFromCart(id)
  }
}

function removeFromCart(id) {
  cart.value = cart.value.filter(item => item.id !== id)
}

function sendRequest() {
  
  formSent.value = true
}


function goToContact() {
  cartOpen.value = false

  setTimeout(() => {
    scrollToSection('contact')
  }, 100)
}

onMounted(() => {
  try {
    cart.value = JSON.parse(
      localStorage.getItem('qinghua-cart') || '[]'
    )
  } catch {
    cart.value = []
  }
})

watch(
  cart,
  value => {
    localStorage.setItem('qinghua-cart', JSON.stringify(value))
  },
  { deep: true }
)
</script>

<template>
  <div class="site">
    <div class="top-line">
      ИСКУССТВО КИТАЙСКОГО ФАРФОРА · ДОСТАВКА ПО РОССИИ
    </div>

    <header class="header">
      <a class="logo" href="#" @click.prevent="goToTop">
        <span class="logo-mark">青</span>
        <span>
          <strong>ЦИНХУА</strong>
          <small>QINGHUA PORCELAIN</small>
        </span>
      </a>

      <nav>
        <a href="#catalog">Каталог</a>
        <a href="#about">О фарфоре</a>
        <a href="#contact">Контакты</a>
      </nav>

      <button class="cart-button" @click="cartOpen = true">
        Корзина <span>{{ cartCount }}</span>
      </button>
    </header>

    <main>
      <section class="hero">
        <div class="hero-content">
          <div class="eyebrow">
            ВОСТОЧНАЯ ЭСТЕТИКА В ДЕТАЛЯХ
          </div>

          <h1>
            Красота,<br />
            созданная <i>веками</i>
          </h1>

          <p>
            Коллекция изящного фарфора и керамики,
            вдохновлённая культурой древнего Китая.
          </p>

          <a href="#catalog" class="primary-button">
            Открыть коллекцию <span>↗</span>
          </a>

          <div class="hero-note">
            ФАРФОР · КЕРАМИКА · ТРАДИЦИИ
          </div>
        </div>

        <div class="hero-art">
          <div class="art-circle"></div>

          
          <div class="art-stamp">
            景<br />德<br />镇
          </div>

          <div class="art-caption">
            ИСКУССТВО<br />
            В КАЖДОЙ ДЕТАЛИ
          </div>
        </div>
      </section>

      <section class="benefits">
        <div><span>✳</span> Эстетика Востока</div>
        <div><span>✳</span> Вещи с характером</div>
        <div><span>✳</span> Бережная упаковка</div>
        <div><span>✳</span> Доставка по России</div>
      </section>

      <section id="catalog" class="catalog section">
        <div class="section-heading">
          <div>
            <div class="eyebrow">КОЛЛЕКЦИЯ ЦИНХУА</div>
            <h2>Предметы <i>искусства</i></h2>
          </div>

          <p>
            Фарфор, который превращает<br />
            обычные моменты в особенные.
          </p>
        </div>

        <div class="catalog-tools">
          <div class="categories">
            <button
              v-for="category in categories"
              :key="category"
              :class="{ active: selectedCategory === category }"
              @click="selectedCategory = category"
            >
              {{ category }}
            </button>
          </div>

          <input
            v-model="search"
            class="search"
            placeholder="Поиск товаров..."
          />
        </div>

        <div class="products">
          <article
            v-for="product in filteredProducts"
            :key="product.id"
            class="product"
          >
            <div
              class="product-image"
              :style="{ background: product.color }"
            >
              <img
                class="product-photo"
                :src="product.image"
                :alt="product.name"
                @error="$event.target.style.display = 'none'"
              />

              <span class="product-tag">ЦИНХУА</span>
            </div>

            <div class="product-info">
              <div class="product-category">
                {{ product.category }}
              </div>

              <h3>{{ product.name }}</h3>
              <p>{{ product.desc }}</p>

              <div class="product-bottom">
                <strong>{{ formatPrice(product.price) }}</strong>

                <button
                  class="add-button"
                  @click="addToCart(product)"
                >
                  В корзину +
                </button>
              </div>
            </div>
          </article>
        </div>

        <div v-if="filteredProducts.length === 0" class="empty">
          Ничего не найдено. Попробуй изменить запрос.
        </div>
      </section>

      <section id="about" class="about section">
        <div class="about-symbol">青花</div>

        <div>
          <div class="eyebrow">НАСЛЕДИЕ ВРЕМЕНИ</div>
          <h2>Больше, чем <i>посуда</i></h2>

          <p>
            Цинхуа — это вдохновение китайским сине-белым
            фарфором, в котором простота формы встречается
            с изяществом орнамента. Мы собрали коллекцию
            вещей, способных добавить в повседневность
            немного восточной гармонии.
          </p>
        </div>
      </section>

      <section id="contact" class="contact section">
        <div>
          <div class="eyebrow">МЫ НА СВЯЗИ</div>
          <h2>Найдём вашу <i>жемчужину</i></h2>

          <p>
            Оставьте заявку — поможем подобрать изделие
            из коллекции.
          </p>
        </div>

        <form class="contact-form" @submit.prevent="sendRequest">
  <template v-if="!formSent">
    <input
      v-model="formName"
      type="text"
      placeholder="Ваше имя"
      @blur="nameTouched = true"
    />

    <p v-if="nameTouched && nameError" class="error-text">
      {{ nameError }}
    </p>

    <input
      v-model="formContact"
      type="text"
      placeholder="Телефон или Telegram"
      @blur="contactTouched = true"
    />

    <p v-if="contactTouched && contactError" class="error-text">
      {{ contactError }}
    </p>

    <button
      class="primary-button"
     type="button"
      @click="sendRequest"
           >
        Оставить заявку ↗
     </button>
  </template>

  <div v-else class="success">
    <h3>Спасибо!</h3>
    <p>Мы вам позвоним.</p>
  </div>
</form>
      </section>
    </main>

    <footer>
      <div class="logo footer-logo">
        <span class="logo-mark">青</span>
        <span>
          <strong>ЦИНХУА</strong>
          <small>QINGHUA PORCELAIN</small>
        </span>
      </div>

      <span>© 2026 Цинхуа · Искусство китайского фарфора</span>
    </footer>

    <div v-if="toast" class="toast">
      {{ toast }}
    </div>

    <div
      v-if="cartOpen"
      class="overlay"
      @click.self="cartOpen = false"
    >
      <aside class="cart-panel">
        <div class="cart-header">
          <h2>Ваша корзина</h2>

          <button
            class="close-button"
            @click="cartOpen = false"
          >
            ✕
          </button>
        </div>

        <div v-if="cart.length === 0" class="cart-empty">
          <span>🏺</span>
          <p>Пока здесь пусто</p>

          <button
            class="primary-button"
            @click="cartOpen = false"
          >
            К покупкам
          </button>
        </div>

        <div v-else class="cart-items">
          <div
            v-for="item in cart"
            :key="item.id"
            class="cart-item"
          >
            <div
              class="cart-item-icon"
              :style="{ background: item.color }"
            >
              <img
                class="cart-item-photo"
                :src="item.image"
                :alt="item.name"
                @error="$event.target.style.display = 'none'"
              />
            </div>

            <div class="cart-item-info">
              <strong>{{ item.name }}</strong>
              <span>{{ formatPrice(item.price) }}</span>

              <div class="quantity">
                <button @click="changeQuantity(item.id, -1)">
                  −
                </button>

                {{ item.quantity }}

                <button @click="changeQuantity(item.id, 1)">
                  +
                </button>

                <button
                  class="remove"
                  @click="removeFromCart(item.id)"
                >
                 ❌ 
                </button>
              </div>
            </div>
          </div>

          <div class="cart-total">
            <span>Итого</span>
            <strong>{{ formatPrice(cartTotal) }}</strong>
          </div>

          <button
            class="primary-button checkout"
            @click="goToContact"
          >
            Оформить заявку ↗
          </button>
        </div>
      </aside>
    </div>
  </div>
</template>
<style scoped>
.product-photo {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
  position: absolute;
  inset: 0;
}

.product-image {
  position: relative;
  overflow: hidden;
}

.product-tag {
  position: absolute;
  z-index: 1;
}

.cart-item-icon {
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

.cart-item-photo {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
}
</style>
