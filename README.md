
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>TON RADAR</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#071426;color:#fff;font-family:Arial,sans-serif;padding:18px}
header{text-align:center;padding:20px 0}
h1{color:#39c5ff;margin:5px}
.sub{color:#91a9c5;font-size:13px}
#status{text-align:center;color:#9bb5ce;margin:15px}
.card{background:#10243c;border:1px solid #23415f;border-radius:13px;padding:14px;margin:12px 0}
.name{font-size:17px;font-weight:bold;color:#5bd0ff}
.info{font-size:13px;color:#c4d3e4;margin-top:9px;line-height:2}
.address{font-size:10px;direction:ltr;word-break:break-all;color:#829bb7}
a{color:#48caff}
.ad{background:#172d45;border:1px dashed #64809b;border-radius:12px;text-align:center;padding:17px;margin:15px 0;color:#ffcf69}
button{background:#168bd0;color:white;border:0;border-radius:9px;padding:11px 20px;font-size:15px}
</style>
</head>
<body>
<header>
<h1>🚀 TON RADAR</h1>
<div class="sub">رادار استخرهای جدید شبکه TON</div>
</header>

<div style="text-align:center">
<button onclick="load()">🔄 بروزرسانی</button>
</div>
<div id="status">در حال دریافت اطلاعات...</div>
<div id="list"></div>

<script>
const API="https://api.ston.fi/v1";

function safe(s){
 return String(s??"").replace(/[&<>"']/g,c=>({
 "&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"
 }[c]));
}

async function load(){
 const list=document.getElementById("list");
 const status=document.getElementById("status");
 status.textContent="در حال دریافت ۲۰ استخر جدید...";
 list.innerHTML="";

 try{
  const params=new URLSearchParams({
   sort_by:"createdAtBlockTimestamp:desc",
   limit:"20"
  });

  const [poolRes,assetRes]=await Promise.all([
   fetch(API+"/pools/query?"+params.toString(),{
    method:"POST"
   }),
   fetch(API+"/assets")
  ]);

  if(!poolRes.ok) throw Error("خطای استخرها: "+poolRes.status);
  if(!assetRes.ok) throw Error("خطای توکن‌ها: "+assetRes.status);

  const pd=await poolRes.json();
  const ad=await assetRes.json();

  const pools=pd.pool_list||[];
  const assets=ad.asset_list||[];

  if(!pools.length) throw Error("هیچ استخری از API دریافت نشد");

  const assetMap={};
  assets.forEach(a=>{
   const addr=a.contract_address||a.address;
   if(addr) assetMap[addr]=a.symbol||a.display_name||"توکن ناشناس";
  });

  pools.sort((a,b)=>
   Number(b.createdAtBlockTimestamp||0)-
   Number(a.createdAtBlockTimestamp||0)
  );

  status.textContent="۲۰ استخر بر اساس زمان ایجاد، از جدید به قدیم";

  pools.slice(0,20).forEach((p,i)=>{
   const a0=assetMap[p.token0_address]||"توکن ناشناس";
   const a1=assetMap[p.token1_address]||"توکن ناشناس";
   const address=p.address||p.id||"";
   const time=p.createdAtBlockTimestamp
    ?new Date(Number(p.createdAtBlockTimestamp)*1000).toLocaleString("fa-IR")
    :"نامشخص";

   const card=document.createElement("div");
   card.className="card";
   card.innerHTML=`
    <div class="name">${i+1}. ${safe(a0)} / ${safe(a1)}</div>
    <div class="info">
     💧 نقدینگی: $${Number(p.lp_total_supply_usd||0).toLocaleString("en-US")}<br>
     🕒 زمان ایجاد: ${safe(time)}<br>
     <a href="https://dexscreener.com/ton/${encodeURIComponent(address)}" target="_blank">مشاهده در DexScreener</a>
    </div>
    <div class="address">${safe(address)}</div>
   `;
   list.appendChild(card);

   if((i+1)%5===0 && i!==19){
    const adBox=document.createElement("div");
    adBox.className="ad";
    adBox.innerHTML="📢 جایگاه تبلیغات TON RADAR";
    list.appendChild(adBox);
   }
  });

 }catch(e){
  status.textContent="خطا: "+e.message;
 }
}

load();
setInterval(load,60000);
</script>
</body>
</html>
