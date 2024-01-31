<template>
    <div>
        <PaymentForm />
        <ConfirmationModal :visible="showModal" :success="paymentSuccess" :errorMessage="errorMessage" />
    </div>
</template>
  
<script setup>
import PaymentForm from '~/components/PaymentForm.vue';
import ConfirmationModal from '~/components/ConfirmationModal.vue';

// You will have state here to manage showing the modal, payment success status, etc.
</script>
  