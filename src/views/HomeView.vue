<template>
  <div class="container">
    <div class="header">
      <h1 class="title">LINE Account Manager</h1>
      <p class="subtitle">Manage multiple LINE accounts</p>
    </div>

    <!-- Add Account Section -->
    <div class="add-section">
      <div class="add-card">
        <div class="add-icon">➕</div>
        <h2>Add New LINE Account</h2>
        <p>Click the button below to login with a LINE account and add it to your list</p>
        <button @click="addNewAccount" class="btn btn-primary btn-large" :disabled="isProcessing">
          <span class="btn-icon">{{ isProcessing ? '⏳' : '🔐' }}</span>
          {{ isProcessing ? 'Processing...' : 'Add Account' }}
        </button>
      </div>
    </div>

    <!-- Accounts List -->
    <div class="accounts-section" v-if="accounts.length > 0">
      <div class="section-header">
        <h2>Account List ({{ accounts.length }})</h2>
        <button @click="clearAll" class="btn btn-danger btn-small">
          Clear All
        </button>
      </div>

      <div class="accounts-grid">
        <div v-for="(account, index) in accounts" :key="account.userId" class="account-card">
          <div class="account-number">{{ index + 1 }}</div>
          <div class="account-content">
            <div class="account-avatar">
              <img v-if="account.pictureUrl" :src="account.pictureUrl" alt="Avatar" />
              <div v-else class="avatar-placeholder">{{ account.displayName?.charAt(0) || '?' }}</div>
            </div>
            <div class="account-details">
              <h3>{{ account.displayName }}</h3>
              <div class="account-id">
                <span class="label">LINE ID:</span>
                <span class="id-value">{{ account.userId }}</span>
                <button @click="copyToClipboard(account.userId)" class="copy-btn" title="Copy ID">
                  📋
                </button>
              </div>
              <div class="account-meta">
                <span class="timestamp">{{ formatDate(account.addedAt) }}</span>
              </div>
            </div>
          </div>
          <button @click="removeAccount(index)" class="remove-btn" title="Remove account">
            ✕
          </button>
        </div>
      </div>
    </div>

    <div v-else class="empty-state">
      <div class="empty-icon">📝</div>
      <p>No accounts added yet</p>
    </div>

    <!-- Toast Notification -->
    <transition name="toast">
      <div v-if="toast.show" class="toast" :class="toast.type">
        {{ toast.message }}
      </div>
    </transition>
  </div>
</template>

<script setup lang="ts">
import liff from '@line/liff';
import { ref, onMounted } from 'vue';

interface LineProfile {
  userId: string;
  displayName: string;
  pictureUrl?: string;
  statusMessage?: string;
  addedAt?: number;
}

const accounts = ref<LineProfile[]>([]);
const isProcessing = ref(false);
const toast = ref({
  show: false,
  message: '',
  type: 'success'
});

const LIFF_ID = '2000635253-WADl2ApA';
const REDIRECT_URL = 'https://test-line-wheat.vercel.app/';
const STORAGE_KEY = 'line_accounts';

// Initialize LIFF
const initLiff = async () => {
  try {
    await liff.init({ liffId: LIFF_ID });
    
    // If user just logged in, save their profile and logout
    if (liff.isLoggedIn()) {
      isProcessing.value = true;
      const profile = await liff.getProfile();
      
      const newAccount: LineProfile = {
        userId: profile.userId,
        displayName: profile.displayName,
        pictureUrl: profile.pictureUrl,
        statusMessage: profile.statusMessage,
        addedAt: Date.now()
      };
      
      // Check if account already exists
      const exists = accounts.value.some(acc => acc.userId === newAccount.userId);
      if (!exists) {
        accounts.value.unshift(newAccount);
        saveAccounts();
        showToast('Account added successfully!', 'success');
      } else {
        showToast('This account is already in the list', 'warning');
      }
      
      // Logout immediately to allow adding another account
      await liff.logout();
      isProcessing.value = false;
    }
  } catch (error) {
    console.error('LIFF init failed:', error);
    showToast('Failed to initialize LIFF', 'error');
    isProcessing.value = false;
  }
};

// Add new account - trigger login
const addNewAccount = async () => {
  try {
    if (!liff.isInClient()) {
      await liff.login({ redirectUri: REDIRECT_URL });
    }
  } catch (error) {
    console.error('Login failed:', error);
    showToast('Login failed', 'error');
  }
};

// Remove account
const removeAccount = (index: number) => {
  accounts.value.splice(index, 1);
  saveAccounts();
  showToast('Account removed', 'success');
};

// Clear all accounts
const clearAll = () => {
  if (confirm('Are you sure you want to remove all accounts?')) {
    accounts.value = [];
    saveAccounts();
    showToast('All accounts cleared', 'success');
  }
};

// Copy to clipboard
const copyToClipboard = async (text: string) => {
  try {
    await navigator.clipboard.writeText(text);
    showToast('ID copied to clipboard', 'success');
  } catch (error) {
    showToast('Failed to copy', 'error');
  }
};

// Save accounts to localStorage
const saveAccounts = () => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(accounts.value));
};

// Load accounts from localStorage
const loadAccounts = () => {
  const saved = localStorage.getItem(STORAGE_KEY);
  if (saved) {
    try {
      accounts.value = JSON.parse(saved);
    } catch (error) {
      console.error('Failed to load accounts:', error);
    }
  }
};

// Format date
const formatDate = (timestamp: number | undefined) => {
  if (!timestamp) return '';
  const date = new Date(timestamp);
  return date.toLocaleString('vi-VN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit'
  });
};

// Show toast notification
const showToast = (message: string, type: 'success' | 'error' | 'warning' = 'success') => {
  toast.value = { show: true, message, type };
  setTimeout(() => {
    toast.value.show = false;
  }, 3000);
};

onMounted(() => {
  initLiff();
  loadAccounts();
});
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 2rem;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.header {
  text-align: center;
  color: white;
  margin-bottom: 3rem;
}

.title {
  font-size: 2.5rem;
  font-weight: 700;
  margin: 0 0 0.5rem 0;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
}

.subtitle {
  font-size: 1.1rem;
  opacity: 0.9;
  margin: 0;
}

.add-section {
  max-width: 600px;
  margin: 0 auto 3rem auto;
  display: flex;
  justify-content: center;
}

.add-card {
  background: white;
  border-radius: 20px;
  padding: 3rem;
  text-align: center;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  width: 100%;
  max-width: 500px;
}

.add-icon {
  font-size: 4rem;
  margin-bottom: 1rem;
}

.add-card h2 {
  margin: 0 0 1rem 0;
  color: #333;
}

.add-card p {
  color: #666;
  margin-bottom: 2rem;
  line-height: 1.6;
}

.btn {
  border: none;
  border-radius: 12px;
  padding: 0.75rem 1.5rem;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  flex: 1;
  justify-content: center;
  min-width: 150px;
}

.btn-icon {
  font-size: 1.2rem;
}

.btn-primary {
  background: #06c755;
  color: white;
}

.btn-primary:hover:not(:disabled) {
  background: #05b04d;
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(6, 199, 85, 0.4);
}

.btn-primary:disabled {
  background: #a0a0a0;
  cursor: not-allowed;
  opacity: 0.6;
}

.btn-secondary {
  background: #f0f0f0;
  color: #333;
}

.btn-secondary:hover {
  background: #e0e0e0;
  transform: translateY(-2px);
}

.btn-large {
  padding: 1rem 2rem;
  font-size: 1.1rem;
}

.btn-small {
  padding: 0.5rem 1rem;
  font-size: 0.9rem;
  min-width: auto;
  flex: none;
}

.btn-danger {
  background: #ff4444;
  color: white;
}

.btn-danger:hover {
  background: #cc0000;
}

.accounts-section {
  max-width: 1200px;
  margin: 0 auto;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
  color: white;
}

.section-header h2 {
  margin: 0;
  font-size: 2rem;
}

.accounts-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
  gap: 1.5rem;
}

.account-card {
  background: white;
  border-radius: 16px;
  padding: 1.5rem;
  position: relative;
  transition: all 0.3s ease;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.account-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
}

.account-number {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 0.9rem;
}

.account-content {
  display: flex;
  gap: 1rem;
  margin-bottom: 0.5rem;
}

.account-avatar img,
.account-avatar .avatar-placeholder {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  object-fit: cover;
}

.account-avatar .avatar-placeholder {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.5rem;
  font-weight: bold;
}

.account-details {
  flex: 1;
  min-width: 0;
}

.account-details h3 {
  margin: 0 0 0.5rem 0;
  font-size: 1.2rem;
  color: #333;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.account-id {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
  flex-wrap: wrap;
}

.label {
  font-weight: 600;
  color: #666;
  font-size: 0.85rem;
}

.id-value {
  font-family: 'Courier New', monospace;
  color: #333;
  font-size: 0.85rem;
  word-break: break-all;
}

.copy-btn {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 1rem;
  padding: 0.25rem;
  opacity: 0.6;
  transition: opacity 0.2s;
}

.copy-btn:hover {
  opacity: 1;
}

.account-meta {
  display: flex;
  gap: 1rem;
  font-size: 0.8rem;
  color: #999;
}

.remove-btn {
  position: absolute;
  bottom: 1rem;
  right: 1rem;
  background: #ff4444;
  color: white;
  border: none;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 1.2rem;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
}

.remove-btn:hover {
  background: #cc0000;
  transform: scale(1.1);
}

.empty-state {
  text-align: center;
  color: white;
  padding: 4rem 2rem;
}

.empty-icon {
  font-size: 4rem;
  margin-bottom: 1rem;
  opacity: 0.7;
}

.empty-state p {
  font-size: 1.2rem;
  opacity: 0.8;
}

/* Toast notification */
.toast {
  position: fixed;
  bottom: 2rem;
  right: 2rem;
  background: white;
  color: #333;
  padding: 1rem 1.5rem;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  z-index: 1000;
  font-weight: 600;
}

.toast.success {
  border-left: 4px solid #06c755;
}

.toast.error {
  border-left: 4px solid #ff4444;
}

.toast.warning {
  border-left: 4px solid #ffa500;
}

.toast-enter-active,
.toast-leave-active {
  transition: all 0.3s ease;
}

.toast-enter-from,
.toast-leave-to {
  opacity: 0;
  transform: translateY(20px);
}

/* Responsive */
@media (max-width: 768px) {
  .container {
    padding: 1rem;
  }

  .title {
    font-size: 2rem;
  }

  .add-section {
    padding: 0 1rem;
  }

  .add-card {
    padding: 2rem 1.5rem;
  }

  .accounts-grid {
    grid-template-columns: 1fr;
  }

  .section-header {
    flex-direction: column;
    gap: 1rem;
    align-items: flex-start;
  }

  .toast {
    left: 1rem;
    right: 1rem;
    bottom: 1rem;
  }
}
</style>
