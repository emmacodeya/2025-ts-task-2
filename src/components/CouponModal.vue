<script setup lang="ts">
import { Modal } from 'bootstrap'
import { onMounted, onUnmounted, ref, useTemplateRef, watch } from 'vue'
import type { CouponData } from '@/types/coupon'
import { formatDate } from '@/utils/date'


// 型別定義
interface CouponModalProps {
  coupon: CouponData
  isLoading: boolean
}

const props = defineProps<CouponModalProps>()


// 初始資料
const getInitialCouponData = (): CouponData => ({
  id: '',
  title: '',
  is_enabled: 0,
  percent: 0,
  due_date: Math.floor(Date.now() / 1000),
  code: '',
})

const tempCoupon = ref<CouponData>(getInitialCouponData())


// 編輯 / 新增
watch(
  () => props.coupon,
  (newCoupon) => {
    if (newCoupon?.id) {
      tempCoupon.value = { ...newCoupon }
    } else {
      tempCoupon.value = getInitialCouponData()
    }
  },
  { immediate: true, deep: true },
)


const modalRef = useTemplateRef<HTMLElement>('modalRef')
let modal: Modal | null = null

let onSaveCoupon: ((couponData: CouponData) => Promise<void>) | null = null

onMounted(() => {
  if (modalRef.value) {
    modal = new Modal(modalRef.value)
  }
})

onUnmounted(() => {
  modal?.dispose()
})


// 對外開啟 / 關閉 Modal

const openModal = (onSave: (couponData: CouponData) => Promise<void>): void => {
  if (!modal) return

  modal.show()
  onSaveCoupon = async (couponData: CouponData) => {
    try {
      await onSave(couponData)
      closeModal()
    } catch {
    }
  }
}

const closeModal = (): void => {
  modal?.hide()
}


// 儲存

const handleSave = (): void => {
  if (!onSaveCoupon) return
  onSaveCoupon(tempCoupon.value)
}


// 日期轉換（date → timestamp）

const handleDateChange = (e: Event): void => {
  const target = e.target as HTMLInputElement
  tempCoupon.value.due_date = Math.floor(
    new Date(target.value).getTime() / 1000,
  )
}

// 對外暴露方法
defineExpose({
  openModal,
  closeModal,
})
</script>

<template>
  <div
    ref="modalRef"
    class="modal fade"
    id="couponModal"
    tabindex="-1"
    aria-labelledby="couponModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog modal-dialog-centered modal-lg">
      <div class="modal-content rounded-lg">
        <div class="modal-header">
          <h5 class="modal-title" id="couponModalLabel">
            {{ tempCoupon.id ? '編輯優惠券' : '新增優惠券' }}
          </h5>
          <button type="button" class="btn-close" @click="closeModal"></button>
        </div>

        <div class="modal-body">
          <form>
            <div class="mb-3">
              <label for="couponTitle" class="form-label">優惠券標題</label>
              <input
                v-model="tempCoupon.title"
                type="text"
                class="form-control rounded-lg"
                id="couponTitle"
                placeholder="例如：新會員專屬折扣"
              />
            </div>

            <div class="mb-3">
              <label for="couponCode" class="form-label">優惠碼</label>
              <input
                v-model="tempCoupon.code"
                type="text"
                class="form-control rounded-lg"
                id="couponCode"
                placeholder="例如：NEW2024"
              />
            </div>

            <div class="mb-3">
              <label for="couponDiscount" class="form-label">折扣</label>
              <input
                v-model.number="tempCoupon.percent"
                type="number"
                min="0"
                max="100"
                class="form-control rounded-lg"
                id="couponDiscount"
                placeholder="例如：90 代表 9 折"
              />
            </div>

            <div class="mb-3">
              <label for="couponEndDate" class="form-label">截止日</label>
              <input
                :value="formatDate(tempCoupon.due_date)"
                @change="handleDateChange"
                type="date"
                class="form-control rounded-lg"
                id="couponEndDate"
              />
            </div>

            <div class="mb-3 d-flex align-items-center">
              <label class="form-label me-3 mb-0">啟用</label>
              <div class="form-check form-switch">
                <input
                  v-model="tempCoupon.is_enabled"
                  class="form-check-input"
                  type="checkbox"
                  :true-value="1"
                  :false-value="0"
                />
              </div>
            </div>
          </form>
        </div>

        <div class="modal-footer">
          <button type="button" class="btn btn-outline-secondary" @click="closeModal">
            取消
          </button>
          <button
            @click="handleSave"
            :disabled="props.isLoading"
            type="button"
            class="btn btn-dark"
          >
            儲存
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped></style>
