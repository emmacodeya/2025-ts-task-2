<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue'
import { RouterLink } from 'vue-router'

import Navbar from '@/components/Navbar.vue'
import Footer from '@/components/Footer.vue'

import { apiGetProducts, apiGetAllProducts } from '@/api/products'
import type { Pagination, Product } from '@/types/product'

// ==========================================
// 狀態
// ==========================================
const currentPage = ref<string>('1')
const selectedCategory = ref<string>('')

// 商品資料
const products = ref<Product[]>([])
const allProducts = ref<Product[]>([])

// 分頁資訊
const pagination = ref<Pagination>({
  total_pages: 0,
  current_page: 0,
  has_pre: false,
  has_next: false,
  category: '',
})

// ==========================================
// API
// ==========================================
const getProducts = async (): Promise<void> => {
  try {
    const res = await apiGetProducts({
      page: currentPage.value,
      category: selectedCategory.value,
    })

    products.value = res.data.products
    pagination.value = res.data.pagination
  } catch {
    alert('取得產品列表失敗')
  }
}

const getAllProducts = async (): Promise<void> => {
  try {
    const res = await apiGetAllProducts()
    allProducts.value = res.data.products
  } catch {
    alert('取得所有產品失敗')
  }
}

// ==========================================
// lifecycle
// ==========================================
onMounted(() => {
  getProducts()
  getAllProducts()
})

// 分頁或分類改變時重新抓資料
watch([currentPage, selectedCategory], () => {
  getProducts()
})

// ==========================================
// 分類列表（去重）
// ==========================================
const categories = computed<string[]>(() => {
  const set = new Set(allProducts.value.map((p) => p.category))
  return Array.from(set)
})
</script>

<template>
  <Navbar />

  <!-- Banner -->
  <div
    class="position-relative d-flex align-items-center justify-content-center"
    style="min-height: 400px"
  >
    <div
      class="position-absolute w-100 h-100"
      style="
        background-image: url(https://images.unsplash.com/photo-1656275537622-7837184a0dcc?q=80&w=1350&auto=format&fit=crop);
        background-position: center;
        background-size: cover;
        opacity: 0.1;
      "
    ></div>
    <h2 class="fw-bold">產品列表</h2>
  </div>

  <div class="container mt-5 mb-7">
    <div class="row">
      <!-- 分類 -->
      <div class="col-md-4">
        <div class="accordion mb-3">
          <div class="card border-0">
            <div
              class="card-header bg-white px-0 py-4"
              data-bs-toggle="collapse"
              data-bs-target="#categoryCollapse"
            >
              <div class="d-flex justify-content-between">
                <h4 class="mb-0">分類</h4>
                <i class="fas fa-chevron-down"></i>
              </div>
            </div>

            <div id="categoryCollapse" class="collapse show">
              <ul class="list-unstyled px-0">
                <li>
                  <button
                    @click="selectedCategory = ''"
                    class="btn btn-link text-muted p-0"
                  >
                    全部
                  </button>
                </li>

                <li v-for="category in categories" :key="category">
                  <button
                    @click="selectedCategory = category"
                    class="btn btn-link text-muted p-0"
                  >
                    {{ category }}
                  </button>
                </li>
              </ul>
            </div>
          </div>
        </div>
      </div>

      <!-- 商品列表 -->
      <div class="col-md-8">
        <div class="row">
          <div v-for="product in products" :key="product.id" class="col-md-6">
            <div class="card border-0 mb-4 position-relative">
              <img
                :src="product.imageUrl"
                class="card-img-top object-fit-cover"
                style="height: 320px"
                alt="product"
              />

              <div class="card-body p-0">
                <h4 class="mt-3 mb-1">
                  <RouterLink :to="`/products/${product.id}`">
                    {{ product.title }}
                  </RouterLink>
                </h4>
                <p class="mb-0">
                  NT$ {{ product.price.toLocaleString() }}
                  <span class="text-muted ms-2">
                    <del>NT$ {{ product.origin_price.toLocaleString() }}</del>
                  </span>
                </p>
              </div>
            </div>
          </div>
        </div>

        <!-- 分頁 -->
        <nav class="d-flex justify-content-center">
          <ul class="pagination">
            <li class="page-item">
              <button
                class="page-link"
                :disabled="!pagination.has_pre"
                @click="currentPage = String(Number(currentPage) - 1)"
              >
                &laquo;
              </button>
            </li>

            <li
              v-for="pageNum in pagination.total_pages"
              :key="pageNum"
              class="page-item"
              :class="{ active: currentPage === pageNum.toString() }"
            >
              <button
                class="page-link"
                @click="currentPage = pageNum.toString()"
              >
                {{ pageNum }}
              </button>
            </li>

            <li class="page-item">
              <button
                class="page-link"
                :disabled="!pagination.has_next"
                @click="currentPage = String(Number(currentPage) + 1)"
              >
                &raquo;
              </button>
            </li>
          </ul>
        </nav>
      </div>
    </div>
  </div>

  <Footer />
</template>

<style lang="scss" scoped></style>
