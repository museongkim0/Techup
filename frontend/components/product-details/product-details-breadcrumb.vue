<template>
  <section class="breadcrumb__area breadcrumb__style-2 include-bg pt-50 pb-20">
    <div class="container">
      <div class="row">
        <div class="col-xxl-12">
          <div class="breadcrumb__content p-relative z-index-1">
            <div class="breadcrumb__list has-icon">
              <span class="breadcrumb-icon me-1">
                <svg-dot/>
              </span>
              <span><a href="#">Home</a></span>
              <span><a href="#">{{ product.category }}</a></span>
              <span><a href="#">{{ product.children }}</a></span>
              <span>{{ product.name }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
const props = defineProps({
  product: Object
})
</script>
