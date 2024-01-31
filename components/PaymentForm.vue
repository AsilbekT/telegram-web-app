<template>
    <div class="payment-form-container">
        <div class="logo-container">
            <img :src="logoUrl" alt="Logo" class="logo" />
            <!-- Update the src with your logo's path -->
        </div>
        <form @submit.prevent="submitPayment" class="payment-form" autocomplete="on">
            <div class="form-group">
                <input v-model="cardDetails.number" type="text" placeholder="Card Number" class="input-field"
                    autocomplete="cc-number" />
            </div>
            <div class="form-group">
                <input v-model="cardDetails.expiry" type="text" placeholder="Expiry MM/YY" class="input-field"
                    autocomplete="cc-exp" />
            </div>
            <div class="form-group">
                <input v-model="cardDetails.cvv" type="password" placeholder="CVV" class="input-field"
                    autocomplete="cc-csc" />
            </div>
            <div class="form-group">
                <button type="submit" class="submit-btn">Submit Payment</button>
            </div>
        </form>
    </div>
</template>

<script setup>
import { ref } from 'vue';
import logoUrl from '@/assets/logo.png';

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
.logo-container {
    text-align: center;
    margin-bottom: 20px;
}

.logo {
    max-width: 100%;
    /* Ensures the logo is responsive and fits within the container */
    height: auto;
    border-radius: 5px;
    /* Optional: rounds the corners of the logo */
}

.payment-form-container {
    max-width: 480px;
    margin: 20px auto;
    padding: 30px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    border-radius: 10px;
    background: white;
    transition: transform 0.3s ease-in-out, box-shadow 0.3s ease-in-out;
}

.payment-form-container:hover {
    transform: translateY(-3px);
    box-shadow: 0 10px 20px rgba(0, 0, 0, 0.15);
}

.payment-form {
    display: flex;
    flex-direction: column;
}

.form-group {
    margin-bottom: 20px;
}

.input-field {
    width: 100%;
    padding: 12px 15px;
    border: 1px solid #ccc;
    border-radius: 5px;
    box-sizing: border-box;
    transition: border-color 0.3s ease-in-out;
}

.input-field:focus {
    outline: none;
    border-color: #007bff;
}

.submit-btn {
    padding: 12px 20px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s, transform 0.3s ease-in-out;
}

.submit-btn:hover {
    background-color: #0056b3;
    transform: translateY(-2px);
}
</style>
  