<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>早安圖生成器</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@tabler/icons@2.44.0/tabler-icons.min.css">
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; background: #f5f5f5; padding: 1rem; }
    
    .container { max-width: 420px; margin: 0 auto; background: white; border-radius: 12px; padding: 1.5rem; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
    
    h1 { font-size: 24px; margin-bottom: 1.5rem; text-align: center; color: #333; }
    
    .section { margin-bottom: 1.5rem; }
    .section h3 { font-size: 14px; font-weight: 600; color: #555; margin-bottom: 0.75rem; text-transform: uppercase; letter-spacing: 0.5px; }
    
    .upload-box {
      border: 2px dashed #ddd;
      border-radius: 8px;
      padding: 1.5rem;
      text-align: center;
      cursor: pointer;
      transition: all 0.2s;
      background: #fafafa;
    }
    .upload-box:hover { border-color: #999; background: #f0f0f0; }
    .upload-box input { display: none; }
    .upload-box-label { display: flex; flex-direction: column; align-items: center; gap: 8px; cursor: pointer; }
    .upload-icon { font-size: 32px; color: #999; }
    .upload-text { font-size: 13px; color: #999; }
    .upload-box.has-image { border-color: #ddd; background: white; }
    .upload-box.has-image .upload-icon { display: none; }
    .upload-box.has-image .upload-text { display: none; }
    
    .thumbnail { width: 100%; border-radius: 8px; max-height: 200px; object-fit: cover; }
    
    .input-group { margin-bottom: 1rem; }
    .input-label { font-size: 13px; color: #666; margin-bottom: 6px; display: block; font-weight: 500; }
    input[type="text"], textarea { 
      width: 100%; 
      padding: 0.75rem; 
      border: 1px solid #ddd; 
      border-radius: 6px;
      font-family: inherit;
      font-size: 15px;
      background: #fafafa;
      color: #333;
    }
    input[type="text"]:focus, textarea:focus { 
      outline: none; 
      border-color: #666;
      background: white;
    }
    textarea { resize: vertical; min-height: 80px; font-size: 14px; }
    
    .char-count { font-size: 12px; color: #999; margin-top: 4px; }
    
    .preview-section {
      background: #fafafa;
      border-radius: 8px;
      padding: 1.5rem;
      margin-bottom: 1.5rem;
      text-align: center;
    }
    .preview-section h3 { margin-bottom: 1rem; text-align: left; }
    
    #preview {
      width: 100%;
      max-width: 280px;
      margin: 0 auto;
      aspect-ratio: 9/16;
      position: relative;
      background: #e0e0e0;
      border-radius: 8px;
      overflow: hidden;
      display: flex;
      flex-direction: column;
    }
    
    #previewImage { width: 100%; height: 100%; object-fit: cover; }
    
    .preview-text-overlay {
      position: absolute;
      bottom: 0;
      left: 0;
      right: 0;
      padding: 1.5rem;
      background: linear-gradient(transparent, rgba(0,0,0,0.4));
      color: black;
      display: flex;
      flex-direction: column;
      justify-content: flex-end;
      min-height: 80px;
    }
    
    .preview-quote { 
      font-size: 16px; 
      line-height: 1.4; 
      font-weight: 500;
      word-wrap: break-word;
    }
    .preview-name {
      font-size: 14px;
      margin-top: 0.5rem;
      opacity: 0.9;
    }
    
    .button-group { display: flex; gap: 8px; }
    .btn {
      flex: 1;
      padding: 0.75rem;
      border: 1px solid #ddd;
      border-radius: 6px;
      background: white;
      color: #333;
      font-size: 15px;
      font-weight: 500;
      cursor: pointer;
      transition: all 0.2s;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 8px;
    }
    .btn:hover { background: #f5f5f5; border-color: #999; }
    .btn:active { transform: scale(0.98); }
    .btn.primary {
      background: #333;
      color: white;
      border-color: #333;
    }
    .btn.primary:hover { background: #555; border-color: #555; }
    .btn:disabled { opacity: 0.5; cursor: not-allowed; }
    
    .empty-state { text-align: center; padding: 2rem 1rem; color: #999; font-size: 14px; }
    
    .success-toast {
      position: fixed;
      bottom: 1.5rem;
      left: 1.5rem;
      right: 1.5rem;
      background: #4caf50;
      color: white;
      padding: 0.75rem 1rem;
      border-radius: 6px;
      font-size: 14px;
      animation: slideUp 0.3s ease;
      z-index: 1000;
    }
    
    @keyframes slideUp { from { transform: translateY(100px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
    @keyframes spin { to { transform: rotate(360deg); } }
  </style>
</head>
<body>

<div class="container">
  <h1>☀️ 早安圖生成器</h1>

  <div class="section">
    <h3>📸 上傳照片</h3>
    <label class="upload-box" id="uploadBox">
      <input type="file" id="photoInput" accept="image/*" />
      <div class="upload-box-label">
        <i class="ti ti-upload upload-icon"></i>
        <span class="upload-text">點擊上傳或拖拽照片</span>
      </div>
      <img id="uploadThumb" class="thumbnail" style="display:none;" />
    </label>
  </div>

  <div class="section">
    <h3>✍️ 輸入文字</h3>
    
    <div class="input-group">
      <label class="input-label">名字（20字以內）</label>
      <input type="text" id="nameInput" placeholder="例：小美" maxlength="20" />
      <div class="char-count"><span id="nameCount">0</span>/20</div>
    </div>

    <div class="input-group">
      <label class="input-label">金句</label>
      <textarea id="quoteInput" placeholder="例：早安！今天又是閃閃發光的一天"></textarea>
    </div>
  </div>

  <div class="preview-section">
    <h3>👁️ 預覽</h3>
    <div id="preview" style="opacity: 0.6;">
      <div class="empty-state">請先上傳照片</div>
    </div>
  </div>

  <div class="button-group">
    <button class="btn" id="resetBtn">重置</button>
    <button class="btn primary" id="downloadBtn" disabled>下載圖片</button>
  </div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
<script>
  const photoInput = document.getElementById('photoInput');
  const uploadBox = document.getElementById('uploadBox');
  const uploadThumb = document.getElementById('uploadThumb');
  const nameInput = document.getElementById('nameInput');
  const quoteInput = document.getElementById('quoteInput');
  const preview = document.getElementById('preview');
  const downloadBtn = document.getElementById('downloadBtn');
  const resetBtn = document.getElementById('resetBtn');
  const nameCount = document.getElementById('nameCount');

  let uploadedImage = null;

  uploadBox.addEventListener('dragover', (e) => {
    e.preventDefault();
    uploadBox.style.borderColor = '#666';
  });

  uploadBox.addEventListener('dragleave', () => {
    uploadBox.style.borderColor = '#ddd';
  });

  uploadBox.addEventListener('drop', (e) => {
    e.preventDefault();
    const files = e.dataTransfer.files;
    if (files.length > 0) handleImageUpload(files[0]);
  });

  photoInput.addEventListener('change', (e) => {
    if (e.target.files.length > 0) handleImageUpload(e.target.files[0]);
  });

  function handleImageUpload(file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      uploadedImage = e.target.result;
      uploadThumb.src = uploadedImage;
      uploadThumb.style.display = 'block';
      uploadBox.classList.add('has-image');
      updatePreview();
      downloadBtn.disabled = false;
    };
    reader.readAsDataURL(file);
  }

  nameInput.addEventListener('input', () => {
    nameCount.textContent = nameInput.value.length;
    updatePreview();
  });

  quoteInput.addEventListener('input', updatePreview);

  function updatePreview() {
    if (!uploadedImage) {
      preview.innerHTML = '<div class="empty-state">請先上傳照片</div>';
      preview.style.opacity = '0.6';
      return;
    }

    preview.innerHTML = `
      <img id="previewImage" src="${uploadedImage}" style="width:100%; height:100%; object-fit:cover;" />
      <div class="preview-text-overlay">
        <div class="preview-quote">${quoteInput.value || '金句在這裡'}</div>
        <div class="preview-name">${nameInput.value || '你的名字'}</div>
      </div>
    `;
    preview.style.opacity = '1';
  }

  downloadBtn.addEventListener('click', async () => {
    if (!uploadedImage) return;

    downloadBtn.disabled = true;
    downloadBtn.textContent = '準備中...';

    try {
      const canvas = await html2canvas(preview, {
        backgroundColor: null,
        scale: 2,
        useCORS: true,
        allowTaint: true
      });

      const link = document.createElement('a');
      link.href = canvas.toDataURL('image/png');
      link.download = `早安圖-${new Date().getTime()}.png`;
      link.click();

      const toast = document.createElement('div');
      toast.className = 'success-toast';
      toast.textContent = '✓ 圖片已下載！';
      document.body.appendChild(toast);
      setTimeout(() => toast.remove(), 3000);
    } catch (err) {
      alert('下載失敗，請重試');
      console.error(err);
    }

    downloadBtn.disabled = false;
    downloadBtn.textContent = '下載圖片';
  });

  resetBtn.addEventListener('click', () => {
    photoInput.value = '';
    nameInput.value = '';
    quoteInput.value = '';
    uploadThumb.style.display = 'none';
    uploadBox.classList.remove('has-image');
    nameCount.textContent = '0';
    downloadBtn.disabled = true;
    preview.innerHTML = '<div class="empty-state">請先上傳照片</div>';
    preview.style.opacity = '0.6';
  });
</script>

</body>
</html>
