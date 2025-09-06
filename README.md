<!doctype html>
<html lang="uk">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Генератор та пошук зображень</title>
  <style>
    :root{
      --bg:#0b0f14; --panel:#121923; --muted:#8aa0b7; --text:#e8eef5; --accent:#4fa3ff; --accent-2:#7bd389; --danger:#ff6b6b;
    }
    *{box-sizing:border-box}
    body{margin:0;font-family:system-ui,-apple-system,Segoe UI,Roboto,Ubuntu,Inter,Arial,sans-serif;background:#0b0f14;color:var(--text);}
    .wrap{max-width:1100px;margin:0 auto;padding:24px}
    .title{font-size:1.8rem;font-weight:800;margin-bottom:16px}
    .card{background:#121923;border:1px solid #1c2a3a;border-radius:14px;padding:16px;margin-bottom:16px}
    .controls{display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:12px}
    .controls label{display:block;font-size:.9rem;color:var(--muted);margin-bottom:4px}
    .controls input, .controls select{width:100%;padding:8px;border-radius:10px;border:1px solid #253445;background:#0f1722;color:var(--text)}
    .btns{display:flex;flex-wrap:wrap;gap:10px;margin-top:10px}
    button{border:none;border-radius:12px;padding:10px 14px;font-weight:600;cursor:pointer;background:#2b6cb0;color:#fff}
    button:hover{filter:brightness(1.1)}
    .success{background:#2f855a}
    .danger{background:#c53030}
    .gallery{margin-top:16px;display:grid;grid-template-columns:repeat(auto-fill,minmax(220px,1fr));gap:12px}
    .gallery img{width:100%;border-radius:10px;object-fit:cover;cursor:pointer;transition:transform .2s}
    .gallery img:hover{transform:scale(1.03)}
    .slideshow{position:fixed;inset:0;background:#000;display:none;align-items:center;justify-content:center;z-index:1000}
    .slideshow img{max-width:90%;max-height:90%;object-fit:contain;animation:fade 1s ease}
    @keyframes fade{from{opacity:0}to{opacity:1}}
  </style>
</head>
<body>
<div class="wrap">
  <div class="title">🖼️ Генератор та пошук зображень</div>

  <!-- Блок генератора випадкових зображень (picsum.photos) -->
  <div class="card">
    <h3>🎲 Генератор випадкових зображень (picsum.photos)</h3>
    <div class="controls">
      <div><label>Ширина</label><input id="w" type="number" value="800"></div>
      <div><label>Висота</label><input id="h" type="number" value="500"></div>
      <div><label>Seed</label><input id="seed" type="text"></div>
      <div><label>Розмиття</label><select id="blur"><option value="0">немає</option><option value="1">легке</option><option value="2">сильніше</option></select></div>
      <div><label><input type="checkbox" id="gray"> ч/б</label></div>
    </div>
    <div class="btns">
      <button id="btnGen">Згенерувати</button>
    </div>
    <div id="viewer"><img id="img" style="max-width:100%;margin-top:12px"></div>
  </div>

  <!-- Блок пошуку Unsplash -->
  <div class="card">
    <h3>🔍 Пошук картинок (Unsplash)</h3>
    <div class="controls">
      <div><label>Запит</label><input id="searchQuery" type="text" placeholder="cats, nature..."></div>
      <div><label>Кількість</label><input id="searchCount" type="number" value="9" min="1" max="30"></div>
      <div><label>Орієнтація</label>
        <select id="searchOrientation">
          <option value="">Будь-яка</option>
          <option value="landscape">Альбом</option>
          <option value="portrait">Портрет</option>
          <option value="squarish">Квадрат</option>
        </select>
      </div>
    </div>
    <div class="btns">
      <button id="btnSearch">Шукати</button>
    </div>
    <div id="searchResults" class="gallery"></div>
  </div>

  <!-- Слайдшоу -->
  <div class="slideshow" id="slideshow"><img id="slideImg"></div>
</div>

<script>
  const ACCESS_KEY = "fwmLZx3MGd5WANwV52SDUacny3E1dLFuL67rs693NvU";

  // генератор picsum
  function buildPicsum(){
    const w=document.getElementById('w').value||800;
    const h=document.getElementById('h').value||600;
    const seed=document.getElementById('seed').value.trim();
    const blur=document.getElementById('blur').value;
    const gray=document.getElementById('gray').checked;
    let url= seed?`https://picsum.photos/seed/${seed}/${w}/${h}`:`https://picsum.photos/${w}/${h}`;
    const q=[];
    if(gray) q.push('grayscale');
    if(blur>0) q.push('blur='+blur);
    if(!seed) q.push('rand='+Date.now());
    if(q.length) url+="?"+q.join("&");
    return url;
  }
  document.getElementById('btnGen').onclick=()=>{
    const url=buildPicsum();
    document.getElementById('img').src=url;
  };

  // пошук Unsplash
  async function searchUnsplash(){
    const query=document.getElementById('searchQuery').value.trim();
    const count=Math.min(30,Math.max(1,+document.getElementById('searchCount').value||9));
    const orientation=document.getElementById('searchOrientation').value;
    let url=`https://api.unsplash.com/search/photos?page=1&query=${encodeURIComponent(query)}&per_page=${count}&client_id=${ACCESS_KEY}`;
    if(orientation){
      url+=`&orientation=${orientation}`;
    }
    const res=await fetch(url);
    const data=await res.json();
    const results=document.getElementById('searchResults');
    results.innerHTML="";
    if(data.results && data.results.length){
      data.results.forEach(r=>{
        const img=document.createElement('img');
        img.src=r.urls.small;
        img.alt=r.alt_description||"unsplash";
        img.onclick=()=>window.open(r.urls.full,'_blank');
        results.appendChild(img);
      });
    } else {
      results.innerHTML="<p>Нічого не знайдено</p>";
    }
  }
  document.getElementById('btnSearch').onclick=searchUnsplash;
</script>
</body>
</html>
