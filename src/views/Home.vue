<template>
  <div class="home">
  <!-- Toast Notification -->
<div v-if="toast.show" class="toast" :class="toast.type">
  {{ toast.message }}
</div>

    <div class="site-titel">
    <div class="logo">
      <img src="../assets/photos/logo.png" alt="">
    </div>
   <div class="wtt"> FileChain Verifier</div>
    </div>
    <div class="site-description">
    Upload up to 2 files (max 20MB each) to instantly verify their 
    integrity using hashing. The last 20 results are securely logged and 
    accessible in the History page. <br>
    ⚠️ Please note that files larger than 10MB 
    may take extra time to process, especially on mobile devices.
    </div>
    <div class="site-dragToUpload">
<div
  class="upload-box-phone"
  :class="{ dragging: isDragging }"
  @click="onBoxClick"
  @dragenter.prevent="onDragEnter"
  @dragover.prevent="onDragOver"
  @dragleave.prevent="onDragLeave"
  @drop.prevent="onDrop"
>
<div class="iconholder">
<svg xmlns="http://www.w3.org/2000/svg" width="120" height="120" viewBox="0 0 640 640"><path fill="#3b82f6" d="M128 128C128 92.7 156.7 64 192 64L341.5 64C358.5 64 374.8 70.7 386.8 82.7L493.3 189.3C505.3 201.3 512 217.6 512 234.6L512 512C512 547.3 483.3 576 448 576L192 576C156.7 576 128 547.3 128 512L128 128zM336 122.5L336 216C336 229.3 346.7 240 360 240L453.5 240L336 122.5zM337 327C327.6 317.6 312.4 317.6 303.1 327L239.1 391C229.7 400.4 229.7 415.6 239.1 424.9C248.5 434.2 263.7 434.3 273 424.9L296 401.9L296 488C296 501.3 306.7 512 320 512C333.3 512 344 501.3 344 488L344 401.9L367 424.9C376.4 434.3 391.6 434.3 400.9 424.9C410.2 415.5 410.3 400.3 400.9 391L336.9 327z"/></svg></div>

  <div class="upload-text">
 <template v-if="selectedFiles.length === 0">
  <div class="primary">Click to Upload a file</div>
  <div class="secondary" v-if="!isMobile">or drag & drop your file here</div>
</template>
<template v-else>
  <div v-for="(file, idx) in selectedFiles" :key="idx" class="file-entry">
    <div class="primary">Selected: {{ file.name }}</div>
    <div class="secondary">{{ formatBytes(file.size) }}</div>
  </div>
</template>

  </div>
<div v-if="showUploadBox" class="upload-progress-box">
  <div class="progress-wrap">
    <svg viewBox="0 0 36 36" class="circular-chart">
      <path class="circle-bg"
        d="M18 2.0845
           a 15.9155 15.9155 0 0 1 0 31.831
           a 15.9155 15.9155 0 0 1 0 -31.831"/>
      <path class="circle"
        :stroke-dasharray="uploadProgress + ', 100'"
        d="M18 2.0845
           a 15.9155 15.9155 0 0 1 0 31.831
           a 15.9155 15.9155 0 0 1 0 -31.831"/>
      <text x="18" y="20.35" class="percentage">{{ Math.round(uploadProgress) }}%</text>
    </svg>
  </div>
  <div class="progress-info">{{ uploadStatus }}</div>
</div>
  <input ref="fileInput" type="file" class="visually-hidden" multiple @change="onFileChange" />
</div>
    </div>

    <div class="buttonholder">
     <div class="compare-submit"  @click="compareFiles">Compare</div>
    </div>
<div v-if="areEqual !== null" class="result-box" :class="areEqual ? 'same-box' : 'diff-box'">
  <div class="result-title">
    <p v-if="areEqual">✅ The files are identical</p>
    <p v-else>❌ The files are different</p>
  </div>

  <div class="hashes">
    <div class="hash">
      <div  class="hashfile"><strong>File 1 Hash:</strong></div>
      <span>
        {{ showFullHash1 ? hash1 : hash1.slice(0, 12) + "..." }}
        <button v-if="hash1.length > 12" @click="showFullHash1 = !showFullHash1">
          {{ showFullHash1 ? "Show less" : "Read more" }}
        </button>
      </span>
    </div>

    <div v-if="!areEqual" class="hash">
      <div class="hashfile"><strong>File 2 Hash:</strong></div>
      <span>
        {{ showFullHash2 ? hash2 : hash2.slice(0, 12) + "..." }}
        <button v-if="hash2.length > 12" @click="showFullHash2 = !showFullHash2">
          {{ showFullHash2 ? "Show less" : "Read more" }}
        </button>
      </span>
    </div>
  </div>
</div>


  </div>
</template>

<script>
export default {
  name: "Home",
  data() {
    return {
        lastHash: null,
    isDragging: false,
    selectedFiles: [],
    isMobile: false,
    hash1: "",
    hash2: "",
    areEqual: null,
    showFullHash1: false,
    showFullHash2: false,
    // upload progress UI
    showUploadBox: false,
    uploadProgress: 0,
    uploadStatus: '',
    maxUploadSize: 20 * 1024 * 1024, // 20 MB
    _uploadTimer: null,
    toast: {
  show: false,
  message: "",
  type: "info", // info | success | error
},

  };
  },
  mounted() {
    const mqCoarse = window.matchMedia && window.matchMedia('(pointer: coarse)').matches;
    const touch = ('ontouchstart' in window) || navigator.maxTouchPoints > 0;
    this.isMobile = mqCoarse || touch || window.innerWidth <= 900;

    const preventDefault = (e) => { e.preventDefault(); };
    ['dragenter','dragover','dragleave','drop'].forEach(evt => {
      document.addEventListener(evt, preventDefault, false);
    });
  },
  beforeDestroy() {
    if (this._uploadTimer) clearInterval(this._uploadTimer);
  },
  methods: {
    onBoxClick() { this.$refs.fileInput.click(); },

onFileChange(e) {
  const files = e.target.files;
  if (files && files.length) {
    Array.from(files).forEach(f => this.addFile(f));
  }
  e.target.value = '';
},

    onDragEnter() { if (this.isMobile) return; this.isDragging = true; },
    onDragOver()  { if (this.isMobile) return; this.isDragging = true; },
    onDragLeave() { if (this.isMobile) return; this.isDragging = false; },

onDrop(e) {
  if (this.isMobile) return;
  const files = e.dataTransfer?.files;
  if (files && files.length) {
    Array.from(files).forEach(f => this.addFile(f));
  }
  this.isDragging = false;
},

  addFile(f) {
    if (f.size > this.maxUploadSize) {
      this.showToast('The file is too large. Maximum allowed size is 20 MB', 'error');
      return;
    }
    const id = Date.now() + "-" + Math.random().toString(36).slice(2,9);
    const fileObj = {
      id,
      file: f,
      name: f.name,
      size: f.size,
      status: 'pending',
      serverHash: null
    };
    this.selectedFiles.push(fileObj);

    // show progress UI
    this.showUploadBox = true;
    this.uploadProgress = 0;
    this.uploadStatus = 'Uploading...';
    this.startUploadSimulation();

    // start actual upload (fire-and-forget but update state)
    this.handleFileUpload(fileObj);
  },
// async uploadFile(f) {
//     const formData = new FormData();
//     formData.append("file", f);

//     try {
//       const res = await fetch("http://localhost:5000/upload", {
//         method: "POST",
//         body: formData,
//       });
//       const data = await res.json();
//       console.log("✅ Uploaded:", data);

//       this.lastHash = data.hash; // نخزّن الهاش للعرض
//     } catch (err) {
// this.showToast("❌ Upload failed. Please try again.", "error");
//     }
//   },
  async handleFileUpload(fileObj) {
    fileObj.status = 'uploading';
    const formData = new FormData();
    formData.append("file", fileObj.file);

    try {
      const res = await fetch("https://file-checker-backend.onrender.com/upload", {
        method: "POST",
        body: formData,
      });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);

      const data = await res.json();

      fileObj.serverHash = data.hash;
      fileObj.status = 'done';
      this.lastHash = data.hash; // for display if needed
      // this.showToast(`Uploaded ${fileObj.name}`, 'success');

    } catch (err) {
      console.error('Upload failed', err);
      fileObj.status = 'error';
      // this.showToast(`Upload failed for ${fileObj.name}`, 'error');
    }
  },

async compareFiles() {
    const doneFiles = this.selectedFiles.filter(s => s.status === 'done' && s.serverHash);
    if (doneFiles.length !== 2) {
      this.areEqual = null;
      this.hash1 = "";
      this.hash2 = "";
      return this.showToast("⚠️ Please upload exactly 2 files first!", "error");
    }

  const formData = new FormData();
for (const f of doneFiles) {
  formData.append("files", f.file);
}


  try {
    const res = await fetch("https://file-checker-backend.onrender.com/compare", {
      method: "POST",
      body: formData,
    });
    const data = await res.json();

    this.hash1 = data.hash1;
    this.hash2 = data.hash2;
    this.areEqual = data.areEqual;
  } catch (err) {
   this.showToast("❌ Error comparing files:", "error");

    this.areEqual = null;
  }
},

 startUploadSimulation() {
    if (this._uploadTimer) clearInterval(this._uploadTimer);
    this._uploadTimer = setInterval(() => {
      const remaining = 100 - this.uploadProgress;
      const step = Math.max(1, Math.min(8, Math.round(remaining * 0.12)));
      this.uploadProgress = Math.min(100, this.uploadProgress + step);
      if (this.uploadProgress >= 100) {
        clearInterval(this._uploadTimer);
        this._uploadTimer = null;
        this.uploadStatus = 'Completed';
        // keep the box visible until uploads finish; hide after short delay
        setTimeout(() => {
          // hide only if no uploading files
          const uploading = this.selectedFiles.some(s => s.status === 'uploading');
          if (!uploading) this.showUploadBox = false;
        }, 1200);
      }
    }, 240);
  },

    formatBytes(bytes) {
      if (!bytes) return '';
      const sizes = ['B','KB','MB','GB'];
      if (bytes === 0) return '0 B';
      const i = Math.floor(Math.log(bytes) / Math.log(1024));
      return (bytes / Math.pow(1024, i)).toFixed(2) + ' ' + sizes[i];
    },
    showToast(message, type = "info") {
  this.toast.message = message;
  this.toast.type = type;
  this.toast.show = true;
  setTimeout(() => {
    this.toast.show = false;
  }, 3000); // يختفي بعد 3 ثواني
},

  },
};
</script>
<style>

body{
    display: flex;
  justify-content: center;
  align-items: center;
height: 100%;
}
.home{
    display: flex;
  align-items: center;

padding: 1rem;
flex-direction: column;
gap: 3rem;
  padding-bottom: 5rem;

}
.site-description{
  color: var(--white);
width: 92%;
display: flex;
align-items: center;
justify-content: center;
text-align: center;
font-size: 1rem;
line-height: 1.6;
}
.wtt{
   position: relative;
   top: 3px;
}
.site-titel{
  color: var(--white);
  margin-top: 3rem;
width: 98%;
gap: 1rem;
display: flex;
align-items: center;
justify-content: center;
font-size: 1.6rem;
font-weight: 900;

}
.site-dragToUpload{
  color: var(--white);
width: 90%;
display: flex;
justify-content: center;
align-items: center;

}
.upload-box-phone{
min-height: 14rem;
width: 80%;
border-radius: 10px;
border: 4px dashed var(--secend-color);
display: flex;
align-items: center;
color: var(--white);
font-size: 1.1rem;
flex-direction: column;
  overflow-wrap: break-word;   /* ✅ breaks long words */
  word-wrap: break-word;       /* ✅ older support */
  word-break: break-word;      /* ✅ fallback */
  white-space: normal; 
  text-align: center;
  gap: .2rem;
  padding: 1rem;
}
.buttonholder{
  height: auto;
  width: 90%;
  border-radius: 5px;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: .2rem .5rem;
  color: white;
}
.compare-submit{
  background-color: var(--secend-color);
    width: 60%;
    height: 3rem;
    border-radius: 8px;
    display: flex;
    justify-content: center;
    align-items: center;
    font-weight: bold;
    font-size: 20px;
    cursor: pointer;
}


.iconholder{
  margin-top: 2rem;
  margin-bottom: .5rem;
}
.logo{
  height: 3rem;
  width: auto;
  display: flex;
  justify-content: center;
  align-items: center;
}
.logo img{
   width: 5rem;
   height: 3.05rem;
   margin-left: .3rem;
   object-fit: contain; /* يحافظ على الشكل */
   margin-left: .3rem;
}

.upload-progress-box {
  width: 90%;
  max-width: 520px;
  margin-top: 1rem;
  display: flex;
  align-items: center;
  gap: 1rem;
  justify-content: center;
  flex-direction: column;
  color: var(--White);
}

/* circular progress chart */
.progress-wrap { width: 88px; height: 88px; }
.circular-chart { width: 88px; height: 88px; transform: rotate(-90deg); }
.circular-chart .circle-bg {
  fill: none;
  stroke: rgba(255,255,255,0.08);
  stroke-width: 3.8;
}
.circular-chart .circle {
  fill: none;
  stroke: var(--secend-color);
  stroke-width: 3.8;
  stroke-linecap: round;
  transition: stroke-dasharray 260ms ease, stroke 260ms ease;
}
.circular-chart .percentage {
  font-size: 10px;
  fill: var(--White);
  text-anchor: middle;
  transform: rotate(90deg);
}

/* progress text */
.progress-info {
  font-size: 0.95rem;
  opacity: 0.95;
  margin-top: 4px;
  text-align: center;
}
.visually-hidden{ position:absolute; width:1px; height:1px; overflow:hidden; clip:rect(0 0 0 0); white-space:nowrap; border:0; padding:0; margin:-1px; }
.upload-box-phone{ cursor:pointer; transition: box-shadow .18s ease, transform .12s ease, border-color .18s ease; }
.upload-box-phone.dragging{ box-shadow: 0 10px 30px rgba(59,130,246,0.15); transform: translateY(-3px); border-color: var(--secend-color); }
.upload-text .primary{ font-weight:600; }
.upload-text .secondary{ font-size:0.9rem; opacity:.85; margin-top:.2rem; }

.result-box {
  margin-top: 1rem;
  width: 95%;
  padding: 1rem;
  border-radius: 10px;
  font-weight: bold;
  text-align: center;
  display: flex;
  flex-direction: column;
  gap: 2rem;
  margin-bottom: 3rem;
}


.result-title p {
  font-size: 1.2rem;
  margin: 0;
}

.hashes {
  display: flex;
  flex-direction: column; /* Mobile: stack */
  gap: 01rem;
}

.hash {
  background: var(--gray);
  color: #000;
  padding: 0.6rem;
  border-radius: 6px;
  word-break: break-all;
  font-size: 0.9rem;
  display: flex;
 justify-content: start;
 gap: 1rem;
    align-items: center;
}

.hash button {
  margin-left: 0.5rem;
  font-size: 0.8rem;
  color: #007bff;
  background: none;
  border: none;
  cursor: pointer;
}

.hash button:hover {
  text-decoration: underline;
}
.hashfile{
  background-color: var(--secend-color);
  color: White;
  flex-shrink: 0;       /* ✅ prevent shrinking */
  width: 6rem;
  padding: .5rem 0;
  text-align: center;
  border-radius: 4px;
}
.hash span {
  flex: 1;              /* ✅ take remaining space */
  white-space: normal;  /* ✅ allow wrapping */
  word-break: break-all;

}
.toast {
  position: fixed;
  top: 50%;
  left: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transform: translate(-50%, -50%);
  padding: 14px 22px;
  border-radius: 10px;
  color: white;
  font-weight: bold;
  z-index: 9999;
  opacity: 0.95;

  transition: opacity 0.3s ease, transform 0.3s ease;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.25);
  text-align: center;
  width: 80%;
  height: 3rem;
  font-size: 1rem;
}
.toast.info { background: #3498db; }
.toast.success { background: #2ecc71; }
.toast.error { background: #e74c3c; }


/* ---------------------DESKTOP ------------------------------------------------- */
@media(min-width:1024px){
  .result-box {
  margin-top: 1rem;
  width: 68%;
  padding: 1rem;
  border-radius: 10px;
  font-weight: bold;
  text-align: center;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-bottom: 8rem;
}

 .toast {

 width: 50%;
  font-size: 1.2rem;
}
  .compare-submit{
    width: 50%;
    font-size: 23px;
}
  .site-description{
width: 74%;
font-size: 1.1rem;
}
  .upload-progress-box { max-width: 60%; }
.site-titel{
  color: var(--white);
  margin-top: 2rem;
width: 50%;
gap: 1rem;
word-spacing: 2px;
display: flex;
align-items: center;
justify-content: center;
padding: 2rem;
font-size: 2.5rem;
font-weight: bold;
}

.logo img{
   width: 7rem;
   height: 5rem;
}
.upload-box-phone{
width: 60%;
min-height: 16rem;
color: var(--white);
font-size: 1.2rem;
gap: .3rem;
  overflow-wrap: break-word;   /* ✅ breaks long words */
  word-wrap: break-word;       /* ✅ older support */
  word-break: break-word;      /* ✅ fallback */
  white-space: normal; 
  text-align: center;
  gap: .2rem;
  padding: 1rem;
}
.buttonholder{
  height: auto;
  width: 45%;

}
}

</style>