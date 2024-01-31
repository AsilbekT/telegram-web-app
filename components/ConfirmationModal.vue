<template>
    <div v-if="visible" class="modal">
        <p>{{ message }}</p>
        <button @click="closeModal">Close</button>
    </div>
</template>
  
<script setup>
import { ref, watchEffect } from 'vue';

const props = defineProps({
    visible: Boolean,
    success: Boolean,
    errorMessage: String
});

const message = ref('');

watchEffect(() => {
    if (props.success) {
        message.value = 'Payment successful!';
    } else {
        message.value = `Payment failed: ${props.errorMessage}`;
    }
});

const closeModal = () => {
    // Logic to close modal
};
</script>
  
<style scoped>
/* Add styles for your modal here */
</style>
  
  