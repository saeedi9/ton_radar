
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

<h2>🆕 ۲۰ استخر اخیر</h2>
<div id="status">در حال دریافت اطلاعات...</div>
<div id="pools"></div>

<script>
const tg=window.Telegram?.WebApp;
if(tg){tg.ready();tg.expand();}

const API="https://api.ston.fi/v1";

function money(v){
 const n=Number(v);
 return Number.isFinite(n)?n.toLocaleString("en-US",{maximumFractionDigits:2}):"نامشخص";
}

function safe(v){
 return String(v??"").replace(/[&<>"']/g,c=>({
 "&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"
 }[c]));
}

function getTime(p){
 const fields=[
  p.createdAtBlockTimestamp,
  p.created_at_block_timestamp,
  p.create_timestamp,
  p.createdAt,
  p.created_at
 ];

 for(const value of fields){
  if(value===null||value===undefined||value==="")continue;
  const n=Number(value);
  if(Number.isFinite(n)&&n>0){
   return n>100000000000?n/1000:n;
  }
  const d=Date.parse(value);
  if(Number.isFinite(d))return d/1000;
 }
 return 0;
}

async function load(){
 const box=document.getElementById("pools");
 const status=document.getElementById("status");

 try{
  const [pr,ar]=await Promise.all([
   fetch(API+"/pools"),
   fetch(API+"/assets")
  ]);

  if(!pr.ok||!ar.ok)throw Error("API error");

  const pd=await pr.json();
  const ad=await ar.json();

  const pools=pd.pool_list||[];
  const assets=ad.asset_list||[];

  const map={};
  assets.forEach(a=>{
   const address=(a.contract_address||a.address||"").toLowerCase();
   if(address)map[address]=a;
  });

  const hasTimes=pools.some(p=>getTime(p)>0);

  let sortedPools=[...pools];

  if(hasTimes){
   sortedPools.sort((a,b)=>getTime(b)-getTime(a));
  }

  const latestPools=sortedPools.slice(0,20);

  document.getElementById("count").textContent=latestPools.length;
  box.innerHTML="";

  if(!latestPools.length){
   status.textContent="هیچ استخری از API دریافت نشد.";
   return;
  }

  status.textContent=hasTimes
   ?"۲۰ استخر مرتب‌شده بر اساس زمان ایجاد"
   :"زمان ایجاد در اطلاعات API موجود نیست؛ نمایش به ترتیب دریافتی";

  latestPools.forEach((p,i)=>{
   const a0=map[(p.token0_address||"").toLowerCase()];
   const a1=map[(p.token1_address||"").toLowerCase()];

   const name0=a0?.symbol||a0?.display_name||"توکن ناشناس";
   const name1=a1?.symbol||a1?.display_name||"توکن ناشناس";

   const address=p.address||"";
   const value=p.lp_total_supply_usd;
   const timestamp=getTime(p);
   const created=timestamp
    ?new Date(timestamp*1000).toLocaleString("fa-IR")
    :"نامشخص";

   const card=document.createElement("div");
   card.className="pool";

   card.innerHTML=`
    <h3>${safe(name0)} / ${safe(name1)}</h3>
    <div class="info">
    🕒 زمان ایجاد: ${safe(created)}<br>
    💧 نقدینگی: $${money(value)}<br>
    📍 آدرس استخر: ${safe(address||"نامشخص")}<br>
    </div>
    <div class="actions">
    <a target="_blank" rel="noopener" href="https://dexscreener.com/ton/${encodeURIComponent(address)}">نمودار و قیمت</a>
    <button type="button">آدرس توکن‌ها</button>
    </div>`;

   card.querySelector("button").onclick=()=>{
    alert("Token 0: "+(p.token0_address||"نامشخص")+
    "\nToken 1: "+(p.token1_address||"نامشخص"));
   };

   box.appendChild(card);

   if((i+1)%5===0){
    const adBox=document.createElement("div");
    adBox.className="ad";
    adBox.innerHTML="<b>📢 جایگاه تبلیغاتی</b><p>فضای معرفی پروژه‌های TON</p>";
    box.appendChild(adBox);
   }
  });

 }catch(e){
  status.textContent="دریافت اطلاعات ناموفق بود. دوباره تلاش کن.";
 }
}

load();
setInterval(load,60000);
</script>
</body>
</html>
