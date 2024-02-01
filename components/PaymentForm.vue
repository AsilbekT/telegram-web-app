<template>
    <div class="payment-form-container">
        <div class="logo-container">
            <img :src="logoUrl" alt="Logo" class="logo" />
        </div>
        <form @submit.prevent="submitPayment" class="payment-form"
            v-if="paymentState === 'idle' || paymentState === 'submitting'">
            <div class="form-group">
                <input v-model="cardDetails.number" type="text" placeholder="Card Number" class="input-field" v-maska
                    data-maska="#### #### #### ####" autocomplete="cc-number" />
            </div>
            <div class="form-group">
                <input v-model="cardDetails.expiry" type="text" placeholder="Expiry MM/YY" class="input-field" v-maska
                    data-maska="##/##" autocomplete="cc-exp" />
            </div>
            <div class="form-group">
                <button type="submit" class="submit-btn disabled" disabled v-if="paymentState === 'submitting'">
                    <img src="~/assets/loader.svg" alt="">
                    Yuklanmoqda
                </button>
                <button type="submit" class="submit-btn" v-else>
                    To'lash
                </button>

            </div>
        </form>
        <div v-if="paymentState === 'awaitingConfirmation'" class="confirmation-container">
            <p class="confirmation-instruction">Please enter the SMS code sent to your phone.</p>
            <div class="input-group">
                <input v-model="smsCode" type="text" placeholder="SMS Code" class="input-field sms-input" />
                <button @click="confirmPayment" class="submit-btn confirm-btn">
                    <span class="btn-text">Confirm</span>
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
import { defineProps } from 'vue';
import axios from 'axios';
import logoUrl from '@/assets/logo.png';

const props = defineProps({
    userId: String
});

const userId = props.userId; // Assuming the parameter is named 'userId'
const cardDetails = ref({
    number: '',
    expiry: '',
});
const smsCode = ref('');
const paymentState = ref('idle'); // 'idle', 'submitting', 'awaitingConfirmation', 'success', 'failure'
const apiUrl = 'https://pandatvbot.inset.uz/api/payme';
let cardId = null;

const payForSubscription = async () => {
    try {
        const paymentResponse = await axios.post(`${apiUrl}/pay-subscription`, {
            telegram_user_id: userId,
            card_id: cardId,
        });

        if (paymentResponse.data && paymentResponse.data.success) {
            paymentState.value = 'success';
        } else {
            throw new Error('Failed to process subscription payment');
        }
    } catch (error) {
        paymentState.value = 'failure';
        console.error('Subscription payment failed:', error.response?.data?.message || error.message);
    }
};

const submitPayment = async () => {
    paymentState.value = 'submitting';
    const [expMonth, expYear] = cardDetails.value.expiry.split('/');
    try {
        const createTokenResponse = await axios.post(`${apiUrl}/create-token`, {
            telegram_user_id: userId,
            card_number: cardDetails.value.number.replace(/\s/g, ''),
            card_expires: `${expMonth}${expYear}`
        });

        if (createTokenResponse.data && createTokenResponse.data.success) {
            cardId = createTokenResponse.data.data.id;
            await getVerifySMS();
        } else {
            throw new Error('Failed to create card token');
        }
    } catch (error) {
        paymentState.value = 'failure';
        console.error('Payment submission failed:', error.response?.data?.message || error.message);
    }
};

const getVerifySMS = async () => {
    try {
        const verifySmsResponse = await axios.post(`${apiUrl}/verify-sms`, {
            card_id: cardId,
            telegram_user_id: userId
        });

        if (verifySmsResponse.data && verifySmsResponse.data.success) {
            paymentState.value = 'awaitingConfirmation';
        } else {
            throw new Error('Failed to send verification SMS');
        }
    } catch (error) {
        paymentState.value = 'failure';
        console.error('SMS verification request failed:', error.response?.data?.message || error.message);
    }
};

const confirmPayment = async () => {
    try {
        const verifyTokenResponse = await axios.post(`${apiUrl}/verify-token`, {
            card_id: cardId,
            code: smsCode.value,
            telegram_user_id: userId
        });

        if (verifyTokenResponse.data && verifyTokenResponse.data.success) {
            await payForSubscription();
        } else {
            throw new Error('Failed to verify card');
        }
    } catch (error) {
        paymentState.value = 'failure';
        console.error('Payment confirmation failed:', error.response?.data?.message || error.message);
    }
};
</script>


  
<style scoped>
* {
    box-sizing: border-box;
}

.logo-container {
    text-align: center;
    margin-bottom: 20px;
}

.logo {
    max-width: 100%;
    height: auto;
    border-radius: 5px;
}

.payment-form-container {
    width: 100%;
    max-width: 480px;
    max-height: 100vh;
    /* Limit the height to viewport height */
    margin: 20px auto;
    padding: 20px;
    overflow-y: auto;
    /* Allow internal scrolling */
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    border-radius: 10px;
    background: white;
    display: flex;
    flex-direction: column;
    justify-content: center;
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
    padding: 10px 15px;
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
    padding: 10px 15px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s, transform 0.3s ease-in-out;
}

.submit-btn.disabled {
    padding: 3px 15px;
    opacity: .8;
    cursor: not-allowed;
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
    box-sizing: border-box;
}

.confirmation-instruction {
    margin-bottom: 15px;
    color: #333;
}

.input-group {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 10px;
    flex-wrap: wrap;
}

.sms-input {
    flex-grow: 1;
    max-width: calc(100% - 120px);
}

.confirm-btn {
    padding: 10px 15px;
    min-width: 100px;
    box-sizing: border-box;
}

.submit-btn {
    position: relative;
    transition: color 0.4s, background-color 0.4s;
    display: flex;
    align-items: center;
    gap: 10px;
}

.submit-btn img {
    /* width: 32px; */
    height: 29.5px;
}

.submit-btn::after {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: #0056b3;
    z-index: -1;
    transition: transform 0.5s;
    transform: scale(0);
    transform-origin: center;
}

.submit-btn:hover::after {
    transform: scale(2);
    opacity: 0;
}

.submit-btn:active {
    background-color: #004a9f;
}

@media (max-width: 600px) {
    .input-group {
        flex-direction: column;
    }

    .sms-input,
    .confirm-btn {
        max-width: 100%;
    }

    .confirm-btn {
        margin-top: 10px;
    }

    .payment-form-container {
        margin: 10px auto;
        padding: 15px;
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
    color: #34a853;
}

.failure {
    background-color: #fde8e9;
    border-left: 5px solid #ea4335;
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
  