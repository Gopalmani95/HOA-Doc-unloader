# HOA-Doc-unloader<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>HOA Document Processor — Interface Builder</title>
  <style>
    :root{--bg:#f6f8fb;--panel:#ffffff;--accent:#0b69ff;--muted:#6b7280}
    body{font-family:Inter,system-ui,Segoe UI,Roboto,'Helvetica Neue',Arial;margin:0;background:var(--bg);color:#111}
    .app{display:flex;min-height:100vh}
    .left{width:280px;padding:18px;background:linear-gradient(180deg,#fff,#fbfdff);box-shadow:2px 0 6px rgba(11,105,255,0.04);border-right:1px solid #eef2f7}
    .brand{font-weight:700;margin-bottom:14px}
    .btn{display:inline-flex;align-items:center;gap:8px;padding:10px 12px;border-radius:8px;border:0;background:var(--accent);color:#fff;cursor:pointer}
    .canvas-wrap{flex:1;padding:18px;}
    .canvas{height:78vh;border:2px dashed #e6eefc;border-radius:10px;padding:12px;background:linear-gradient(180deg,#fbfdff, #ffffff)}
    .node{width:280px;padding:12px;border-radius:8px;background:var(--panel);box-shadow:0 2px 6px rgba(2,6,23,0.06);position:relative;margin:12px}
    .node .title{font-weight:600}
    .node .meta{font-size:13px;color:var(--muted);margin-top:6px}
    .node .edit-btn{position:absolute;top:8px;right:8px;padding:6px 8px;border-radius:6px;border:0;background:#f1f5f9;color:#0b69ff;cursor:pointer;display:none}
    .node:hover .edit-btn{display:inline-flex}
    .controls{display:flex;gap:8px;align-items:center;margin-bottom:12px}
    .modal-backdrop{position:fixed;inset:0;background:rgba(2,6,23,0.35);display:flex;align-items:center;justify-content:center}
    .modal{width:720px;background:var(--panel);padding:18px;border-radius:10px;box-shadow:0 8px 30px rgba(2,6,23,0.12)}
    .form-row{display:flex;gap:10px;margin-bottom:10px}
    label{display:block;font-weight:600;margin-bottom:6px}
    input[type="text"],textarea,select{width:100%;padding:10px;border-radius:8px;border:1px solid #e6eefc;background:#fbfdff}
    textarea{min-height:84px}
    .small{font-size:13px;color:var(--muted)}
    .preview{border:1px dashed #e9eef8;padding:12px;border-radius:8px;background:#fbfdff}
    .open-interface{margin-top:8px;padding:8px 10px;border-radius:8px;border:0;background:#eef6ff;color:var(--accent);cursor:pointer}
    .uploader{margin-top:10px;padding:12px;border-radius:8px;border:1px dashed #dbeafe;background:#fff}
    .file-info{margin-top:8px;font-size:14px}
    .chip{display:inline-block;padding:6px 8px;border-radius:999px;background:#eef2ff;color:var(--accent);font-weight:600}
  </style>
</head>
<body>
  <div class="app">
    <aside class="left">
      <div class="brand">HOA Toolkit — Interface Builder</div>
      <div class="controls">
        <button id="add-interface" class="btn">➕ Add Interface</button>
      </div>
      <p class="small">Click <strong>Add Interface</strong> to create a new interface node on the canvas. Hover a node, then click <strong>Edit Interface</strong> to configure it.</p>
      <hr style="margin:14px 0;border:none;border-top:1px solid #eef2f7" />
      <div class="small"><strong>Saved Interfaces</strong></div>
      <div id="saved-list" style="margin-top:10px"></div>
    </aside>

    <main class="canvas-wrap">
      <h2 style="margin-top:0">Canvas</h2>
      <div class="canvas" id="canvas" aria-label="Design canvas">
        <!-- Nodes will appear here -->
      </div>
    </main>
  </div>

  <!-- Modal for editing interface -->
  <div id="modal" style="display:none" class="modal-backdrop">
    <div class="modal" role="dialog" aria-modal="true">
      <h3 id="modal-title">Edit Interface</h3>
      <div style="display:flex;gap:12px;margin-top:12px">
        <div style="flex:1">
          <label for="iface-title">Title</label>
          <input id="iface-title" type="text" placeholder="HOA Document Processor" />
        </div>
        <div style="width:120px">
          <label for="iface-icon">Icon</label>
          <select id="iface-icon">
            <option value="🏠">🏠 House</option>
            <option value="📄">📄 Document</option>
            <option value="🗂️">🗂️ Folder</option>
          </select>
        </div>
      </div>

      <div style="margin-top:12px">
        <label for="iface-desc">Description</label>
        <textarea id="iface-desc" placeholder="Upload your HOA PDF document to automatically extract key information and post to Slack"></textarea>
      </div>

      <div style="margin-top:12px;display:flex;gap:12px;align-items:center">
        <input id="add-upload" type="checkbox" checked /> <label for="add-upload" class="small">Add file upload field</label>
        <div class="small" style="margin-left:auto">Connect to input name: <code class="chip">pdf_file</code></div>
      </div>

      <div style="display:flex;gap:8px;justify-content:flex-end;margin-top:14px">
        <button id="cancel-modal" style="padding:8px 12px;border-radius:8px;border:1px solid #e6eefc;background:transparent;cursor:pointer">Cancel</button>
        <button id="save-modal" class="btn">Save Interface</button>
      </div>

      <hr style="margin:12px 0;border:none;border-top:1px solid #eef2f7" />
      <div>
        <strong>Preview</strong>
        <div class="preview" id="preview-area" aria-live="polite" style="margin-top:8px">
          <div style="font-weight:700">HOA Document Processor</div>
          <div class="small">Upload your HOA PDF document to automatically extract key information and post to Slack</div>
          <div style="margin-top:8px">
            <button id="open-preview" class="open-interface">Open Interface</button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Interface runtime panel (when user opens the configured interface) -->
  <div id="runtime" style="display:none;position:fixed;right:24px;bottom:24px;width:380px;background:var(--panel);padding:14px;border-radius:10px;box-shadow:0 10px 30px rgba(2,6,23,0.12)}">
    <div id="runtime-content"></div>
  </div>

  <script>
    // Simple in-memory store for created interfaces
    const canvas = document.getElementById('canvas');
    const addBtn = document.getElementById('add-interface');
    const modal = document.getElementById('modal');
    const ifaceTitle = document.getElementById('iface-title');
    const ifaceDesc = document.getElementById('iface-desc');
    const ifaceIcon = document.getElementById('iface-icon');
    const addUpload = document.getElementById('add-upload');
    const saveModal = document.getElementById('save-modal');
    const cancelModal = document.getElementById('cancel-modal');
    const previewArea = document.getElementById('preview-area');
    const openPreview = document.getElementById('open-preview');
    const savedList = document.getElementById('saved-list');

    let interfaces = []; // {id,title,desc,icon,hasUpload}
    let editingId = null;

    function createNode(obj){
      const node = document.createElement('div');
      node.className = 'node';
      node.dataset.id = obj.id;
      node.innerHTML = `
        <div style="display:flex;align-items:center;gap:10px">
          <div style="font-size:20px">${obj.icon}</div>
          <div style="flex:1">
            <div class="title">${obj.title}</div>
            <div class="meta">${obj.desc}</div>
          </div>
        </div>
        <button class="edit-btn">Edit Interface</button>
      `;
      const editBtn = node.querySelector('.edit-btn');
      editBtn.addEventListener('click', ()=>openEditor(obj.id));
      node.addEventListener('dblclick', ()=>openInterface(obj.id));
      canvas.appendChild(node);
      renderSavedList();
    }

    function renderSavedList(){
      savedList.innerHTML = '';
      interfaces.forEach(i=>{
        const d = document.createElement('div');
        d.style.display='flex';d.style.justifyContent='space-between';d.style.alignItems='center';d.style.padding='6px 0';
        d.innerHTML = `
          <div style="display:flex;gap:8px;align-items:center"><div>${i.icon}</div><div>${i.title}</div></div>
          <div>
            <button style="margin-right:8px" onclick="openInterface('${i.id}')">Open</button>
            <button onclick="openEditor('${i.id}')">Edit</button>
          </div>
        `;
        savedList.appendChild(d);
      });
    }

    function openEditor(id){
      editingId = id;
      const iface = interfaces.find(x=>x.id===id);
      ifaceTitle.value = iface.title;
      ifaceDesc.value = iface.desc;
      ifaceIcon.value = iface.icon;
      addUpload.checked = iface.hasUpload;
      previewArea.querySelector('div').textContent = iface.title;
      previewArea.querySelector('.small').textContent = iface.desc;
      modal.style.display = 'flex';
    }

    function openInterface(id){
      // Show runtime upload UI
      const iface = interfaces.find(x=>x.id===id);
      const runtime = document.getElementById('runtime');
      const runtimeContent = document.getElementById('runtime-content');
      runtimeContent.innerHTML = '';
      const el = document.createElement('div');
      el.innerHTML = `
        <div style="display:flex;align-items:center;gap:10px;margin-bottom:8px">
          <div style="font-size:20px">${iface.icon}</div>
          <div style="flex:1">
            <div style="font-weight:700">${iface.title}</div>
            <div class="small">${iface.desc}</div>
          </div>
        </div>
      `;
      if(iface.hasUpload){
        const uploader = document.createElement('div');
        uploader.className = 'uploader';
        uploader.innerHTML = `
          <label for="runtime-file-${iface.id}"><strong>Upload HOA PDF</strong></label>
          <input id="runtime-file-${iface.id}" name="pdf_file" type="file" accept="application/pdf" style="display:block;margin-top:8px" />
          <div class="file-info" id="file-info-${iface.id}"></div>
          <div style="margin-top:10px;display:flex;gap:8px;justify-content:flex-end">
            <button id="cancel-runtime-${iface.id}" style="padding:8px 10px;border-radius:8px;border:1px solid #e6eefc;background:transparent;cursor:pointer">Close</button>
            <button id="upload-runtime-${iface.id}" class="btn">Upload</button>
          </div>
        `;
        el.appendChild(uploader);
      } else {
        el.innerHTML += `<div class="small" style="margin-top:10px">This interface does not have a file upload field configured.</div>`;
      }

      runtimeContent.appendChild(el);
      runtime.style.display = 'block';

      // wire file input actions
      const fileInput = document.getElementById(`runtime-file-${iface.id}`);
      const fileInfo = document.getElementById(`file-info-${iface.id}`);
      if(fileInput){
        fileInput.addEventListener('change', (ev)=>{
          const f = ev.target.files[0];
          if(!f){ fileInfo.textContent = ''; return; }
          if(f.type !== 'application/pdf'){
            fileInfo.textContent = 'Please select a PDF file only.';
            fileInput.value = '';
            return;
          }
          fileInfo.textContent = `Selected: ${f.name} (${Math.round(f.size/1024)} KB)`;
        });

        document.getElementById(`upload-runtime-${iface.id}`).addEventListener('click', ()=>{
          const f = fileInput.files[0];
          if(!f){ fileInfo.textContent = 'Please choose a PDF to upload.'; return; }
          // For demo: we simply show the file in saved area. In real app, you would POST this file to your server or API.
          fileInfo.textContent = 'Upload complete — file ready to be processed: ' + f.name;
          // Example: create a FormData and POST to /upload endpoint (not implemented in this static demo)
          // const fd = new FormData(); fd.append('pdf_file', f);
          // fetch('/upload', {method:'POST', body: fd})...
        });

        document.getElementById(`cancel-runtime-${iface.id}`).addEventListener('click', ()=>{
          document.getElementById('runtime').style.display = 'none';
        });
      } else {
        // close button for no-upload case
        const btn = runtimeContent.querySelector('button');
        if(btn) btn.addEventListener('click', ()=>{ document.getElementById('runtime').style.display = 'none'; });
      }
    }

    addBtn.addEventListener('click', ()=>{
      // create default interface object
      const id = 'iface_' + Math.random().toString(36).slice(2,9);
      const obj = {id,title:'HOA Document Processor',desc:'Upload your HOA PDF document to automatically extract key information and post to Slack',icon:'🏠',hasUpload:true};
      interfaces.push(obj);
      createNode(obj);
    });

    cancelModal.addEventListener('click', ()=>{ modal.style.display='none'; editingId=null });

    saveModal.addEventListener('click', ()=>{
      const title = ifaceTitle.value.trim() || 'HOA Document Processor';
      const desc = ifaceDesc.value.trim() || 'Upload your HOA PDF document to automatically extract key information and post to Slack';
      const icon = ifaceIcon.value;
      const hasUpload = addUpload.checked;
      if(!editingId){ modal.style.display='none'; return; }
      const idx = interfaces.findIndex(x=>x.id===editingId);
      interfaces[idx].title = title;
      interfaces[idx].desc = desc;
      interfaces[idx].icon = icon;
      interfaces[idx].hasUpload = hasUpload;

      // update node on canvas
      const node = canvas.querySelector(`[data-id='${editingId}']`);
      if(node){
        node.querySelector('.title').textContent = title;
        node.querySelector('.meta').textContent = desc;
        node.querySelector('div').firstElementChild.textContent = icon;
      }

      // update preview
      previewArea.querySelector('div').textContent = title;
      previewArea.querySelector('.small').textContent = desc;

      modal.style.display = 'none';
      editingId = null;
      renderSavedList();
    });

    // expose some functions for inline onclick handlers
    window.openEditor = (id)=>openEditor(id);
    window.openInterface = (id)=>openInterface(id);

    // initial sample node
    addBtn.click();
  </script>
</body>
</html>
