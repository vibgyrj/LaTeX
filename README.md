<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>LaTeX Compiler</title>
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;600&family=Syne:wght@400;700;800&display=swap" rel="stylesheet">
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --bg: #0f0f13;
    --panel: #16161d;
    --panel2: #1c1c26;
    --border: #2a2a38;
    --accent: #e8c56d;
    --accent2: #7c6af7;
    --text: #e2e2e8;
    --muted: #6b6b80;
    --error: #f07070;
    --success: #6ddfb0;
    --mono: 'JetBrains Mono', monospace;
    --sans: 'Syne', sans-serif;
  }

  body {
    background: var(--bg); color: var(--text);
    font-family: var(--sans); height: 100vh;
    display: flex; flex-direction: column; overflow: hidden;
  }

  /* ── HEADER ── */
  header {
    display: flex; align-items: center; justify-content: space-between;
    padding: 12px 20px; border-bottom: 1px solid var(--border);
    background: var(--panel); flex-shrink: 0; gap: 12px; flex-wrap: wrap;
  }

  .logo { font-size: 1.15rem; font-weight: 800; color: var(--accent); }
  .logo span { color: var(--muted); font-weight: 400; }

  .header-right { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; }

  .main-name-wrap {
    display: flex; align-items: center; gap: 6px;
    font-family: var(--mono); font-size: 0.72rem; color: var(--muted);
  }
  .main-name-wrap input {
    background: #1e1e2a; border: 1px solid var(--border); color: var(--text);
    font-family: var(--mono); font-size: 0.72rem; padding: 4px 8px;
    border-radius: 5px; outline: none; width: 140px;
  }
  .main-name-wrap input:focus { border-color: var(--accent2); }

  .status {
    font-family: var(--mono); font-size: 0.7rem; padding: 4px 12px;
    border-radius: 20px; background: #1e1e2a; border: 1px solid var(--border);
    color: var(--muted); transition: all 0.3s; white-space: nowrap;
  }
  .status.running { color: var(--accent);  border-color: var(--accent);  background: #2a2410; }
  .status.success { color: var(--success); border-color: var(--success); background: #0f2a1e; }
  .status.error   { color: var(--error);   border-color: var(--error);   background: #2a0f0f; }

  .btn {
    border: none; padding: 7px 14px; border-radius: 6px;
    font-family: var(--sans); font-weight: 700; font-size: 0.76rem;
    cursor: pointer; transition: opacity 0.2s, transform 0.1s;
    letter-spacing: 0.03em; white-space: nowrap;
  }
  .btn:hover   { opacity: 0.85; }
  .btn:active  { transform: scale(0.97); }
  .btn:disabled { opacity: 0.35; cursor: not-allowed; }
  .btn-compile  { background: var(--accent);  color: #0f0f13; }
  .btn-pdf      { background: var(--accent2); color: #fff; display: none; }
  .btn-log      { background: #2a2a38; color: var(--muted); border: 1px solid var(--border); display: none; }
  .btn-log:hover { color: var(--text); }

  /* ── LAYOUT ── */
  .workspace {
    display: grid;
    grid-template-columns: 210px 1fr 1fr;
    flex: 1; overflow: hidden;
  }

  /* ── FILE PANEL ── */
  .file-panel {
    background: var(--panel); border-right: 1px solid var(--border);
    display: flex; flex-direction: column; overflow: hidden;
  }

  .pane-header {
    padding: 8px 12px; background: var(--panel);
    border-bottom: 1px solid var(--border);
    font-size: 0.65rem; font-family: var(--mono); color: var(--muted);
    text-transform: uppercase; letter-spacing: 0.08em;
    display: flex; align-items: center; justify-content: space-between;
    flex-shrink: 0;
  }

  .drop-zone {
    margin: 10px; border: 1.5px dashed var(--border);
    border-radius: 8px; padding: 12px 10px; text-align: center;
    cursor: pointer; transition: border-color 0.2s, background 0.2s;
    font-size: 0.68rem; color: var(--muted); font-family: var(--mono); flex-shrink: 0;
  }
  .drop-zone:hover, .drop-zone.dragover {
    border-color: var(--accent2); background: #1a1a2e; color: var(--text);
  }
  .drop-zone svg { display: block; margin: 0 auto 6px; opacity: 0.4; }

  #fileInput { display: none; }

  .file-list { flex: 1; overflow-y: auto; padding: 4px 6px; }

  .file-item {
    display: flex; align-items: center; gap: 6px;
    padding: 5px 8px; border-radius: 5px;
    font-family: var(--mono); font-size: 0.7rem; color: var(--muted);
    transition: background 0.15s; user-select: none;
    cursor: pointer;
  }
  .file-item:hover { background: var(--panel2); color: var(--text); }
  .file-item.active { background: #1e1e30; color: var(--text); border-left: 2px solid var(--accent2); padding-left: 6px; }
  .file-item .fname { flex: 1; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
  .file-item .del {
    opacity: 0; font-size: 0.65rem; color: var(--error);
    padding: 2px 4px; border-radius: 3px; transition: opacity 0.15s;
    flex-shrink: 0;
  }
  .file-item:hover .del { opacity: 1; }
  .file-item .ext {
    font-size: 0.58rem; padding: 1px 4px; border-radius: 3px;
    background: #2a2a38; color: var(--muted); white-space: nowrap; flex-shrink: 0;
  }
  .file-item.main-file .ext { background: #2a2410; color: var(--accent); }

  .no-files { padding: 12px; font-family: var(--mono); font-size: 0.68rem; color: var(--muted); text-align: center; }

  .file-hint { padding: 6px 12px 8px; font-family: var(--mono); font-size: 0.62rem; color: var(--muted); border-top: 1px solid var(--border); flex-shrink: 0; line-height: 1.5; }

  /* ── EDITOR ── */
  .editor-pane { display: flex; flex-direction: column; overflow: hidden; border-right: 1px solid var(--border); }

  textarea {
    flex: 1; background: var(--bg); color: var(--text);
    font-family: var(--mono); font-size: 0.82rem; line-height: 1.7;
    padding: 18px; border: none; outline: none; resize: none; tab-size: 2;
  }
  textarea::selection { background: #3a3060; }

  /* ── PREVIEW ── */
  .preview-pane { display: flex; flex-direction: column; overflow: hidden; }

  iframe { flex: 1; border: none; background: #fff; }

  .log-box {
    display: none; flex: 1; overflow-y: auto;
    padding: 16px 20px; font-family: var(--mono); font-size: 0.73rem;
    line-height: 1.7; color: var(--error); white-space: pre-wrap; word-break: break-all;
  }
  .log-box.visible { display: block; }

  .placeholder {
    flex: 1; display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    color: var(--muted); font-size: 0.82rem; gap: 10px; font-family: var(--mono);
  }
  .placeholder svg { opacity: 0.18; }

  .info-txt { font-family: var(--mono); font-size: 0.63rem; color: var(--muted); }

  ::-webkit-scrollbar { width: 5px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 3px; }
</style>
</head>
<body>

<header>
  <div class="logo">TeX<span>local</span></div>
  <div class="header-right">
    <div class="main-name-wrap">
      <span>main:</span>
      <input type="text" id="mainName" value="doc.tex" title="Main .tex filename for compilation">
    </div>
    <span class="info-txt">Ctrl+Enter</span>
    <div class="status" id="status">idle</div>
    <button class="btn btn-log"      id="downloadLogBtn">⬇ .log</button>
    <button class="btn btn-pdf"      id="downloadPdfBtn">⬇ PDF</button>
    <button class="btn btn-compile"  id="compileBtn" onclick="compile()">▶ Compile</button>
  </div>
</header>

<div class="workspace">

  <!-- ── FILE PANEL ── -->
  <div class="file-panel">
    <div class="pane-header">
      <span>files</span>
      <span id="fileCount">0</span>
    </div>

    <div class="drop-zone" id="dropZone" onclick="document.getElementById('fileInput').click()">
      <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
        <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/>
        <polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/>
      </svg>
      Drop files here<br>or click to browse
    </div>
    <input type="file" id="fileInput" multiple
      accept=".tex,.sty,.cls,.bib,.png,.jpg,.jpeg,.pdf,.eps,.tikz,.def,.cfg,.ldf,.bbx,.cbx,.lbx">

    <div class="file-list" id="fileList">
      <div class="no-files">No support files uploaded</div>
    </div>

    <div class="file-hint">Click file → load in editor<br>✕ → remove from list</div>
  </div>

  <!-- ── EDITOR ── -->
  <div class="editor-pane">
    <div class="pane-header">
      <span id="editorLabel">editor — doc.tex</span>
      <span class="info-txt" id="lineCount">0 lines</span>
    </div>
    <textarea id="editor" spellcheck="false"></textarea>
  </div>

  <!-- ── PREVIEW ── -->
  <div class="preview-pane">
    <div class="pane-header">
      <span id="previewLabel">preview</span>
      <span class="info-txt" id="compileTime"></span>
    </div>
    <div class="placeholder" id="placeholder">
      <svg width="46" height="46" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.2">
        <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/>
        <polyline points="14 2 14 8 20 8"/>
        <line x1="16" y1="13" x2="8" y2="13"/>
        <line x1="16" y1="17" x2="8" y2="17"/>
      </svg>
      <span>PDF preview appears here</span>
    </div>
    <iframe id="preview" style="display:none;"></iframe>
    <div class="log-box" id="logBox"></div>
  </div>

</div>

<script>
// ── state ─────────────────────────────────────────────────
const uploadedFiles = {};      // filename → { file: File, content: string|null }
let activeFile     = null;     // filename currently shown in editor
let lastPdfB64     = null;
let lastLogText    = null;
let lastPdfName    = 'output.pdf';
let lastLogName    = 'output.log';

// ── DOM refs ──────────────────────────────────────────────
const editor         = document.getElementById('editor');
const preview        = document.getElementById('preview');
const logBox         = document.getElementById('logBox');
const statusEl       = document.getElementById('status');
const compileBtn     = document.getElementById('compileBtn');
const downloadPdfBtn = document.getElementById('downloadPdfBtn');
const downloadLogBtn = document.getElementById('downloadLogBtn');
const placeholder    = document.getElementById('placeholder');
const lineCount      = document.getElementById('lineCount');
const compileTime    = document.getElementById('compileTime');
const previewLabel   = document.getElementById('previewLabel');
const editorLabel    = document.getElementById('editorLabel');
const mainNameEl     = document.getElementById('mainName');
const fileListEl     = document.getElementById('fileList');
const fileCountEl    = document.getElementById('fileCount');
const dropZone       = document.getElementById('dropZone');
const fileInput      = document.getElementById('fileInput');

// ── default template ──────────────────────────────────────
editor.value = `\\documentclass{article}
\\usepackage{amsmath}
\\title{My Document}
\\author{Author Name}
\\date{\\today}

\\begin{document}
\\maketitle

\\section{Introduction}
Hello from your local \\LaTeX\\ compiler!

% To include another file:
% \\input{chapter1}

\\end{document}`;
updateLineCount();

// ── editor events ─────────────────────────────────────────
editor.addEventListener('input', () => {
  updateLineCount();
  // keep in-memory content synced for the active file
  if (activeFile && uploadedFiles[activeFile]) {
    uploadedFiles[activeFile].content = editor.value;
  }
});

editor.addEventListener('keydown', e => {
  if (e.ctrlKey && e.key === 'Enter') { e.preventDefault(); compile(); }
  if (e.key === 'Tab') {
    e.preventDefault();
    const s = editor.selectionStart;
    editor.value = editor.value.substring(0, s) + '  ' + editor.value.substring(editor.selectionEnd);
    editor.selectionStart = editor.selectionEnd = s + 2;
  }
});

function updateLineCount() {
  lineCount.textContent = editor.value.split('\n').length + ' lines';
}

// ── file upload ───────────────────────────────────────────
fileInput.addEventListener('change', e => { addFiles(e.target.files); fileInput.value = ''; });
dropZone.addEventListener('dragover',  e => { e.preventDefault(); dropZone.classList.add('dragover'); });
dropZone.addEventListener('dragleave', () => dropZone.classList.remove('dragover'));
dropZone.addEventListener('drop', e => {
  e.preventDefault(); dropZone.classList.remove('dragover');
  addFiles(e.dataTransfer.files);
});

function addFiles(fileObjs) {
  for (const f of fileObjs) {
    const isText = /\.(tex|sty|cls|bib|tikz|def|cfg|ldf|bbx|cbx|lbx|txt)$/i.test(f.name);

    if (isText) {
      const reader = new FileReader();
      reader.onload = ev => {
        uploadedFiles[f.name] = { file: f, content: ev.target.result };
        // auto-set main name if first .tex
        if (f.name.endsWith('.tex') && mainNameEl.value === 'doc.tex') {
          mainNameEl.value = f.name;
        }
        renderFileList();
      };
      reader.readAsText(f);
    } else {
      // binary files (images, etc.) — keep file object, no text content
      uploadedFiles[f.name] = { file: f, content: null };
      renderFileList();
    }
  }
}

// ── click file → load in editor ───────────────────────────
function loadFileInEditor(name) {
  const entry = uploadedFiles[name];
  if (!entry) return;

  if (entry.content === null) {
    alert(`"${name}" is a binary file (image/pdf) and cannot be edited as text.`);
    return;
  }

  // save current editor content back to whichever file was open
  if (activeFile && uploadedFiles[activeFile]) {
    uploadedFiles[activeFile].content = editor.value;
  }

  activeFile = name;
  editor.value = entry.content;
  editorLabel.textContent = 'editor — ' + name;

  // ✅ auto-update main: field so compile targets this file
  if (name.endsWith('.tex')) {
    mainNameEl.value = name;
  }

  updateLineCount();
  renderFileList();
  editor.focus();
}

function removeFile(name, event) {
  event.stopPropagation();   // don't trigger loadFileInEditor
  delete uploadedFiles[name];
  if (activeFile === name) {
    activeFile = null;
    editorLabel.textContent = 'editor — doc.tex';
  }
  renderFileList();
}

function renderFileList() {
  const names = Object.keys(uploadedFiles);
  fileCountEl.textContent = names.length;
  if (names.length === 0) {
    fileListEl.innerHTML = '<div class="no-files">No support files uploaded</div>';
    return;
  }
  fileListEl.innerHTML = names.map(name => {
    const ext    = name.includes('.') ? name.split('.').pop() : '?';
    const isMain = name === mainNameEl.value;
    const isActive = name === activeFile;
    const isBinary = uploadedFiles[name].content === null;
    return `
      <div class="file-item ${isMain ? 'main-file' : ''} ${isActive ? 'active' : ''}"
           onclick="loadFileInEditor('${name}')">
        <span class="ext">.${ext}</span>
        <span class="fname" title="${name}">${name}</span>
        ${isBinary ? '<span style="font-size:0.58rem;color:var(--muted);">bin</span>' : ''}
        <span class="del" onclick="removeFile('${name}', event)" title="Remove">✕</span>
      </div>`;
  }).join('');
}

mainNameEl.addEventListener('input', renderFileList);

// ── compile ───────────────────────────────────────────────
async function compile() {
  // save current editor → active file before sending
  if (activeFile && uploadedFiles[activeFile]) {
    uploadedFiles[activeFile].content = editor.value;
  }

  const source = editor.value.trim();
  if (!source) return;

  compileBtn.disabled = true;
  downloadPdfBtn.style.display = 'none';
  downloadLogBtn.style.display = 'none';
  setStatus('running', 'compiling…');
  logBox.classList.remove('visible');
  preview.style.display = 'none';
  placeholder.style.display = 'none';
  lastPdfB64 = null; lastLogText = null;

  const t0   = performance.now();
  const form = new FormData();
  form.append('source',    source);
  form.append('main_name', mainNameEl.value || 'doc.tex');

  // attach all support files (use in-memory content for text files)
  for (const [name, entry] of Object.entries(uploadedFiles)) {
    if (entry.content !== null) {
      // re-pack edited text content as a Blob
      const blob = new Blob([entry.content], { type: 'text/plain' });
      form.append('files', blob, name);
    } else {
      form.append('files', entry.file, name);
    }
  }

  try {
    const res  = await fetch('http://localhost:5000/compile', { method: 'POST', body: form });
    const data = await res.json();
    const elapsed = ((performance.now() - t0) / 1000).toFixed(2);
    compileTime.textContent = elapsed + 's';

    // ── always store log ──────────────────────────────
    if (data.log) {
      lastLogText = data.log;
      lastLogName = data.log_name || 'output.log';
      downloadLogBtn.style.display = 'inline-block';
    }

    if (data.success && data.pdf) {
      lastPdfB64  = data.pdf;
      lastPdfName = data.pdf_name || 'output.pdf';

      // convert base64 → blob → object URL
      const pdfBytes = Uint8Array.from(atob(data.pdf), c => c.charCodeAt(0));
      const pdfBlob  = new Blob([pdfBytes], { type: 'application/pdf' });
      preview.src    = URL.createObjectURL(pdfBlob) + '#toolbar=1&view=FitH';
      preview.style.display = 'block';
      setStatus('success', '✓ success');
      previewLabel.textContent = 'preview — ' + lastPdfName;
      downloadPdfBtn.style.display = 'inline-block';
    } else {
      logBox.textContent = data.log || data.error || 'Unknown error';
      logBox.classList.add('visible');
      setStatus('error', '✗ error');
      previewLabel.textContent = 'compile log';
    }

  } catch (err) {
    logBox.textContent = '⚠ Could not reach server at http://localhost:5000\n\nMake sure server.py is running:\n  pip install flask flask-cors\n  python server.py';
    logBox.classList.add('visible');
    setStatus('error', '✗ unreachable');
    previewLabel.textContent = 'error';
  }

  compileBtn.disabled = false;
}

// ── download PDF ──────────────────────────────────────────
downloadPdfBtn.addEventListener('click', () => {
  if (!lastPdfB64) return;
  const bytes = Uint8Array.from(atob(lastPdfB64), c => c.charCodeAt(0));
  triggerDownload(new Blob([bytes], { type: 'application/pdf' }), lastPdfName);
});

// ── download LOG ──────────────────────────────────────────
downloadLogBtn.addEventListener('click', () => {
  if (!lastLogText) return;
  triggerDownload(new Blob([lastLogText], { type: 'text/plain' }), lastLogName);
});

function triggerDownload(blob, filename) {
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = filename;
  a.click();
}

function setStatus(cls, text) {
  statusEl.className = 'status ' + cls;
  statusEl.textContent = text;
}
</script>
</body>
</html>
