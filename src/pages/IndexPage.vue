<script setup>
import QPhoneInput from 'src/components/QPhoneInput.vue'
import { ref } from 'vue'

const countryCode = ref('')
const phoneNumber = ref('')
</script>

<template>
  <q-page padding>
    <QPhoneInput
      v-model:country-code="countryCode"
      v-model:phone-number="phoneNumber"
      outlined
    />
  </q-page>
</template>
