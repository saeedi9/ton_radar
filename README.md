
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>TON RADAR</title>
<script src="https://telegram.org/js/telegram-web-app.js"></script>
<style>
*{box-sizing:border-box}
body{margin:0;background:#0b1422;color:#fff;font-family:Arial,sans-serif;padding:16px}
header{text-align:center;padding:15px}
h1{color:#55c7ff;margin:5px}
.sub{color:#91a4bb;font-size:13px}
.stats{display:flex;gap:10px;margin:15px 0}
.stat,.pool,.ad{background:#17263a;border:1px solid #263d58;border-radius:14px;padding:14px}
.stat{flex:1;text-align:center}
.stat b{display:block;font-size:20px;color:#55c7ff;margin-top:7px}
h2{font-size:17px}
.pool{margin:12px 0}
.pool h3{margin:0 0 10px;color:#65ceff}
.info{font-size:13px;color:#b5c5d8;line-height:2;overflow-wrap:anywhere}
.actions{display:flex;gap:8px;margin-top:10px}
a,button{border:0;border-radius:9px;padding:10px;text-decoration:none;font-size:13px}
a{background:#167bb9;color:white}
button{background:#263c56;color:white}
.ad{margin:16px 0;border:1px dashed #e7ae45;text-align:center}
.ad b{color:#ffc65b}
.ad p{font-size:13px;color:#c4d0df}
#status{text-align:center;color:#9eb2c8;padding:20px}
</style>
</head>
<body>

<header>
<h1>TON RADAR 🚀</h1>
<div class="sub">رادار استخرهای جدید شبکه TON</div>
</header>

<div class="stats">
<div class="stat">شبکه<b>TON</b></div>
<div class="stat">تعداد استخرها<b id="count">۰</b></div>
</div>

<h2>🆕 ۲۰ استخر جدیدتر</h2>
<div id="status">در حال دریافت اطلاعات...</div>
<div id="pools"></div>

<script>
const tg=window.Telegram?.WebApp;
if(tg){tg.ready();tg.expand();}

const API="https://api.ston.fi/v1";

function money(v){
 const n=Number(v);
 return Number.isFinite(n)
 ?n.toLocaleString("en-US",{maximumFractionDigits:2})
 :"نامشخص";
}

function safe(v){
 return String(v??"").replace(/[&<>"']/g,c=>({
 "&":"&amp;","<":"&lt;",">":"&gt;",
 '"':"&quot;","'":"&#39;"
 }[c]));
}

async function load(){
 const box=document.getElementById("pools");
 const status=document.getElementById("status");

 status.textContent="در حال دریافت استخرهای جدید...";

 try{
  const [poolResponse,assetResponse]=await Promise.all([

   fetch(API+"/pools/query",{
    method:"POST",
    headers:{"Content-Type":"application/json"},
    body:JSON.stringify({
     condition:null,
     sort_by:["createdAtBlockTimestamp:desc"],
     limit:20
    })
   }),

   fetch(API+"/assets")
  ]);

  if(!poolResponse.ok){
   throw Error("Pool API: "+poolResponse.status);
  }

  if(!assetResponse.ok){
   throw Error("Asset API: "+assetResponse.status);
  }

  const poolData=await poolResponse.json();
  const assetData=await assetResponse.json();

  const pools=poolData.pool_list||[];
  const assets=assetData.asset_list||[];

  const assetMap={};

  assets.forEach(a=>{
   const key=(a.contract_address||a.address||"").toLowerCase();
   if(key)assetMap[key]=a;
  });

  document.getElementById("count").textContent=pools.length;
  box.innerHTML="";

  if(!pools.length){
   status.textContent="استخر جدیدی دریافت نشد.";
   return;
  }

  status.textContent="استخرها از API دریافت شدند.";

  pools.forEach((p,i)=>{

   const a0=assetMap[(p.token0_address||"").toLowerCase()];
   const a1=assetMap[(p.token1_address||"").toLowerCase()];

   const name0=a0?.symbol||a0?.display_name||"توکن ناشناس";
   const name1=a1?.symbol||a1?.display_name||"توکن ناشناس";

   const address=p.address||"";
   const timestamp=Number(p.createdAtBlockTimestamp);
   const created=timestamp
    ?new Date(timestamp*1000).toLocaleString("fa-IR")
    :"نامشخص";

   const card=document.createElement("div");
   card.className="pool";

   card.innerHTML=`
    <h3>${safe(name0)} / ${safe(name1)}</h3>
    <div class="info">
    🕒 زمان ایجاد: ${safe(created)}<br>
    💧 نقدینگی: $${money(p.lp_total_supply_usd)}<br>
    📍 آدرس استخر: ${safe(address||"نامشخص")}<br>
    </div>
    <div class="actions">
    <a target="_blank" rel="noopener"
    href="https://dexscreener.com/ton/${encodeURIComponent(address)}">
    نمودار و قیمت</a>
    <button type="button">آدرس توکن‌ها</button>
    </div>`;

   card.querySelector("button").onclick=()=>{
    alert("Token 0: "+(p.token0_address||"نامشخص")+
    "\nToken 1: "+(p.token1_address||"نامشخص"));
   };

   box.appendChild(card);

   if((i+1)%5===0){
    const ad=document.createElement("div");
    ad.className="ad";
    ad.innerHTML=
    "<b>📢 جایگاه تبلیغاتی</b><p>فضای معرفی پروژه‌های TON</p>";
    box.appendChild(ad);
   }
  });

 }catch(e){
  console.error(e);
  status.textContent="خطا در دریافت اطلاعات: "+e.message;
 }
}

load();
setInterval(load,60000);
</script>
</body>
</html>
