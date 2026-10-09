
<!DOCTYPE html>
<html lang="bn">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="theme-color" content="#087f73">
<title>NR Electronic BD</title>
<style>
*{box-sizing:border-box}
body{margin:0;font-family:Arial,sans-serif;background:#f2f5f7;color:#17212b}
header{background:#087f73;color:white;padding:18px 12px;position:sticky;top:0;z-index:5}
.head{max-width:1100px;margin:auto;display:flex;justify-content:space-between;align-items:center;gap:10px}
h1{font-size:21px;margin:0}header p{font-size:12px;margin:5px 0 0}
main{max-width:1100px;margin:18px auto;padding:0 12px 35px}
button{border:0;border-radius:9px;padding:10px 12px;font-weight:bold;cursor:pointer}
.primary{background:#087f73;color:white}.secondary{background:#e1f3ef;color:#075e55}.danger{background:#ffebeb;color:#b91c1c}
input,textarea{width:100%;padding:11px;border:1px solid #d5dfe4;border-radius:9px;font:inherit}
.toolbar{display:flex;flex-wrap:wrap;gap:8px;margin-bottom:16px}
.toolbar input{flex:1;min-width:150px}
.products{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:12px}
.card{background:white;border-radius:14px;overflow:hidden;border:1px solid #e2e8ed;display:flex;flex-direction:column}
.photo{aspect-ratio:1;background:#e9eff1;display:flex;align-items:center;justify-content:center;font-size:40px}
.photo img{width:100%;height:100%;object-fit:cover}
.body{padding:11px;display:flex;flex-direction:column;flex:1}
.name{font-weight:bold;overflow-wrap:anywhere}
.desc{font-size:13px;color:#687680;margin:7px 0;white-space:pre-wrap;overflow-wrap:anywhere;flex:1}
.price{font-size:18px;color:#087f73;font-weight:bold;margin:5px 0 10px}
.actions{display:flex;flex-wrap:wrap;gap:5px}
.actions button{flex:1;padding:8px 4px;font-size:12px}
.panel{background:white;padding:16px;border-radius:14px;margin-top:16px}
.panel h2{font-size:19px;margin-top:0}
.field{margin:12px 0}.field label{display:block;font-weight:bold;font-size:14px;margin-bottom:6px}
.hidden{display:none!important}.hint{font-size:12px;color:#687680}
.total{text-align:right;font-size:20px;font-weight:bold;margin:15px 0}
.cartrow{padding:12px 0;border-bottom:1px solid #eee}
#preview img{max-width:150px;max-height:150px;object-fit:contain}
footer{text-align:center;color:#78858e;font-size:12px;padding:25px}
@media(max-width:800px){.products{grid-template-columns:repeat(3,minmax(0,1fr))}}
@media(max-width:540px){.products{grid-template-columns:repeat(2,minmax(0,1fr));gap:9px}h1{font-size:18px}main{padding:0 9px 30px}.body{padding:9px}.name{font-size:14px}.desc{font-size:12px}.price{font-size:16px}}
</style>
</head>
<body>
<header>
 <div class="head">
  <div><h1>⚡ NR Electronic BD</h1><p>আপনার বিশ্বস্ত ইলেকট্রনিক্স শপ</p></div>
  <button class="secondary" onclick="showCart()">🛒 কার্ট <span id="count">0</span></button>
 </div>
</header>

<main>
 <div class="toolbar">
  <input id="search" placeholder="🔎 পণ্য খুঁজুন..." oninput="render()">
  <button class="primary" onclick="openForm()">＋ পণ্য যোগ</button>
  <button class="secondary" onclick="backup()">Backup</button>
  <button class="secondary" onclick="document.getElementById('importFile').click()">Import</button>
  <input id="importFile" type="file" accept=".json,application/json" class="hidden" onchange="importData(event)">
 </div>

 <div id="products" class="products"></div>
 <div id="empty" class="panel hidden">কোনো পণ্য নেই। পণ্য যোগ করুন।</div>

 <section id="editor" class="panel hidden">
  <h2 id="formTitle">নতুন পণ্য যোগ করুন</h2>
  <form id="productForm" onsubmit="saveProduct(event)">
   <input id="pid" type="hidden">
   <div class="field"><label>পণ্যের নাম *</label><input id="pname" required maxlength="120" placeholder="যেমন: 12V LED লাইট"></div>
   <div class="field"><label>দাম (টাকা) *</label><input id="pprice" type="number" min="0" step="0.01" required placeholder="250"></div>
   <div class="field">
    <label>পণ্যের ছবি</label><input id="pimage" type="file" accept="image/*">
    <p class="hint">মোবাইলের গ্যালারি থেকে ছবি নির্বাচন করুন। সর্বোচ্চ ৬ MB।</p>
    <div id="preview"></div>
    <button type="button" class="danger" onclick="removeImage()">ছবি সরান</button>
   </div>
   <div class="field"><label>পণ্যের বিবরণ</label><textarea id="pdesc" rows="3" maxlength="1500" placeholder="পণ্যের বৈশিষ্ট্য লিখুন"></textarea></div>
   <button class="primary" type="submit">সংরক্ষণ করুন</button>
   <button class="secondary" type="button" onclick="closeForm()">বাতিল</button>
  </form>
 </section>

 <section id="cart" class="panel hidden">
  <h2>🛒 আপনার কার্ট</h2>
  <div id="cartItems"></div>
  <div class="total" id="total"></div>
  <button class="secondary" onclick="document.getElementById('cart').classList.add('hidden')">শপিং চালিয়ে যান</button>
  <button class="primary" onclick="checkout()">অর্ডার করুন</button>
 </section>

 <section id="checkout" class="panel hidden">
  <h2>📦 অর্ডারের তথ্য</h2>
  <form onsubmit="sendOrder(event)">
   <div class="field"><label>আপনার নাম *</label><input id="cname" required></div>
   <div class="field"><label>মোবাইল নম্বর *</label><input id="cphone" type="tel" required placeholder="01XXXXXXXXX"></div>
   <div class="field"><label>সম্পূর্ণ ঠিকানা *</label><textarea id="caddress" rows="3" required placeholder="গ্রাম, উপজেলা, জেলা"></textarea></div>
   <div class="field"><label>অতিরিক্ত তথ্য</label><textarea id="cnote" rows="2"></textarea></div>
   <p class="hint">WhatsApp খুললে মেসেজটি দেখে Send চাপুন।</p>
   <button class="primary" type="submit">WhatsApp-এ অর্ডার পাঠান</button>
   <button class="secondary" type="button" onclick="document.getElementById('checkout').classList.add('hidden')">বাতিল</button>
  </form>
 </section>

 <p class="hint">পণ্যের তথ্য এই ব্রাউজারে সংরক্ষিত হয়। নিয়মিত Backup ডাউনলোড করুন।</p>
 <footer>© NR Electronic BD</footer>
</main>

<script>
const KEY='nreb_products_v1';
const CARTKEY='nreb_cart_v1';
let products=[],cart={},currentImage='',removePic=false;

const $=id=>document.getElementById(id);
function load(){
 try{
  products=JSON.parse(localStorage.getItem(KEY)||'[]');
  cart=JSON.parse(localStorage.getItem(CARTKEY)||'{}');
  if(!Array.isArray(products))products=[];
 }catch(e){products=[];cart={}}
 render();updateCount();
}
function persist(){
 try{
  localStorage.setItem(KEY,JSON.stringify(products));
  localStorage.setItem(CARTKEY,JSON.stringify(cart));
 }catch(e){alert('স্টোরেজ পূর্ণ! ছবির সাইজ কমান অথবা Backup নিন।')}
}
function money(n){return '৳'+Number(n||0).toLocaleString('en-BD',{maximumFractionDigits:2})}
function esc(s){
 return String(s??'').replace(/[&<>"']/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
}
function render(){
 const q=$('search').value.toLowerCase().trim();
 const list=products.filter(p=>(p.name+' '+p.desc).toLowerCase().includes(q));
 $('products').innerHTML=list.map(p=>`
 <article class="card">
  <div class="photo">${p.image?`<img src="${p.image}" alt="${esc(p.name)}">`:'📦'}</div>
  <div class="body">
   <div class="name">${esc(p.name)}</div>
   <div class="desc">${esc(p.desc||'বিবরণ দেওয়া হয়নি')}</div>
   <div class="price">${money(p.price)}</div>
   <div class="actions">
    <button class="primary" onclick="addCart('${p.id}')">কার্টে নিন</button>
    <button class="secondary" onclick="openForm('${p.id}')">এডিট</button>
    <button class="danger" onclick="delProduct('${p.id}')">ডিলিট</button>
   </div>
  </div>
 </article>`).join('');
 $('empty').classList.toggle('hidden',list.length!==0);
}
function openForm(id=''){
 const p=products.find(x=>x.id===id);
 $('productForm').reset();
 $('pid').value=p?p.id:'';
 $('pname').value=p?p.name:'';
 $('pprice').value=p?p.price:'';
 $('pdesc').value=p?p.desc:'';
 currentImage=p?p.image:'';
 removePic=false;
 $('formTitle').textContent=p?'পণ্য এডিট করুন':'নতুন পণ্য যোগ করুন';
 preview();
 $('editor').classList.remove('hidden');
 $('editor').scrollIntoView({behavior:'smooth'});
}
function closeForm(){$('editor').classList.add('hidden')}
function preview(){
 $('preview').innerHTML=currentImage&&!removePic?`<img src="${currentImage}" alt="ছবির প্রিভিউ">`:'';
}
function removeImage(){currentImage='';removePic=true;$('pimage').value='';preview()}
$('pimage').addEventListener('change',e=>{
 const f=e.target.files[0];if(!f)return;
 if(!f.type.startsWith('image/')){alert('ছবি নির্বাচন করুন');return}
 if(f.size>6*1024*1024){alert('৬ MB-এর চেয়ে ছোট ছবি নির্বাচন করুন');e.target.value='';return}
 const r=new FileReader();
 r.onload=()=>{
  const img=new Image();
  img.onload=()=>{
   const c=document.createElement('canvas'),max=900;
   let w=img.width,h=img.height;
   if(w>h&&w>max){h=Math.round(h*max/w);w=max}
   else if(h>max){w=Math.round(w*max/h);h=max}
   c.width=w;c.height=h;c.getContext('2d').drawImage(img,0,0,w,h);
   currentImage=c.toDataURL('image/jpeg',.75);removePic=false;preview();
  };
  img.onerror=()=>alert('ছবিটি খোলা যায়নি');
  img.src=r.result;
 };
 r.readAsDataURL(f);
});
function saveProduct(e){
 e.preventDefault();
 const id=$('pid').value,old=products.find(p=>p.id===id);
 const name=$('pname').value.trim(),price=Number($('pprice').value);
 if(!name||!Number.isFinite(price)||price<0){alert('সঠিক নাম ও দাম দিন');return}
 const p={id:id||('p'+Date.now()),name,price,desc:$('pdesc').value.trim(),image:removePic?'':(currentImage||(old?old.image:''))};
 if(old)products=products.map(x=>x.id===id?p:x);else products.unshift(p);
 persist();render();closeForm();
}
function delProduct(id){
 const p=products.find(x=>x.id===id);
 if(!p||!confirm(p.name+' ডিলিট করবেন?'))return;
 products=products.filter(x=>x.id!==id);delete cart[id];
 persist();render();renderCart();updateCount();
}
function addCart(id){
 cart[id]=(cart[id]||0)+1;persist();updateCount();showCart();
}
function updateCount(){$('count').textContent=Object.values(cart).reduce((a,b)=>a+Number(b),0)}
function showCart(){
 $('cart').classList.remove('hidden');renderCart();
 $('cart').scrollIntoView({behavior:'smooth'});
}
function renderCart(){
 const entries=Object.entries(cart).filter(([id,q])=>q>0&&products.some(p=>p.id===id));
 $('cartItems').innerHTML=entries.length?entries.map(([id,q])=>{
  const p=products.find(x=>x.id===id);
  return `<div class="cartrow"><b>${esc(p.name)}</b><br>${money(p.price)} × ${q} = <b>${money(p.price*q)}</b><br>
  <button class="secondary" onclick="changeQty('${id}',-1)">−</button>
  <button class="secondary" onclick="changeQty('${id}',1)">＋</button>
  <button class="danger" onclick="changeQty('${id}',-${q})">সরান</button></div>`;
 }).join(''):'কার্ট খালি';
 const total=entries.reduce((s,[id,q])=>s+products.find(p=>p.id===id).price*q,0);
 $('total').textContent='মোট: '+money(total);
}
function changeQty(id,n){
 cart[id]=Math.max(0,(cart[id]||0)+n);
 if(!cart[id])delete cart[id];
 persist();updateCount();renderCart();
}
function checkout(){
 if(!Object.values(cart).some(q=>q>0)){alert('কার্টে পণ্য যোগ করুন');return}
 $('checkout').classList.remove('hidden');
 $('checkout').scrollIntoView({behavior:'smooth'});
}
function sendOrder(e){
 e.preventDefault();
 const items=Object.entries(cart).filter(([id,q])=>q>0&&products.some(p=>p.id===id));
 if(!items.length){alert('কার্ট খালি');return}
 const total=items.reduce((s,[id,q])=>s+products.find(p=>p.id===id).price*q,0);
 const lines=items.map(([id,q])=>{
  const p=products.find(x=>x.id===id);
  return `• ${p.name} — ${q} × ${money(p.price)} = ${money(p.price*q)}`;
 });
 const msg=`আসসালামু আলাইকুম, NR Electronic BD থেকে অর্ডার করতে চাই।\n\n${lines.join('\n')}\n\nমোট: ${money(total)}\nনাম: ${$('cname').value}\nমোবাইল: ${$('cphone').value}\nঠিকানা: ${$('caddress').value}\nঅতিরিক্ত তথ্য: ${$('cnote').value||'নেই'}`;
 window.open('https://wa.me/8801740116023?text='+encodeURIComponent(msg),'_blank');
}
function backup(){
 const data={app:'NR Electronic BD',products};
 const blob=new Blob([JSON.stringify(data,null,2)],{type:'application/json'});
 const url=URL.createObjectURL(blob),a=document.createElement('a');
 a.href=url;a.download='nr-electronic-bd-backup.json';a.click();
 URL.revokeObjectURL(url);
}
function importData(e){
 const f=e.target.files[0];if(!f)return;
 const r=new FileReader();
 r.onload=()=>{
  try{
   const data=JSON.parse(r.result);
   const list=Array.isArray(data)?data:data.products;
   if(!Array.isArray(list))throw Error('ফাইল সঠিক নয়');
   const valid=list.filter(p=>p.name!==undefined&&Number.isFinite(Number(p.price))&&Number(p.price)>=0);
   if(!valid.length)throw Error('কোনো বৈধ পণ্য পাওয়া যায়নি');
   if(!confirm(valid.length+'টি পণ্য বর্তমান তালিকার সঙ্গে যোগ করবেন?'))return;
   valid.forEach((p,i)=>products.push({
    id:'imp'+Date.now()+i,
    name:String(p.name),price:Number(p.price),
    desc:String(p.desc||''),image:String(p.image||'')
   }));
   persist();render();alert('Import সম্পন্ন হয়েছে');
  }catch(err){alert('Import করা যায়নি: '+err.message)}
  finally{e.target.value=''}
 };
 r.readAsText(f);
}
load();
</script>
</body>
</html>
