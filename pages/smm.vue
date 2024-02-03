<template>
    <div class="container">
        <form @submit.prevent="sendMessages" class="message-form">
            <div class="form-group">
                <label for="message">Message:</label>
                <textarea id="message" v-model="messageText" placeholder="Enter your message here..." required></textarea>
            </div>
            <div class="form-group">
                <label for="picture">Picture (optional):</label>
                <input type="file" id="picture" @change="handleFileUpload($event, 'picture')" accept="image/*" />
            </div>
            <div class="form-group">
                <label for="video">Video (optional):</label>
                <input type="file" id="video" @change="handleFileUpload($event, 'video')" accept="video/*" />
            </div>
            <button type="submit" :disabled="sending" class="send-button">
                Send Messages
                <span v-if="sending" class="loader"></span>
            </button>
        </form>
        <table v-if="users.length > 0">
            <thead>
                <tr>
                    <th>User ID</th>
                    <th>Status</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="user in users" :key="user.uuid">
                    <td>{{ user.uuid }}</td>
                    <td>{{ user.firstname }} {{ user.lastname }}</td>
                    <td>
                        <span :class="`status-badge ${user.status}`">{{ user.status }}</span>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</template>
  
<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const users = ref([]);
const sending = ref(false);
const messageText = ref('');
const mediaFile = ref(null);
const mediaType = ref(null);
const fileCache = {
    photo: {},
    video: {}
};

function cacheFileId(type, userId, fileId) {
    if (type === 'photo' || type === 'video') {
        fileCache[type][userId] = fileId;
    }
}

function getCachedFileId(type, userId) {
    if (fileCache[type] && fileCache[type][userId]) {
        return fileCache[type][userId];
    }
    return null;
}

onMounted(() => {
    fetchUsers();
});

const fetchUsers = async () => {
    try {
        const response = await axios.get('https://telegrambot.pandatv.uz/api/bot-users/list/?per-page=1000000');
        users.value = response.data.data.map(user => ({
            ...user,
            status: 'pending'
        }));
    } catch (error) {
        console.error('Failed to fetch users:', error);
    }
}

const handleFileUpload = (event, type) => {
    mediaFile.value = event.target.files[0];
    mediaType.value = type;
};

const sendMessages = async () => {
    sending.value = true;

    for (const user of users.value) {
        try {
            let formData = new FormData();
            formData.append('chat_id', user.uuid);
            const cachedFileId = getCachedFileId(mediaType.value, user.uuid);

            if (mediaFile.value && !cachedFileId) {
                formData.append(mediaType.value === 'picture' ? 'photo' : 'video', mediaFile.value);
                formData.append('caption', messageText.value);
            } else if (cachedFileId) {
                formData.append(mediaType.value === 'picture' ? 'photo' : 'video', cachedFileId);
                formData.append('caption', messageText.value);
            } else {
                formData.append('text', messageText.value);
            }

            const config = {
                headers: {
                    'Content-Type': 'multipart/form-data'
                }
            };

            let apiMethod = mediaType.value === 'picture' ? 'sendPhoto' : mediaType.value === 'video' ? 'sendVideo' : 'sendMessage';
            const response = await axios.post(`https://api.telegram.org/bot6449151677:AAGQcfg2HtgrvyIj77Z9iNCh3rsV8ZECNus/${apiMethod}`, formData, config);

            // Cache the file ID if a new file was sent
            if (mediaFile.value && response.data.ok) {
                const fileId = mediaType.value === 'picture'
                    ? response.data.result.photo[response.data.result.photo.length - 1].file_id
                    : response.data.result.video.file_id;
                cacheFileId(mediaType.value, user.uuid, fileId);
            }

            user.status = 'success';
        } catch (error) {
            user.status = 'failed';
            console.error(`Failed to send message to user ${user.uuid}:`, error);
        }
    }

    resetForm();
};

const resetForm = () => {
    sending.value = false;
    messageText.value = '';
    mediaFile.value = null;
    mediaType.value = null;
}
</script>


<style scoped>
.container {
    max-width: 800px;
    /* Slightly increased width for better layout */
    margin: 20px auto;
    /* Center the container and add vertical spacing */
    padding: 20px;
    background: #ffffff;
    /* Clean white background */
    border-radius: 8px;
    /* Rounded corners */
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    /* Subtle shadow for depth */
    font-family: 'Open Sans', sans-serif;
    /* Modern, readable font */
}

.form-group {
    margin-bottom: 20px;
}

label {
    display: block;
    margin-bottom: 0.5rem;
    color: #333;
    /* Dark grey color for text */
    font-weight: 600;
}

textarea,
input[type="file"] {
    width: 100%;
    padding: 10px;
    border-radius: 5px;
    border: 1px solid #ddd;
    /* Subtle border */
    font-size: 16px;
    box-sizing: border-box;
    /* Prevents padding from increasing width */
}

textarea {
    height: 120px;
    /* Slightly taller for more text */
    resize: vertical;
    /* Allow vertical resizing only */
}

input[type="file"] {
    background: #fafafa;
    /* Slightly off-white background */
    border: 1px dashed #ccc;
    /* Dashed border for the input file */
    padding: 15px;
    /* Larger padding for better click area */
    text-align: center;
    /* Center the text */
    cursor: pointer;
    /* Indicate it's clickable */
}

input[type="file"]::file-selector-button {
    display: none;
    /* Hide the default button */
}

.send-button {
    width: 100%;
    padding: 10px 15px;
    background-color: #4caf50;
    /* Green color for success actions */
    color: white;
    border: none;
    border-radius: 5px;
    margin-top: 20px;
    cursor: pointer;
    position: relative;
    font-size: 16px;
    transition: background-color 0.3s;
    /* Smooth transition for hover effect */
}

.send-button:hover {
    background-color: #43a047;
    /* Slightly darker green on hover */
}

.send-button:disabled {
    background-color: #cccccc;
    /* Greyed out when disabled */
    cursor: not-allowed;
}

.loader {
    border: 2px solid #f3f3f3;
    border-radius: 50%;
    border-top: 2px solid #3498db;
    width: 14px;
    height: 14px;
    -webkit-animation: spin 2s linear infinite;
    animation: spin 2s linear infinite;
    position: absolute;
    right: 15px;
    top: 11px;
}

@keyframes spin {
    0% {
        transform: rotate(0deg);
    }

    100% {
        transform: rotate(360deg);
    }
}

table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    /* Consistent shadow with the container */
}

th,
td {
    border: 1px solid #ddd;
    padding: 12px;
    /* Slightly more padding for better spacing */
    text-align: left;
}

tr:nth-child(even) {
    background-color: #f9f9f9;
    /* Very light grey for every other row */
}

tr:hover {
    background-color: #f1f1f1;
    /* Slightly darker on hover for interactivity */
}

th {
    background-color: #4caf50;
    /* Same green as buttons for consistency */
    color: white;
}

.status-badge {
    display: inline-block;
    padding: 5px 10px;
    border-radius: 15px;
    color: white;
    text-transform: capitalize;
    font-size: 12px;
    cursor: default;
    /* Default cursor for non-interactive elements */
}

.status-badge.pending {
    background-color: #ffc107;
    /* Amber for pending status */
}

.status-badge.success {
    background-color: #28a745;
    /* Green for success status */
}

.status-badge.failed {
    background-color: #dc3545;
    /* Red for failed status */
}
</style>