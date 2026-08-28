# xe-dien-os
<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>OS Electric Rental</title>
<style>
*{box-sizing:border-box}body{margin:0;font-family:Inter,Arial,sans-serif;background:#f4f7f6;color:#17221f}
header{background:#0d1714;color:#fff;padding:18px 24px;display:flex;justify-content:space-between;align-items:center}
.logo{font-size:22px;font-weight:800}.sub{font-size:12px;color:#aebdb8}
.wrap{max-width:1200px;margin:auto;padding:22px}
.stats{display:grid;grid-template-columns:repeat(4,1fr);gap:14px;margin-bottom:20px}
.card{background:white;border-radius:16px;padding:18px;box-shadow:0 4px 18px #0000000b}.stat b{font-size:28px;display:block;margin-top:6px}
.grid{display:grid;grid-template-columns:1.4fr .8fr;gap:18px}.toolbar{display:flex;gap:10px;margin-bottom:15px}
input,select,button{border:1px solid #dbe4e1;border-radius:10px;padding:11px 12px;font:inherit}
input{width:100%}button{cursor:pointer;background:#0d1714;color:white;border:0}.secondary{background:#edf3f1;color:#17221f}
.btn-export{background:#107c41;color:white;font-weight:600;display:inline-flex;align-items:center;gap:6px;transition:background 0.2s}
.btn-export:hover{background:#0b5c30}
.vehicles{display:grid;grid-template-columns:repeat(3,1fr);gap:12px}
.vehicle{border:1px solid #e1e9e6;border-radius:14px;padding:14px;background:#fff;cursor:pointer}
.vehicle:hover{border-color:#19a974;transform:translateY(-1px)}.id{font-weight:800}.badge{font-size:11px;padding:5px 8px;border-radius:20px;float:right}
.free{background:#dcf8e9;color:#087443}.rent{background:#e0efff;color:#1761a8}.maint{background:#fff0d8;color:#9a6200}
.price{margin-top:12px;color:#52635e;font-size:13px}
.panel h2{margin-top:0}.row{display:grid;grid-template-columns:1fr 1fr;gap:10px}.full{grid-column:1/-1}
.total{font-size:25px;font-weight:800;margin:12px 0}.hint{font-size:12px;color:#6c7b76}
table{width:100%;border-collapse:collapse;font-size:13px}th,td{padding:9px;border-bottom:1px solid #edf1ef;text-align:left}
.section-header{display:flex;justify-content:space-between;align-items:center;margin-bottom:14px}
.section-header h2{margin:0}
@media(max-width:800px){.stats{grid-template-columns:repeat(2,1fr)}.grid{grid-template-columns:1fr}.vehicles{grid-template-columns:repeat(2,1fr)}}@media(max-width:500px){.vehicles{grid-template-columns:1fr}.wrap{padding:12px}}
</style>
</head>
<body>
<header><div><div class="logo">⚡ OS ELECTRIC RENTAL</div><div class="sub">Quản lý & cho thuê xe điện</div></div><div id="clock"></div></header>
<div class="wrap">
<div class="stats">
<div class="card stat">Tổng xe<b>35</b></div><div class="card stat">Xe trống<b id="freeCount">35</b></div>
<div class="card stat">Đang thuê<b id="rentCount">0</b></div><div class="card stat">Doanh thu<b id="revenue">0đ</b></div>
</div>
<div class="grid">
<div class="card"><div class="toolbar"><input id="search" placeholder="Tìm mã xe..."><select id="filter"><option value="all">Tất cả</option><option value="free">Xe trống</option><option value="rent">Đang thuê</option></select></div><div class="vehicles" id="vehicles"></div></div>
<div class="card panel"><h2>📝 Tạo lượt thuê</h2><div class="row">
<div><label>Mã xe</label><select id="vehicleId"><option value="">-- Chọn xe --</option></select></div>
<div><label>Khách thuê</label><input id="customer" placeholder="Tên khách"></div>
<div><label>SĐT</label><input id="phone" placeholder="Số điện thoại"></div>
<div><label>Ngày thuê</label><input id="start" type="date"></div>
<div><label>Ngày trả</label><input id="end" type="date"></div>
<div><label>Giờ nhận</label><input id="startTime" type="time"></div>
<div><label>Giờ trả</label><input id="endTime" type="time"></div>
<div><label>Vị trí nhận/trả</label><input id="location" placeholder="VD: An Hải / Hội An"></div>
<div><label>Tiền cọc</label><input id="deposit" type="number" value="1000000"></div>
</div><div class="hint" id="rateHint">Chọn xe và ngày thuê/trả để tính giá.</div><div class="total" id="total">0đ</div><button onclick="rentVehicle()">Xác nhận cho thuê</button>
<div id="returnBox" style="display:none;margin-top:14px;border-top:1px solid #edf1ef;padding-top:14px">
<h3>🔄 Trả xe</h3>
<div class="row">
<div><label>Giờ trả thực tế</label><input id="returnTime" type="time"></div>
<div><label>Trạm sạc</label><input id="station" placeholder="VD: Trạm 01"></div>
<div class="full"><label>Note riêng cho xe</label><input id="returnNote" placeholder="VD: Đang sạc, pin 45%, chờ khách..."></div>
</div>
<button onclick="returnVehicle()">Xác nhận trả xe</button>
</div></div>
</div>
<div class="card" style="margin-top:18px">
<div class="section-header">
<h2>Lịch sử thuê</h2>
<button class="btn-export" onclick="exportHistoryCSV()">📊 Xuất Excel / CSV</button>
</div>
<table><thead><tr><th>Xe</th><th>Khách</th><th>Nhận → Trả</th><th>Vị trí</th><th>Trạm sạc / Note</th><th>Tổng</th><th>Trạng thái</th></tr></thead><tbody id="history"></tbody></table>
</div>
</div>
<script>
const prices=[{max:3,p:150000},{max:7,p:130000},{max:14,p:110000},{max:25,p:90000},{max:9999,p:70000}];
let vehicles=JSON.parse(localStorage.getItem('osVehicles')||'null')||Array.from({length:35},(_,i)=>({id:`OS-${String(i+1).padStart(2,'0')}`,status:'free'}));
let history=JSON.parse(localStorage.getItem('osHistory')||'[]');
const money=n=>new Intl.NumberFormat('vi-VN').format(n)+'đ';

function save(){
  localStorage.setItem('osVehicles',JSON.stringify(vehicles));
  localStorage.setItem('osHistory',JSON.stringify(history));
}

function render(){
  const q=document.getElementById('search').value.toLowerCase(), f=document.getElementById('filter').value;
  document.getElementById('vehicles').innerHTML=vehicles.filter(v=>(f==='all'||v.status===f)&&v.id.toLowerCase().includes(q)).map(v=>`
  <div class="vehicle" onclick="selectVehicle('${v.id}')"><span class="id">${v.id}</span><span class="badge ${v.status}">${v.status==='free'?'TRỐNG':'ĐANG THUÊ'}</span>
  <div class="price">${v.status==='free'?(v.station?`📍 ${v.station}`:'Sẵn sàng nhận khách'):`👤 ${v.customer||''}<br>📍 ${v.location||'Chưa có vị trí'}<br>🕒 ${v.startTime||''} → ${v.endTime||''}`}</div>
  ${v.status==='free'&&v.note?`<div class="hint">📝 ${v.note}</div>`:''}</div>`).join('');
  
  document.getElementById('freeCount').textContent=vehicles.filter(v=>v.status==='free').length;
  document.getElementById('rentCount').textContent=vehicles.filter(v=>v.status==='rent').length;
  document.getElementById('revenue').textContent=money(history.reduce((s,x)=>s+x.total,0));
  populateVehicleSelect();
  document.getElementById('history').innerHTML=history.slice().reverse().map(x=>`<tr><td>${x.id}</td><td>${x.customer}</td><td>${x.start} ${x.startTime||''}<br>→ ${x.end} ${x.returnTime||x.endTime||''}</td><td>${x.location||''}</td><td>${x.station||'-'}${x.note?`<br>📝 ${x.note}`:''}</td><td>${money(x.total)}</td><td>${x.status}</td></tr>`).join('')||'<tr><td colspan="7">Chưa có lượt thuê.</td></tr>';
}

function populateVehicleSelect(){
  const sel=document.getElementById('vehicleId');
  const current=sel.value;
  sel.innerHTML='<option value="">-- Chọn xe --</option>'+vehicles.map(v=>`<option value="${v.id}" ${v.status==='rent'?'disabled':''}>${v.id} — ${v.status==='free'?'TRỐNG':'ĐANG THUÊ'}</option>`).join('');
  if(vehicles.some(v=>v.id===current)) sel.value=current;
}

function selectVehicle(id){
  const sel=document.getElementById('vehicleId');
  sel.value=id;
  const v=vehicles.find(x=>x.id===id);
  document.getElementById('returnBox').style.display=v && v.status==='rent'?'block':'none';
  if(v && v.status==='rent'){
    document.getElementById('customer').value=v.customer||'';
    document.getElementById('phone').value=v.phone||'';
    document.getElementById('start').value=v.start||'';
    document.getElementById('end').value=v.end||'';
    document.getElementById('startTime').value=v.startTime||'';
    document.getElementById('endTime').value=v.endTime||'';
    document.getElementById('location').value=v.location||'';
    document.getElementById('deposit').value=v.deposit||0;
    document.getElementById('returnNote').value=v.note||'';
    document.getElementById('station').value=v.station||'';
  } else if(v){
    document.getElementById('customer').value='';
    document.getElementById('phone').value='';
    document.getElementById('start').value='';
    document.getElementById('end').value='';
    document.getElementById('startTime').value='';
    document.getElementById('endTime').value='';
    document.getElementById('location').value='';
    document.getElementById('returnNote').value='';
    document.getElementById('station').value='';
  }
}

function calc(){
  let s=document.getElementById('start').value,e=document.getElementById('end').value;
  if(!s||!e)return 0;
  let days=Math.ceil((new Date(e)-new Date(s))/86400000);
  if(days===0) days=1;
  if(days<1){document.getElementById('rateHint').textContent='Ngày trả không được trước ngày thuê.';return 0}
  let rate=prices.find(x=>days<=x.max).p,total=days*rate;
  document.getElementById('rateHint').textContent=`${days} ngày × ${money(rate)}/ngày`;
  document.getElementById('total').textContent=money(total);
  return total;
}

function rentVehicle(){
  let id=document.getElementById('vehicleId').value,customer=document.getElementById('customer').value.trim(),s=document.getElementById('start').value,e=document.getElementById('end').value,total=calc();
  if(!id||!customer||!total)return alert('Chỉ cần chọn xe, nhập tên khách và ngày thuê/trả là có thể lưu. Các thông tin khác có thể bổ sung sau.');
  let v=vehicles.find(x=>x.id===id);if(v.status!=='free')return alert('Xe này đang được thuê.');
  let days=Math.ceil((new Date(e)-new Date(s))/86400000),rate=total/(days||1);
  v.status='rent';v.customer=customer;v.phone=document.getElementById('phone').value;
  v.start=s;v.end=e;v.startTime=document.getElementById('startTime').value;v.endTime=document.getElementById('endTime').value;
  v.location=document.getElementById('location').value;v.deposit=Number(document.getElementById('deposit').value||0);
  history.push({id,customer,start:s,end:e,startTime:v.startTime,endTime:v.endTime,location:v.location,rate,total,status:'Đang thuê',returned:false});
  save();render();alert(`Đã tạo lượt thuê ${id} - ${customer}`);
}

function returnVehicle(){
  let id=document.getElementById('vehicleId').value,v=vehicles.find(x=>x.id===id);
  if(!v||v.status!=='rent')return alert('Xe này không ở trạng thái đang thuê.');
  let h=[...history].reverse().find(x=>x.id===id&&x.status==='Đang thuê');
  let station=document.getElementById('station').value.trim(),note=document.getElementById('returnNote').value.trim();
  v.status='free';v.station=station;v.note=note;v.returnTime=document.getElementById('returnTime').value;
  if(h){h.status='Đã trả';h.returnTime=v.returnTime;h.station=station;h.note=note;h.returned=true}
  save();render();selectVehicle(id);
  alert(`Đã trả ${id}. Xe được ghi nhận tại ${station||'chưa nhập trạm sạc'}.`);
}

function exportHistoryCSV(){
  if(!history || history.length === 0){
    return alert('Chưa có dữ liệu lịch sử thuê để xuất file!');
  }
  
  const headers = ["Mã Xe", "Khách Hàng", "SĐT", "Ngày Thuê", "Giờ Thuê", "Ngày Trả", "Giờ Trả", "Vị Trí", "Tiền Cọc (VNĐ)", "Đơn Giá/Ngày (VNĐ)", "Tổng Tiền (VNĐ)", "Trạm Sạc", "Ghi Chú", "Trạng Thái"];
  
  const rows = history.map(item => [
    item.id || '',
    item.customer || '',
    item.phone || '',
    item.start || '',
    item.startTime || '',
    item.end || '',
    item.returnTime || item.endTime || '',
    item.location || '',
    item.deposit || 0,
    item.rate || 0,
    item.total || 0,
    item.station || '',
    item.note || '',
    item.status || ''
  ]);
  
  let csvContent = "\uFEFF";
  csvContent += headers.map(h => `"${h.replace(/"/g, '""')}"`).join(",") + "\r\n";
  
  rows.forEach(row => {
    csvContent += row.map(cell => `"${String(cell).replace(/"/g, '""')}"`).join(",") + "\r\n";
  });
  
  const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
  const url = URL.createObjectURL(blob);
  const link = document.createElement("a");
  const now = new Date();
  const dateStr = now.toISOString().slice(0,10);
  
  link.setAttribute("href", url);
  link.setAttribute("download", `Lich_Su_Thue_Xe_OS_${dateStr}.csv`);
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}

document.getElementById('search').oninput=render;
document.getElementById('filter').onchange=render;
document.getElementById('vehicleId').onchange=function(){selectVehicle(this.value)};
['start','end'].forEach(x=>document.getElementById(x).onchange=calc);
function tick(){document.getElementById('clock').textContent=new Date().toLocaleString('vi-VN')}
setInterval(tick,1000);tick();populateVehicleSelect();render();
</script>
</body>
</html>
