<template>
  <div class="history-page">
    <div class="page-header">
      <div class="view-toggle">
        <button 
          @click="setViewMode('table')"
          :class="{ active: viewMode === 'table' }"
          class="toggle-btn"
        >
           Table
        </button>
        <button 
          @click="setViewMode('cards')" 
          :class="{ active: viewMode === 'cards' }"
          class="toggle-btn"
        >
           Cards
        </button>
      </div>
    </div>
    
    <!-- Loading indicator -->
    <div v-if="loading" class="loading">
      <div class="spinner"></div>
      Loading history...
    </div>
    
    <!-- Error message -->
    <div v-if="error" class="error">
      {{ error }}
    </div>
    
    <!-- History count -->
    <div class="stats-bar" v-if="!loading">
      <div class="stat-item">
        <span class="stat-label">Total uploads:</span>
        <span class="stat-value">{{ history.length }}</span>
      </div>
      <div class="stat-item" v-if="history.length > 0">
        <span class="stat-label">Latest:</span>
        <span class="stat-value">{{ formatDate(history[0]?.date, true) }}</span>
      </div>
    </div>

    <!-- TABLE VIEW -->
    <div v-if="viewMode === 'table' && history.length > 0" class="table-container">
      <div class="table-wrapper">
        <table class="history-table">
          <thead>
            <tr>
              <th class="th-name">Name</th>
              <th class="th-hash">Hash</th>
              <th class="th-date">Date</th>
              <th class="th-action">Compare</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, idx) in history" :key="idx" class="table-row">
              <td class="td-name">
                <div class="file-info">
                  <i class="file-icon">{{ getFileIcon(item.name) }}</i>
                  <span class="file-name" :title="item.name">{{ item.name }}</span>
                </div>
              </td>

              <td class="td-hash">
                <div class="hash-container">
                  <code class="hash-display" :title="item.hash">
                    {{ (item.hash || '').slice(0, 8) }}...
                  </code>
                  <button 
                    @click="copyHash(item.hash)" 
                    class="copy-btn"
                    title="Copy hash"
                  >
                    📋Copy
                  </button>
                </div>
              </td>

              <td class="td-date">
                <div class="date-info">
                  <span class="date-main">{{ formatDate(item.date) }}</span>
                  <span class="date-relative">{{ getRelativeTime(item.date) }}</span>
                </div>
              </td>

              <td class="td-action">
                <div class="compare-controls">
                  <!-- واحد input مخفي لكل صف، ref ديناميكي -->
                  <input 
                    :ref="'fileInput' + idx"
                    type="file" 
                    @change="compareFile($event, item.hash, idx)" 
                    :disabled="comparing === idx"
                    style="display: none;"
                  />
                  <button 
                    @click="openFilePicker(idx)"
                    :disabled="comparing === idx"
                    class="compare-btn"
                  >
                    <span v-if="comparing === idx" class="btn-loading">
                      <div class="mini-spinner"></div>
                      Comparing...
                    </span>
                    <span v-else>
                      Compare
                    </span>
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- CARDS VIEW -->
    <div v-if="viewMode === 'cards' && history.length > 0" class="cards-container">
      <div class="card" v-for="(item, idx) in history" :key="idx">
        <div class="card-header">
          <div class="file-info-card">
            <i class="file-icon-large">{{ getFileIcon(item.name) }}</i>
            <div class="file-details">
              <h4 class="file-name-card" :title="item.name">{{ item.name }}</h4>
              <p class="file-date">{{ formatDate(item.date) }}</p>
              <p class="file-relative-time">{{ getRelativeTime(item.date) }}</p>
            </div>
          </div>
        </div>
        
        <div class="card-body">
          <div class="hash-section">
            <label class="hash-label">File Hash:</label>
            <div class="hash-container-card">
              <code class="hash-full">{{ item.hash }}</code>
              <button @click="copyHash(item.hash)" class="copy-btn-card" title="Copy hash">
                📋 Copy
              </button>
            </div>
          </div>
          
          <div class="compare-section">
            <input 
              :ref="'cardFileInput' + idx"
              type="file" 
              @change="compareFile($event, item.hash, idx)" 
              :disabled="comparing === idx"
              style="display: none;"
            />
            <button 
              @click="openCardFilePicker(idx)"
              :disabled="comparing === idx"
              class="compare-btn-card"
            >
              <span v-if="comparing === idx" class="btn-loading">
                <div class="mini-spinner"></div>
                Comparing...
              </span>
              <span v-else>
                Compare with this file
              </span>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Empty State -->
    <div v-if="!loading && history.length === 0" class="empty-state">
      <div class="empty-icon">📂</div>
      <h3>No files uploaded yet</h3>
      <p>Start by uploading your first file</p>
      <router-link to="/" class="cta-button">
        <i class="btn-icon">⬆️</i>
        Go to Upload
      </router-link>
    </div>

    <!-- Compare result modal -->
    <div v-if="compareResult" class="modal-overlay" @click.self="compareResult = null">
      <div class="modal-content" :class="compareResult.areEqual ? 'same' : 'diff'">
        <div class="modal-header">
          <div class="modal-title">
            <i class="result-icon">{{ compareResult.areEqual ? '✅' : '❌' }}</i>
            <h3>Comparison Result</h3>
          </div>
          <button @click="compareResult = null" class="close-modal-btn">×</button>
        </div>
        
        <div class="modal-body">
          <div class="result-status">
            <p class="status-text" v-if="compareResult.areEqual">
              Files are identical! 
            </p>
            <p class="status-text" v-else>
              Files are different! 
            </p>
          </div>
          
          <div class="result-details">
            <div class="detail-section">
              <label class="detail-label">Uploaded File:</label>
              <div class="detail-value file-detail">
                <i class="file-icon">{{ getFileIcon(compareResult.uploadedFile) }}</i>
                {{ compareResult.uploadedFile }}
              </div>
            </div>
            
            <div class="detail-section">
              <label class="detail-label">Uploaded Hash:</label>
              <div class="detail-value hash-detail">
                <code>{{ compareResult.uploadedHash }}</code>
                <button @click="copyHash(compareResult.uploadedHash)" class="copy-btn-small">Copy</button>
              </div>
            </div>
            
            <div class="detail-section">
              <label class="detail-label">Target Hash:</label>
              <div class="detail-value hash-detail">
                <code>{{ compareResult.targetHash }}</code>
                <button @click="copyHash(compareResult.targetHash)" class="copy-btn-small">Copy</button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Toast notification -->
    <div v-if="toast.show" :class="`toast toast-${toast.type}`">
      <i class="toast-icon">{{ getToastIcon(toast.type) }}</i>
      {{ toast.message }}
    </div>
  </div>
</template>

<script>
export default {
  name: "History",
  data() {
    return {
      history: [],
      compareResult: null,
      loading: false,
      error: null,
      userSelected: false,
      comparing: null,
      viewMode: 'table',
      toast: {
        show: false,
        message: '',
        type: 'info'
      },
      _historyInterval: null,
    };
  },
  mounted() {
    this.updateViewMode();
    window.addEventListener('resize', this.updateViewMode);
    
    this.loadHistory();
this._historyInterval = setInterval(() => this.loadHistory(false), 10000);
  },
  beforeDestroy() {
    if (this._historyInterval) {
      clearInterval(this._historyInterval);
    }
    window.removeEventListener('resize', this.updateViewMode);
  this.loadHistory(true); // أول مرة بس يظهر loading
this._historyInterval = setInterval(() => this.loadHistory(false), 10000);

  },

  methods: {
    updateViewMode() {
      if (window.innerWidth <= 768) {
        this.viewMode = 'cards';
      } else if (this.viewMode === 'cards' && window.innerWidth > 768) {
        this.viewMode = 'table';
      }
    },
  setViewMode(mode) {
    this.viewMode = mode;
    this.userSelected = true; // المستخدم اختار
  },
  updateViewMode() {
    if (this.userSelected) return; // لا تغير إذا المستخدم اختار
    if (window.innerWidth <= 768) {
      this.viewMode = 'cards';
    } else {
      this.viewMode = 'table';
    }
  },
async loadHistory(showLoading = false) {
  if (showLoading) this.loading = true;
  this.error = null;

  try {
    const res = await fetch("https://file-checker-backend.onrender.com/history");
    if (!res.ok) throw new Error(`HTTP ${res.status}: ${res.statusText}`);

    const data = await res.json();
    this.history = data || [];
  } catch (err) {
    console.error("❌ loadHistory error:", err);
    this.error = `Problem in downloading history try again `;
    this.history = [];
  } finally {
    if (showLoading) this.loading = false;
  }
},


    // فتح ملف المقارنة من الجدول
 openFilePicker(idx) {
    const input = this.$refs['fileInput' + idx];
    if (input) input.click(); // بدل [0].click()
  },

    // فتح ملف المقارنة من الكارت
  openCardFilePicker(idx) {
    const input = this.$refs['cardFileInput' + idx];
    if (input) input.click(); // نفس الشي
  },

     async compareFile(event, targetHash, idx) {
    const file = event.target.files[0];
    if (!file) return;

    this.comparing = idx;
    try {
      const formData = new FormData();
      formData.append("file", file);

      const res = await fetch(`https://file-checker-backend.onrender.com/history/compare/${targetHash}`, {
        method: "POST",
        body: formData,
      });

      if (!res.ok) throw new Error("Server error");
      const data = await res.json();

      this.compareResult = {
        ...data,
        uploadedFile: file.name,
      };
    } catch (err) {
      this.showToast("Comparison failed, try again.", "error");
    } finally {
      this.comparing = null;
      event.target.value = ""; // يسمح ترفع نفس الملف مره ثانية
    }
  },

    formatDate(iso, short = false) {
      if (!iso) return "Unknown";
      try {
        const d = new Date(iso);
        if (short) {
          return d.toLocaleDateString();
        }
        return d.toLocaleString();
      } catch (err) {
        return "Invalid Date";
      }
    },

    getRelativeTime(iso) {
      if (!iso) return "";
      try {
        const now = new Date();
        const date = new Date(iso);
        const diff = now - date;
        const minutes = Math.floor(diff / 60000);
        const hours = Math.floor(minutes / 60);
        const days = Math.floor(hours / 24);

        if (minutes < 1) return "Just now";
        if (minutes < 60) return `${minutes}m ago`;
        if (hours < 24) return `${hours}h ago`;
        if (days < 30) return `${days}d ago`;
        return this.formatDate(iso, true); // <- استخدمت this هنا
      } catch (err) {
        return "";
      }
    },

    getFileIcon(fileName) {
      if (!fileName) return "📄";
      const ext = fileName.toLowerCase().split('.').pop();
      const icons = {
        'pdf': '',
        'doc': '', 'docx': '',
        'xls': '', 'xlsx': '',
        'ppt': '', 'pptx': '',
        'txt': '',
        'js': '', 'html': '', 'css': '', 'json': '',
        'jpg': '', 'jpeg': '', 'png': '', 'gif': '',
        'mp4': '', 'avi': '', 'mov': '',
        'mp3': '', 'wav': '',
        'zip': '', 'rar': '', '7': '',
      };
      return icons[ext] || "";
    },

    getToastIcon(type) {
      const icons = {
        'success': '✅',
        'error': '❌',
        'warning': '⚠️',
        'info': 'ℹ️'
      };
      return icons[type] || 'ℹ️';
    },

    copyHash(hash) {
      if (!hash) return;
      if (navigator.clipboard) {
        navigator.clipboard.writeText(hash);
        // this.showToast("Hash copied to clipboard!", "success");
      } else {
        const textArea = document.createElement('textarea');
        textArea.value = hash;
        document.body.appendChild(textArea);
        textArea.select();
        document.execCommand('copy');
        document.body.removeChild(textArea);
        // this.showToast("Hash copied to clipboard!", "success");
      }
    },

    showToast(message, type = 'info') {
      this.toast.message = message;
      this.toast.type = type;
      this.toast.show = true;
      setTimeout(() => {
        this.toast.show = false;
      }, 3000);
    }
  },
};
</script>

<style scoped>
:root {
  --main-color: #111827;
  --White: #fbfbfb;
  --secend-color: #3B82F6;
    --gray:#1b2b4d;
    --hr:#505760;
    --support-color:#18233b;
    --buttons-clr:#ffffff1a;

}

html[data-theme="light"] {
  --main-color: #f6f0df;  
  --White:      #0F1724;    
  --secend-color: #2563EB;
  --gray:         #eae7df;  
  --support-color:#e5ddc7;  
  --hr: #D1D5DB;      
  --buttons-clr:#a49d9d7e;      
}
.history-page {
    margin-top: 2rem;
  margin-bottom: 3rem;
  padding: 1.5rem;
  color: var(--white);
  min-height: 100vh;
  background: var(--main-color);
}

.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
  flex-wrap: wrap;
  gap: 1rem;
}

.page-header h2 {
  margin: 0;
  font-size: 2rem;
  background: linear-gradient(45deg, #fff, #e8f4fd);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.view-toggle {
  display: flex;
  background: var(--gray);
  border-radius: 8px;
  padding: 4px;
}

.toggle-btn {
  padding: 0.5rem 1rem;
  border: none;
  background: transparent;
  color: var(--white);
  cursor: pointer;
  border-radius: 6px;
  transition: all 0.2s ease;
  font-size: 0.9rem;
}

.toggle-btn.active {
  color: white;
  background:var(--secend-color);
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.loading {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  padding: 2rem;
  background: rgba(52, 152, 219, 0.1);
  border: 1px solid rgba(52, 152, 219, 0.3);
  border-radius: 12px;
  color: var(--support-color);
}

.spinner, .mini-spinner {
  width: 20px;
  height: 20px;
  border: 2px solid rgba(52, 152, 219, 0.3);
  color: var(--support-color);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.mini-spinner {
  width: 16px;
  height: 16px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.error {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 1rem;
  background: rgba(231, 76, 60, 0.1);
  border: 1px solid rgba(231, 76, 60, 0.3);
  border-radius: 8px;
  color: #e74c3c;
  margin-bottom: 1rem;
}

.stats-bar {
  display: flex;
  gap: 2rem;
  margin-bottom: 2rem;
  flex-wrap: wrap;
}

.stat-item {
  background: var(--support-color);
  padding: 0.8rem 1.2rem;
  border-radius: 8px;
  backdrop-filter: blur(10px);
}

.stat-label {
  color: var(--White);
  font-size: 0.9rem;
  margin-right: 0.5rem;
}

.stat-value {
  color: var(--White);
  font-weight: bold;
}

/* TABLE STYLES */
.table-container {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(10px);
margin-bottom: 5rem;
}

.table-wrapper {
  overflow-x: auto;
  max-width: 100%;
}

.history-table {
  width: 100%;
  border-collapse: collapse;
  background: transparent;
}

.history-table th {
  background: var(--support-color);
  color: var(--white);
  padding: 1.2rem;
  text-align: left;
  font-weight: 600;
  border-bottom: 2px solid rgba(255, 255, 255, 0.1);
}

.header-icon {
  margin-right: 0.5rem;
}

.th-name { width: 35%; }
.th-hash { width: 25%; }
.th-date { width: 25%; }
.th-action { width: 15%; }

.table-row {
  transition: all 0.2s ease;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.table-row:hover {
  background: rgba(255, 255, 255, 0.05);

}

.history-table td {
  padding: 1rem 1.2rem;
  vertical-align: middle;
    background: var(--gray);
}

.file-info {
  display: flex;
  align-items: center;
  gap: 0.8rem;
}

.file-icon {
  font-size: 1.2rem;
}

.file-name {
  font-weight: 500;
  max-width: 200px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.hash-container {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.hash-display {
  font-family: 'Courier New', monospace;
  background: rgba(52, 152, 219, 0.1);
  color: var(--secend-color);
  padding: 0.3rem 0.6rem;
  border-radius: 4px;
  font-size: 0.85rem;
}

.copy-btn, .copy-btn-small {
  background: rgba(255, 255, 255, 0.1);
  border: none;
  color: var(--white);
  padding: 0.3rem;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 0.8rem;
  margin-left: .4rem;
}

.copy-btn:hover, .copy-btn-small:hover {
  background: rgba(255, 255, 255, 0.2);
}
.card-body{
  padding: .8rem;
}
.date-info {
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
}

.date-main {
  font-weight: 500;
  font-size: 0.9rem;
}

.date-relative {
  font-size: 0.75rem;
  color: var(--White);
}

.compare-controls {
  display: flex;
  justify-content: center;
}

.compare-btn {
  background: linear-gradient(45deg, #27ae60, #2ecc71);
  color: var(--white);
  border: none;
  padding: 0.6rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 0.85rem;
  display: flex;
  align-items: center;
  gap: 0.4rem;
  min-width: 100px;
  justify-content: center;
}

.compare-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(46, 204, 113, 0.3);
}

.compare-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.btn-loading {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

/* CARDS STYLES */
.cards-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
  gap: 1rem;
}

.card {
  background: var(--support-color);
  border-radius: 12px;
 padding: 1rem 10px;
  width: 85%;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  transition: all 0.3s ease;
}

.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 32px rgba(0, 0, 0, 0.2);
  border-color: rgba(255, 255, 255, 0.2);
}

.card-header {
  margin-bottom: 1.5rem;
}

.file-info-card {
  display: flex;
  align-items: flex-start;
  gap: 1rem;
}

.file-icon-large {
  font-size: 2rem;
}

.file-details {
  flex: 1;
}

.file-name-card {
  margin: 0 0 0.5rem 0;
  font-size: 1.1rem;
  color: var(--White);
  word-break: break-word;
  min-height: 73px;
}

.file-date {
  margin: 0 0 0.3rem 0;
  color: #6a6d6e;
  font-size: 0.9rem;
}

.file-relative-time {
  margin: 0;
  color: #818586;
  font-size: 0.8rem;
}

.hash-section {
  margin-bottom: 1.5rem;
}

.hash-label {
  display: block;
  color: #bdc3c7;
  font-size: 0.9rem;
  margin-bottom: 0.5rem;
}

.hash-container-card {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  flex-wrap: wrap;
}

.hash-full {
  font-family: 'Courier New', monospace;
  background: rgba(52, 152, 219, 0.1);
  color: var(--secend-color);
  padding: 0.5rem;
  border-radius: 6px;
  font-size: 0.8rem;
  word-break: break-all;
  flex: 1;
  min-width: 0;
}

.copy-btn-card {
  background: #ffffff1a;
  border: none;
  color: var(--white);
  padding: 0.5rem 0.8rem;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 0.85rem;
  white-space: nowrap;
}

.copy-btn-card:hover {
  background: rgba(255, 255, 255, 0.2);
}

.compare-btn-card {
  width: 100%;
  background: linear-gradient(45deg, #27ae60, #2ecc71);
  color: var(--white);
  border: none;
  padding: 0.8rem 1rem;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.compare-btn-card:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(46, 204, 113, 0.4);
}

.compare-btn-card:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* EMPTY STATE */
.empty-state {
  text-align: center;
  padding: 4rem 2rem;
  color: #bdc3c7;
}

.empty-icon {
  font-size: 4rem;
  margin-bottom: 1rem;
  opacity: 0.5;
}

.empty-state h3 {
  margin: 0 0 1rem 0;
  color: #ecf0f1;
}

.cta-button {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  background: linear-gradient(45deg, #3498db, #2980b9);
  color: var(--white);
  text-decoration: none;
  padding: 0.8rem 1.5rem;
  border-radius: 8px;
  margin-top: 1rem;
  transition: all 0.2s ease;
}

.cta-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(52, 152, 219, 0.4);
  text-decoration: none;
}

/* MODAL STYLES */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(5px);
  padding: 1rem;
}

.modal-content {
  background: var(--gray);
  border-radius: 16px;
  width: 80%;
  overflow-y: auto;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
  animation: modalSlide 0.3s ease;
  padding: .5rem;
}
.modal-body{
  padding: 1rem;
  gap: .8rem;
  overflow: hidden;
}

@keyframes modalSlide {
  from {
    opacity: 0;
    transform: translateY(-30px) scale(0.9);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 2rem;
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
}

.modal-title {

  display: flex;
  align-items: center;
  gap: 0.8rem;
}

.result-icon {
  font-size: 1.5rem;
}
.status-text{
  font-weight: 900;
  font-size: 1.1rem;
}
.modal-header h3 {
  margin: 0;
  color: var(--White);
}
.close-modal-btn{
  padding: .5rem;
}
.detail-label{
  font-weight: 700;
}

   @media (max-width: 1024px) { 
.history-page {
  margin-top: 4rem;
  padding: .5rem;
}
.cards-container {
  gap: 1.5rem;
  display: flex;
  justify-content: center;
align-items: center;
margin-bottom: 5rem;
flex-direction: column;

}
   }
</style>


<!-- ------------------------------------------------------------------------------------------- -->



