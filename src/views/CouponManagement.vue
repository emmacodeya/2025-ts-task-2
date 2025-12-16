<script setup lang="ts">
import { onMounted, ref, useTemplateRef, watch } from 'vue'
import { apiCreateCoupon, apiDeleteCoupon, apiEditCoupon, apiGetCoupons } from '@/api/coupons'
import DeleteModal from '@/components/DeleteModal.vue'
import CouponModal from '@/components/CouponModal.vue'
import type { CouponData, Pagination } from '@/types/coupon'
import { formatDate } from '@/utils/date'

// 初始表單資料
const getInitialFormData = (): CouponData => ({
  id: '',
  title: '',
  is_enabled: 0,
  percent: 0,
  due_date: Math.floor(Date.now() / 1000),
  code: '',
})

const form = ref<CouponData>(getInitialFormData())

// 分頁 & 列表
const currentPage = ref<string>('1')
const coupons = ref<CouponData[]>([])
const pagination = ref<Pagination>({
  total_pages: 0,
  current_page: 0,
  has_pre: false,
  has_next: false,
  category: '',
})

// 取得優惠券列表
const getCoupons = async (): Promise<void> => {
  try {
    const res = await apiGetCoupons({
      page: currentPage.value,
    })

    coupons.value = res.data.coupons
    pagination.value = res.data.pagination
  } catch {
    alert('取得優惠券列表失敗')
  }
}

onMounted(getCoupons)

watch(currentPage, getCoupons)

// Modal refs
const couponModalRef =
  useTemplateRef<InstanceType<typeof CouponModal>>('couponModalRef')

const deleteModalRef =
  useTemplateRef<InstanceType<typeof DeleteModal>>('deleteModalRef')

// 新增 / 編輯優惠券
const isLoading = ref<boolean>(false)

const openModal = (coupon?: CouponData): void => {
  if (coupon) {
    form.value = { ...coupon }
  } else {
    form.value = getInitialFormData()
  }

  couponModalRef.value?.openModal(async (couponData: CouponData) => {
    isLoading.value = true

    try {
      if (couponData.id) {
        await apiEditCoupon({
          id: couponData.id,
          data: {
            ...couponData,
            is_enabled: Number(couponData.is_enabled),
          },
        })
      } else {
        await apiCreateCoupon({
          ...couponData,
          is_enabled: Number(couponData.is_enabled),
        })
      }
    } catch {
      alert('新增 / 編輯優惠券失敗')
    } finally {
      isLoading.value = false
      getCoupons()
    }
  })
}

// 刪除優惠券
const openDeleteModal = (couponId: string): void => {
  deleteModalRef.value?.openModal(() => deleteCoupon(couponId))
}

const deleteCoupon = async (couponId: string): Promise<void> => {
  try {
    await apiDeleteCoupon(couponId)
  } catch {
    alert('刪除優惠券失敗')
  } finally {
    getCoupons()
  }
}
</script>

<template>
  <div class="d-flex justify-content-end align-items-center pe-3 mb-4">
    <button @click="openModal()" class="btn btn-dark rounded-lg px-4 py-2" type="button">
      <i class="fas fa-plus me-2"></i>新增優惠券
    </button>
  </div>

  <p v-if="coupons.length === 0" class="text-center">目前還沒有優惠券</p>

  <div v-else class="card shadow-sm rounded-lg flex-grow-1">
    <div class="card-body p-4">
      <div class="table-responsive">
        <table class="table table-hover align-middle">
          <thead>
            <tr>
              <th>優惠券標題</th>
              <th>優惠碼</th>
              <th>折扣</th>
              <th>截止日</th>
              <th class="text-center">啟用</th>
              <th></th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="coupon in coupons" :key="coupon.id">
              <td>{{ coupon.title }}</td>
              <td>{{ coupon.code }}</td>
              <td>{{ coupon.percent }}%</td>
              <td>{{ formatDate(coupon.due_date) }}</td>
              <td class="text-center">
                <div class="form-check form-switch d-flex justify-content-center">
                  <input
                    class="form-check-input"
                    type="checkbox"
                    readonly
                    style="pointer-events: none"
                    :checked="Boolean(coupon.is_enabled)"
                  />
                </div>
              </td>
              <td class="text-nowrap">
                <button
                  @click="openModal(coupon)"
                  class="btn btn-sm btn-outline-dark me-2"
                >
                  編輯
                </button>
                <button
                  @click="openDeleteModal(coupon.id)"
                  class="btn btn-sm btn-outline-danger"
                >
                  刪除
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- pagination -->
      <nav class="d-flex justify-content-center mt-4">
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
          >
            <button
              class="page-link"
              :class="{ active: currentPage === pageNum.toString() }"
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

  <!-- 新增 / 編輯 Modal -->
  <CouponModal
    ref="couponModalRef"
    :coupon="form"
    :is-loading="isLoading"
  />

  <!-- 刪除確認 Modal -->
  <DeleteModal
    ref="deleteModalRef"
    title="刪除優惠券"
    content="確定要刪除該優惠券嗎？"
  />
</template>

<style lang="scss" scoped></style>
