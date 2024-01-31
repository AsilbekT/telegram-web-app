<template>
    <div class="payment-form-container">
        <form @submit.prevent="submitPayment" class="payment-form">
            <div class="form-group">
                <input v-model="cardDetails.number" type="text" placeholder="Card Number" />
            </div>
            <div class="form-group">
                <input v-model="cardDetails.expiry" type="text" placeholder="Expiry MM/YY" />
            </div>
            <div class="form-group">
                <input v-model="cardDetails.cvv" type="password" placeholder="CVV" />
            </div>
            <div class="form-group">
                <button type="submit">Submit Payment</button>
            </div>
        </form>
    </div>
</template>
  
<script setup>
import { ref } from 'vue';

const cardDetails = ref({
    number: '',
    expiry: '',
    cvv: ''
});

const submitPayment = () => {
    // Logic to handle payment submission
    console.log(cardDetails.value);
    // You will call your backend API here
};
</script>
  
<style scoped>
.payment-form-container {
    max-width: 480px;
    margin: 0 auto;
    padding: 20px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    border-radius: 8px;
    background: white;
}

.payment-form {
    display: flex;
    flex-direction: column;
}

.form-group {
    margin-bottom: 15px;
}

.form-group input {
    width: 100%;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
}

.form-group button {
    padding: 10px 20px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s;
}

.form-group button:hover {
    background-color: #0056b3;
}
</style>
  