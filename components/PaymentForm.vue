<template>
    <div class="payment-form-container">
        <div class="logo-container">
            <img :src="logoUrl" alt="Logo" class="logo" />
        </div>
        <form @submit.prevent="submitPayment" class="payment-form"
            v-if="paymentState === 'idle' || paymentState === 'submitting'">
            <div class="form-group">
                <input v-model="cardDetails.number" type="text" placeholder="Card Number" class="input-field"
                    autocomplete="cc-number" />
            </div>
            <div class="form-group">
                <input v-model="cardDetails.expiry" type="text" placeholder="Expiry MM/YY" class="input-field"
                    autocomplete="cc-exp" />
            </div>
            <div class="form-group">
                <button type="submit" class="submit-btn" :disabled="paymentState === 'submitting'">To'lash</button>
            </div>
        </form>
        <div v-if="paymentState === 'awaitingConfirmation'" class="confirmation-container">
            <p>Please enter the SMS code sent to your phone.</p>
            <div class="input-group">
                <input v-model="smsCode" type="text" placeholder="SMS Code" class="input-field sms-input" />
                <button @click="confirmPayment" class="submit-btn confirm-btn">
                    <span class="btn-text">Confirm Payment</span>
                    <span class="icon"><i class="fas fa-check"></i></span>
                </button>
            </div>
        </div>
        <div v-if="paymentState === 'success'" class="message-container success">
            <div class="message-icon">
                <i class="fas fa-check-circle"></i> <!-- Success icon -->
            </div>
            <div class="message-content">
                <h4>Payment Successful!</h4>
                <p>Your payment was processed successfully.</p>
            </div>
        </div>
        <div v-if="paymentState === 'failure'" class="message-container failure">
            <div class="message-icon">
                <i class="fas fa-times-circle"></i> <!-- Failure icon -->
            </div>
            <div class="message-content">
                <h4>Payment Failed</h4>
                <p>Payment failed. Please try again or contact support.</p>
            </div>
        </div>

    </div>
</template>

<script setup>
import { ref } from 'vue';
import logoUrl from '@/assets/logo.png';

const cardDetails = ref({
    number: '',
    expiry: '',
});
const smsCode = ref('');
const paymentState = ref('idle'); // 'idle', 'submitting', 'awaitingConfirmation', 'success', 'failure'

const submitPayment = async () => {
    paymentState.value = 'submitting';
    try {
        // Simulate API call
        await new Promise(resolve => setTimeout(resolve, 2000));
        paymentState.value = 'awaitingConfirmation';
    } catch (error) {
        paymentState.value = 'failure';
        console.error('Payment submission failed:', error);
    }
};

const confirmPayment = async () => {
    try {
        // Simulate API call with smsCode
        await new Promise(resolve => setTimeout(resolve, 2000));
        paymentState.value = 'success';
    } catch (error) {
        paymentState.value = 'failure';
        console.error('Payment confirmation failed:', error);
    }
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
    width: 80%;
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

.confirmation-container {
    text-align: center;
    padding: 20px;
    background-color: #f9f9f9;
    border-radius: 5px;
    margin-top: 20px;
}

.input-group {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 15px;
}

.sms-input {
    margin-right: 10px;
    flex-grow: 1;
    /* Takes the available space */
}

.confirm-btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
}

.btn-text {
    margin-right: 10px;
}

.icon {
    display: inline-block;
}

.instruction-text {
    font-size: 0.9em;
    color: #666;
    margin-top: 10px;
}

/* Optional: Add animation for the confirmation section to draw attention */
.confirmation-container {
    animation: fadeIn 0.5s ease-in-out;
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(-10px);
    }

    to {
        opacity: 1;
        transform: translateY(0);
    }
}

.message-container {
    padding: 20px;
    margin-top: 20px;
    border-radius: 5px;
    display: flex;
    align-items: center;
    animation: fadeIn 0.5s ease-in-out;
}

.message-icon {
    font-size: 3em;
    /* Adjust size as needed */
    margin-right: 20px;
}

.message-content h4 {
    margin: 0;
    margin-bottom: 5px;
    color: #333;
}

.success {
    background-color: #e9f9e9;
    border-left: 5px solid #34a853;
    /* Adjust color as needed */
    color: #34a853;
}

.failure {
    background-color: #fde8e9;
    border-left: 5px solid #ea4335;
    /* Adjust color as needed */
    color: #ea4335;
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(-10px);
    }

    to {
        opacity: 1;
        transform: translateY(0);
    }
}
</style>
  