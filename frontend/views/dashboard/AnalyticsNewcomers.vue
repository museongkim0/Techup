<script setup>
import { VCard, VCardItem, VCardText, VCardTitle } from 'vuetify/components';
import { useAdminStore } from '../../pinia/useAdminStore';
import { storeToRefs } from 'pinia';

const adminStore = useAdminStore();
const storeRef = storeToRefs(adminStore);

let newComers = ref(storeRef.newComers);

</script>

<template>
  <VCard>
    <VCardItem>
      <VCardTitle>이 달의 신규 회원</VCardTitle>
    </VCardItem>

    <VCardText>
      <div class="d-flex align-center">
        <h5 class="text-h5">
          {{ newComers }}
        </h5>
      </div>
      <!-- 활성화? -->
      <!--
      <VIcon size="24" icon="ri-arrow-up-s-line" color="success" />
      <div class="text-success">
        10%
      </div>
      -->
    </VCardText>
  </VCard>
</template>

<style lang="scss" scoped>
.card-list {
  --v-card-list-gap: 1.5rem;
}
</style>
