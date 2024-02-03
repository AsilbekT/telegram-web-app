<template>
    <div>
        <PaymentForm :userId="userId" :channelId="channelId" />
        <ConfirmationModal :visible="showModal" :success="paymentSuccess" :errorMessage="errorMessage" />
    </div>
</template>
  
<script setup>
import { ref, onMounted } from 'vue';
import { useRoute } from 'vue-router';
import PaymentForm from '~/components/PaymentForm.vue';
import ConfirmationModal from '~/components/ConfirmationModal.vue';

const route = useRoute();
const userId = ref(route.query.userId);
const channelId = ref(route.query.channel_id);

const showModal = ref(false);
const paymentSuccess = ref(false);
const errorMessage = ref('');
</script>
  