<template>
  <div class="tp-checkout-place white-bg">
    <h3 class="tp-checkout-place-title">주문 내역</h3>
    <div class="tp-order-info-list">
      <ul>
        <!-- header -->
        <li class="tp-order-info-list-header">
          <h4>상품명</h4>
          <h4>총합</h4>
        </li>

        <!-- item list -->
        <li v-for="d in displayedItems" :key="d.product.productIdx" class="tp-order-info-list-desc">
          <p class="d-flex align-items-center">
            {{ d.product.name }}
            <span class="mx-2">x {{ d.cartItemQuantity }}</span>

            <small v-if="props.couponInfo.couponIdx && d.product.productIdx === props.couponInfo.productIdx"
              class="text-danger ms-auto">
              – {{ props.couponInfo.couponDiscountRate }}% 할인!
            </small>
          </p>
          <span>
            {{ formatPrice(d.finalPrice * d.cartItemQuantity) }}
          </span>
        </li>

        <!-- subtotal -->
        <li class="tp-order-info-list-subtotal">
          <span>소계</span>
          <span>{{ formatPrice(subtotal) }}</span>
        </li>

        <!-- shipping -->
        <li class="tp-order-info-list-shipping">
          <span>배송비</span>
          <div class="tp-order-info-list-shipping-item d-flex flex-column align-items-end">
            <label v-for="opt in shippingOptions" :key="opt.value">
              <input type="radio" name="shipping" :value="opt.value" v-model="shippingMethodLocal" />
              {{ opt.label }} {{ formatPrice(opt.cost) }}
            </label>
          </div>
        </li>

        <!-- total -->
        <li class="tp-order-info-list-total d-flex justify-content-between">
          <span>총 주문 금액</span>
          <span>{{ formatPrice(subtotal + cartStore.shipCost) }}</span>
        </li>
      </ul>
    </div>

    <div class="tp-checkout-payment">
      <div class="tp-checkout-payment-item">
        <input type="radio" id="credit_card" name="payment" />
        <label @click="handlePayment('credit_card')" for="credit_card" data-bs-toggle="direct-bank-transfer">
          신용카드 결제
        </label>
        <div v-if="payment_name === 'credit_card'" class="tp-checkout-payment-desc direct-bank-transfer">
          <p>
            신용카드 결제는 현재 지원하지 않습니다. 다른 결제 수단을 선택해주세요.
          </p>
        </div>
      </div>

      <div class="tp-checkout-payment-item">
        <input type="radio" id="EASY_PAY" name="payment" />
        <label @click="handlePayment('EASY_PAY')" for="EASY_PAY">
          카카오 페이
        </label>
        <div v-if="payment_name === 'EASY_PAY'" class="tp-checkout-payment-desc cheque-payment">
          <p>
            카카오페이를 통한 결제를 지원합니다.
          </p>
        </div>
      </div>
    </div>

    <div class="tp-checkout-agree mb-4">
      <div class="tp-checkout-option">
        <input id="read_all" type="checkbox" v-model="agreeLocal" />
        <label for="read_all">사이트 이용 약관에 동의합니다.</label>
      </div>
    </div>

    <div class="tp-checkout-btn-wrapper">
      <button type="submit" class="tp-checkout-btn w-100">
        주문하기
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, computed } from 'vue'
import { useCartStore } from '@/pinia/useCartStore'

const props = defineProps({
  couponInfo: {
    type: Object,
    default: () => ({
      couponIdx: null,
      couponDiscountRate: 0,
      productIdx: null,
    })
  },
  shippingMethod: { type: String },
  agree: { type: Boolean, default: false }
})
const emit = defineEmits(['update:shipping', 'update:agree', 'update:payment'])
const cartStore = useCartStore()

const displayedItems = computed(() => {
  return cartStore.cart_products.map(item => {
    const basePrice = item.product.price * (1 - item.product.discount / 100)
    let finalPrice = basePrice

    if (
      props.couponInfo.couponIdx &&
      item.product.productIdx === props.couponInfo.productIdx
    ) {
      finalPrice = basePrice * (1 - props.couponInfo.couponDiscountRate / 100)
    }

    return { ...item, finalPrice }
  })
})

const subtotal = computed(() =>
  displayedItems.value.reduce(
    (sum, d) => sum + d.finalPrice * d.cartItemQuantity,
    0
  )
)

const defaultShipping = computed(() =>
  cartStore.totalPriceQuantity.total < 64000
    ? 'flat_rate'
    : 'free_shipping'
)

const shippingMethodLocal = ref(
  props.shippingMethod ?? defaultShipping.value
)
watch(shippingMethodLocal, val => emit('update:shipping', val))

const shippingOptions = computed(() => {
  const total = cartStore.totalPriceQuantity.total
  const all = [
    { value: 'flat_rate', label: '일반 배송:', cost: 3000 },
    { value: 'local_pickup', label: '매장 수령:', cost: 1500 },
    { value: 'free_shipping', label: '무료 배송', cost: 0 },
  ]

  if (total < 64000) {
    return all.filter(o => o.value !== 'free_shipping')
  }

  return all.filter(o => o.value !== 'flat_rate')
})

const agreeLocal = ref(props.agree)
watch(agreeLocal, v => emit('update:agree', v))
watch(() => props.agree, v => agreeLocal.value = v)

const payment_name = ref('')
function handlePayment(val) {
  payment_name.value = val
  emit('update:payment', val)
}

</script>