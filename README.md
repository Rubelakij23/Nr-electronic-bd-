
<!DOCTYPE html>
<html lang="bn">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>NR Electronic BD - Online Shop</title>
<meta name="theme-color" content="#087f73">

<style>
*{box-sizing:border-box}
body{margin:0;font-family:Arial,"Noto Sans Bengali",sans-serif;background:#f1f6f8;color:#263238}
header{background:linear-gradient(120deg,#08786f,#109c8d);color:#fff;padding:20px 15px;display:flex;justify-content:space-between;align-items:center;gap:10px}
header h1{font-size:23px;margin:0 0 7px}
header p{margin:0;font-size:14px}
button,input,textarea{font:inherit}
button{cursor:pointer;border:0;border-radius:10px;padding:12px}
.cartbtn{background:#ffffff25;color:white;border:1px solid #ffffff70}
main{max-width:1100px;margin:auto;padding:16px}
.searchrow{display:flex;gap:10px;margin-bottom:18px}
.searchrow input{min-width:0;flex:1;padding:13px;border:1px solid #ccd5da;border-radius:12px;background:white}
.refresh{background:#dff3f0;color:#086d64;font-weight:bold}
#status{color:#64747b;margin:18px 0;line-height:1.6;overflow-wrap:anywhere}
.products{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:13px}
.card{background:white;border-radius:14px;overflow:hidden;box-shadow:0 2px 9px #00000010}
.card img{width:100%;height:155px;object-fit:contain;background:#fff}
.info{padding:12px}
.info h3{font-size:16px;margin:0 0 8px;overflow-wrap:anywhere}
.price{color:#078276;font-weight:bold;font-size:18px;margin:7px 0}
.desc{font-size:13px;color:#69777d;line-height:1.5;overflow-wrap:anywhere}
.add{width:100%;background:#087d72;color:white;margin-top:10px;font-weight:bold}
#cartbox{display:none;background:white;border-radius:14px;padding:16px;margin:18px 0;box-shadow:0 2px 9px #00000010}
#cartitems p{border-bottom:1px solid #eee;padding:9px 0;line-height:1.8;overflow-wrap:anywhere}
#cartbox input,#cartbox textarea{display:block;width:100%;padding:12px;margin:10px 0;border:1px solid #ccd5da;border-radius:9px}
.order{background:#16854c;color:white;width:100%;font-weight:bold}
.empty{background:#fff;padding:18px;border-radius:12px;color:#66757c}
footer{text-align:center;color:#68777e;padding:25px 10px;font-size:13px}
@media(min-width:650px){.products{grid-template-columns:repeat(4,minmax(0,1fr))}.card img{height:190px}}
</style>
</head>

<body>
<header>
  <div>
    <h1>⚡ NR Electronic BD</h1>
    <p>আপনার অনলাইন ইলেকট্রনিক্স শপ</p>
  </div>
  <button class="cartbtn" onclick="toggleCart()">🛒 কার্ট <span id="cartcount">0</span></button>
</header>

<main>
  <div class="searchrow">
    <input id="search" type="search" placeholder="🔎 পণ্যের নাম দিয়ে খুঁজুন" oninput="renderProducts()">
    <button class="refresh" onclick="loadProducts()">↻ পণ্য আপডেট</button>
  </div>

  <div id="status">অনলাইন পণ্যের তালিকা লোড হচ্ছে...</div>
  <section id="products" class="products"></section>

  <section id="cartbox">
    <h2>🛒 আপনার কার্ট</h2>
    <div id="cartitems"></div>
    <h3 id="total">মোট: ৳০</h3>

    <input id="customer" placeholder="আপনার নাম">
    <input id="phone" type="tel" placeholder="মোবাইল নম্বর">
    <textarea id="address" rows="3" placeholder="সম্পূর্ণ ঠিকানা"></textarea>

    <button class="order" onclick="sendOrder()">WhatsApp-এ অর্ডার করুন</button>
  </section>
</main>

<footer>© NR Electronic BD · Made for mobile and desktop</footer>

<script>
/* Supabase connection */
const SUPABASE_URL = "https://klevjsjrvygbfjwotgwx.supabase.co";
const SUPABASE_KEY = "sb_publishable_YWfU6ElbbaE8grfwShwlRA_Be0cffXc";
const WHATSAPP_NUMBER = "8801740116023";

/* Products are read from the Supabase products table */
let products = [];
let cart = [];

async function loadProducts() {
  const status = document.getElementById("status");
  status.textContent = "অনলাইন পণ্যের তালিকা লোড হচ্ছে...";

  try {
    const url = SUPABASE_URL +
      "/rest/v1/products?select=id,name,price,description,image_url,stock&order=id.desc";

    const response = await fetch(url, {
      method: "GET",
      headers: {
        "apikey": SUPABASE_KEY,
        "Authorization": "Bearer " + SUPABASE_KEY
      }
    });

    if (!response.ok) {
      const detail = await response.text();
      throw new Error("HTTP " + response.status + " — " + detail);
    }

    const data = await response.json();

    if (!Array.isArray(data)) {
      throw new Error("পণ্যের ডেটা সঠিক ফরম্যাটে পাওয়া যায়নি।");
    }

    products = data;
    renderProducts();

    status.textContent = products.length
      ? "✅ মোট " + products.length + "টি পণ্য পাওয়া গেছে।"
      : "এখনো কোনো পণ্য যোগ করা হয়নি। Supabase Dashboard থেকে পণ্য যোগ করুন।";

  } catch (error) {
    console.error("Supabase error:", error);
    status.textContent =
      "❌ পণ্য লোড হয়নি। Supabase-এর products টেবিল, কলামের নাম ও SELECT policy পরীক্ষা করুন। বিস্তারিত: " +
      error.message;
  }
}

function escapeHTML(value) {
  return String(value ?? "").replace(/[&<>"']/g, function(ch) {
    return {
      "&": "&amp;",
      "<": "&lt;",
      ">": "&gt;",
      '"': "&quot;",
      "'": "&#39;"
    }[ch];
  });
}

function safeImage(value) {
  if (!value) return "";

  try {
    const url = new URL(value);
    if (url.protocol === "https:" || url.protocol === "http:") {
      return url.href;
    }
  } catch (error) {
    return "";
  }

  return "";
}

function renderProducts() {
  const query = document.getElementById("search").value
    .trim()
    .toLowerCase();

  const area = document.getElementById("products");

  const filtered = products.filter(function(p) {
    return String(p.name || "").toLowerCase().includes(query);
  });

  if (!filtered.length) {
    area.innerHTML = products.length
      ? '<div class="empty">আপনার অনুসন্ধানের সঙ্গে মিলে এমন পণ্য পাওয়া যায়নি।</div>'
      : "";
    return;
  }

  area.innerHTML = filtered.map(function(p) {
    const image = safeImage(p.image_url);
    const id = Number(p.id);
    const price = Number(p.price || 0);

    return `
      <article class="card">
        ${image ? `
          <img src="${escapeHTML(image)}"
               alt="${escapeHTML(p.name)}"
               loading="lazy"
               onerror="this.style.display='none'">
        ` : ""}
        <div class="info">
          <h3>${escapeHTML(p.name || "পণ্য")}</h3>
          <div class="price">৳${price.toLocaleString("en-US")}</div>
          <div class="desc">${escapeHTML(p.description || "")}</div>
          <div class="desc">স্টক: ${Number(p.stock || 0)}</div>
          <button class="add" onclick="addToCart(${id})">
            কার্টে যোগ করুন
          </button>
        </div>
      </article>
    `;
  }).join("");
}

function addToCart(id) {
  const product = products.find(function(p) {
    return Number(p.id) === id;
  });

  if (!product) return;

  const existing = cart.find(function(item) {
    return Number(item.id) === id;
  });

  if (existing) {
    existing.qty++;
  } else {
    cart.push({
      id: product.id,
      name: product.name,
      price: Number(product.price || 0),
      qty: 1
    });
  }

  updateCart();
}

function updateCart() {
  document.getElementById("cartcount").textContent =
    cart.reduce(function(sum, item) {
      return sum + item.qty;
    }, 0);

  document.getElementById("cartitems").innerHTML = cart.map(function(item) {
    return `
      <p>
        <strong>${escapeHTML(item.name)}</strong><br>
        ৳${item.price.toLocaleString("en-US")} × ${item.qty}
        <button onclick="changeQty(${Number(item.id)},-1)">−</button>
        <button onclick="changeQty(${Number(item.id)},1)">+</button>
        <button onclick="removeItem(${Number(item.id)})">বাদ দিন</button>
      </p>
    `;
  }).join("");

  const total = cart.reduce(function(sum, item) {
    return sum + item.price * item.qty;
  }, 0);

  document.getElementById("total").textContent =
    "মোট: ৳" + total.toLocaleString("en-US");
}

function changeQty(id, amount) {
  const item = cart.find(function(p) {
    return Number(p.id) === id;
  });

  if (!item) return;

  item.qty += amount;

  if (item.qty <= 0) {
    removeItem(id);
  } else {
    updateCart();
  }
}

function removeItem(id) {
  cart = cart.filter(function(p) {
    return Number(p.id) !== id;
  });

  updateCart();
}

function toggleCart() {
  const box = document.getElementById("cartbox");
  box.style.display = box.style.display === "block" ? "none" : "block";
  updateCart();

  if (box.style.display === "block") {
    box.scrollIntoView({behavior: "smooth", block: "start"});
  }
}

function sendOrder() {
  if (!cart.length) {
    alert("আগে কার্টে পণ্য যোগ করুন।");
    return;
  }

  const customer = document.getElementById("customer").value.trim();
  const phone = document.getElementById("phone").value.trim();
  const address = document.getElementById("address").value.trim();

  if (!customer || !phone || !address) {
    alert("আপনার নাম, মোবাইল নম্বর ও ঠিকানা পূরণ করুন।");
    return;
  }

  const total = cart.reduce(function(sum, item) {
    return sum + item.price * item.qty;
  }, 0);

  let message = "NR Electronic BD - নতুন অর্ডার\n";
  message += "নাম: " + customer + "\n";
  message += "ফোন: " + phone + "\n";
  message += "ঠিকানা: " + address + "\n\n";
  message += "পণ্যের তালিকা:\n";

  cart.forEach(function(item) {
    message += item.name + " × " + item.qty +
      " = ৳" + (item.price * item.qty) + "\n";
  });

  message += "\nসর্বমোট: ৳" + total;

  const whatsappURL = "https://wa.me/" +
    WHATSAPP_NUMBER + "?text=" + encodeURIComponent(message);

  window.open(whatsappURL, "_blank");
}

/* Load products when the website opens */
loadProducts();
</script>

</body>
</html>