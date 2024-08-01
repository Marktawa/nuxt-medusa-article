```vue
// composables/useCart.ts
import { ref } from 'vue'

export const useCart = () => {
  const { $medusa } = useNuxtApp()
  const cartId = ref(localStorage.getItem('cart_id') || null)

  const createCart = async () => {
    try {
      const { cart } = await $medusa.carts.create()
      cartId.value = cart.id
      localStorage.setItem('cart_id', cart.id)
      console.log("The cart ID is " + cart.id)
    } catch (error) {
      console.error('Failed to create cart:', error)
    }
  }

  const addToCart = async (variantId, quantity = 1) => {
    if (!cartId.value) {
      await createCart()
    }
    try {
      const { cart } = await $medusa.carts.lineItems.create(cartId.value, {
        variant_id: variantId,
        quantity: quantity
      })
      console.log('Product added to cart')
      return cart
    } catch (error) {
      console.error('Failed to add product to cart:', error)
    }
  }

  return {
    cartId,
    createCart,
    addToCart
  }
}

// pages/index.vue
<template>
  <div>
    <h1>Welcome to the Medusa Nuxt Store</h1>
    <h2>Products</h2>
    <ul>
      <li v-for="product in products" :key="product.id">
        {{ product.title }} - ${{ formatPrice(product.variants[0].prices[0].amount) }}
        <button @click="handleAddToCart(product.variants[0].id)">
          Add To Cart
        </button>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { onMounted } from 'vue'

const { $medusa } = useNuxtApp()
const { createCart, addToCart } = useCart()

const { data: productData } = await useAsyncData('products', () => 
  $medusa.products.list()
)

const products = computed(() => productData.value?.products || [])

const formatPrice = (amount) => {
  return (amount / 100).toFixed(2)
}

const handleAddToCart = async (variantId) => {
  await addToCart(variantId)
  alert('Added to Cart')
}

onMounted(() => {
  if (!localStorage.getItem('cart_id')) {
    createCart()
  }
})
</script>
```