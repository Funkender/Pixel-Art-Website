# Pixel-Art-Website
This is a simple test pixel art website with like system 

<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <title>Pixel64 Plattform</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root { --scale: 8; }
    body { background:#121212; color:#eaeaea; font-family:sans-serif; margin:0; padding:20px; }
    h1 { text-align:center; }
    nav { display:flex; gap:12px; justify-content:center; margin-bottom:20px; }
    nav button { background:#1f1f1f; color:#fff; border:1px solid #333; padding:8px 12px; border-radius:6px; cursor:pointer; }
    nav button.active { background:#333; }
    .toolbar { display:flex; gap:12px; justify-content:center; margin-bottom:12px; flex-wrap:wrap; }
    #canvasWrapper { margin:0 auto; width:calc(64*var(--scale)*1px); height:calc(64*var(--scale)*1px); border:2px solid #333; background:#1a1a1a; position:relative; }
    canvas { width:100%; height:100%; image-rendering:pixelated; display:block; }
    .grid { position:absolute; inset:0; pointer-events:none;
      background-image:linear-gradient(to right,rgba(255,255,255,0.06) 1px,transparent 1px),
                       linear-gradient(to bottom,rgba(255,255,255,0.06) 1px,transparent 1px);
      background-size:calc(var(--scale)*1px) calc(var(--scale)*1px); mix-blend-mode:screen; }
    .gallery { display:grid; grid-template-columns:repeat(auto-fill,minmax(120px,1fr)); gap:12px; }
    .gallery img { width:100%; image-rendering:pixelated; border:1px solid #333; background:#222; }
    .note { text-align:center; font-size:12px; opacity:0.7; margin-top:8px; }
  </style>
</head>
<body>
  <h1>Pixel64 Plattform</h1>
  <nav>
    <button id="nav-editor" class="active">Editor</button>
    <button id="nav-uploads">Uploads</button>
    <button id="nav-my">Meine Bilder</button>
  </nav>

  <!-- Editor -->
  <section id="page-editor">
    <div class="toolbar">
      <label>Farbe <input type="color" id="color" value="#ff4d4d"></label>
      <label>Zoom <input type="range" id="scale" min="4" max="16" value="8"></label>
      <button id="clear">Leeren</button>
      <button id="download">Download</button>
      <button id="upload">Upload</button>
    </div>
    <div id="canvasWrapper">
      <canvas id="canvas" width="64" height="64"></canvas>
      <div class="grid"></div>
    </div>
    <div class="note">Linksklick: malen • Rechtsklick: radieren</div>
  </section>

  <!-- Uploads -->
  <section id="page-uploads" style="display:none">
    <h2>Alle Uploads</h2>
    <div class="gallery" id="gallery-uploads"></div>
  </section>

  <!-- Meine Bilder -->
  <section id="page-my" style="display:none">
    <h2>Meine Bilder</h2>
    <div class="gallery" id="gallery-my"></div>
  </section>

  <script>
    const canvas=document.getElementById('canvas');
    const ctx=canvas.getContext('2d',{willReadFrequently:true});
    const color=document.getElementById('color');
    const scale=document.getElementById('scale');
    const clearBtn=document.getElementById('clear');
    const downloadBtn=document.getElementById('download');
    const uploadBtn=document.getElementById('upload');
    let drawing=false;

    canvas.addEventListener('contextmenu',e=>e.preventDefault());
    function getPixel(e){
      const rect=canvas.getBoundingClientRect();
      const x=Math.floor((e.clientX-rect.left)*canvas.width/rect.width);
      const y=Math.floor((e.clientY-rect.top)*canvas.height/rect.height);
      return {x:Math.max(0,Math.min(63,x)),y:Math.max(0,Math.min(63,y))};
    }
    function paint(x,y){ctx.fillStyle=color.value;ctx.fillRect(x,y,1,1);}
    function erase(x,y){ctx.clearRect(x,y,1,1);}
    function drawHandler(e){const {x,y}=getPixel(e);const right=e.button===2||e.buttons===2;right?erase(x,y):paint(x,y);}
    canvas.addEventListener('mousedown',e=>{drawing=true;drawHandler(e);});
    window.addEventListener('mouseup',()=>drawing=false);
    canvas.addEventListener('mousemove',e=>{if(drawing)drawHandler(e);});
    scale.addEventListener('input',e=>document.documentElement.style.setProperty('--scale',e.target.value));
    clearBtn.addEventListener('click',()=>ctx.clearRect(0,0,64,64));
    downloadBtn.addEventListener('click',()=>{
      const a=document.createElement('a');
      a.download='pixel64.png';
      a.href=canvas.toDataURL('image/png');
      a.click();
    });

    // Uploads speichern in LocalStorage
    function saveUpload(mine=false){
      const data=canvas.toDataURL('image/png');
      const uploads=JSON.parse(localStorage.getItem('uploads')||'[]');
      uploads.push({src:data,mine});
      localStorage.setItem('uploads',JSON.stringify(uploads));
      alert("Bild hochgeladen!");
    }
    uploadBtn.addEventListener('click',()=>saveUpload(true));

    // Navigation
    const pages={editor:document.getElementById('page-editor'),
                 uploads:document.getElementById('page-uploads'),
                 my:document.getElementById('page-my')};
    function showPage(name){
      for(const key in pages){pages[key].style.display=(key===name)?'block':'none';}
      document.querySelectorAll('nav button').forEach(b=>b.classList.remove('active'));
      document.getElementById('nav-'+name).classList.add('active');
      if(name!=='editor') renderGallery(name);
    }
    document.getElementById('nav-editor').onclick=()=>showPage('editor');
    document.getElementById('nav-uploads').onclick=()=>showPage('uploads');
    document.getElementById('nav-my').onclick=()=>showPage('my');

    // Galerie rendern
    function renderGallery(name){
      const uploads=JSON.parse(localStorage.getItem('uploads')||'[]');
      const container=document.getElementById('gallery-'+name);
      container.innerHTML='';
      uploads.filter(u=>name==='uploads'||u.mine).forEach(u=>{
        const img=document.createElement('img');
        img.src=u.src;
        container.appendChild(img);
      });
    }
  </script>
</body>
</html>
