---
layout: post
title: coordinateConvert
---

## 经纬度 ↔ 高斯3度带XY坐标转换
（带带号输入输出，符合中国国家测绘标准）

### 1. 经纬度 → 高斯XY（带带号）
<input type="number" id="lat" placeholder="纬度（如39.9）" step="0.000001" min="-90" max="90">
<input type="number" id="lng" placeholder="经度（如116.4）" step="0.000001" min="-180" max="180">
<button onclick="convertToGauss()">转换</button>
<p>结果：<span id="gauss-result"></span></p>

### 2. 高斯XY（带带号） → 经纬度
<div style="display:flex;gap:10px;align-items:center">
  <div>
    <label>带号：</label>
    <input type="number" id="input-zone" placeholder="如50" min="1" max="60" value="50">
  </div>
  <div>
    <label>X：</label>
    <input type="number" id="input-x" placeholder="如500000" step="0.01">
  </div>
  <div>
    <label>Y：</label>
    <input type="number" id="input-y" placeholder="如4421415" step="0.01">
  </div>
</div>
<button onclick="convertToWGS84()">转换</button>
<p>结果：<span id="wgs84-result"></span></p>

<script src="https://cdnjs.cloudflare.com/ajax/libs/proj4js/2.8.0/proj4.js"></script>
<script>
// 定义坐标系
proj4.defs("EPSG:4326", "+proj=longlat +ellps=WGS84 +datum=WGS84 +no_defs");

// 计算3度带带号（中国范围25-45带）
function calculateZone(longitude) {
  return Math.floor((longitude + 1.5) / 3);
}

// 验证带号有效性（中国常用25-45带）
function isValidZone(zone) {
  return zone >= 25 && zone <= 45;
}

// 经纬度 → 高斯XY（自动计算带号）
function convertToGauss() {
  const lat = parseFloat(document.getElementById('lat').value);
  const lng = parseFloat(document.getElementById('lng').value);
  
  if (isNaN(lat) || isNaN(lng)) {
    alert("请输入有效的经纬度！");
    return;
  }

  const zone = calculateZone(lng);
  if (!isValidZone(zone)) {
    alert(`警告：计算出的带号${zone}不在中国常用范围(25-45)`);
  }

  // 定义高斯投影（3度带）
  const projCode = `EPSG:${zone < 30 ? 2430 + zone : 2380 + zone}`;
  proj4.defs(projCode, 
    `+proj=tmerc +lat_0=0 +lon_0=${3*zone} +k=1 +x_0=500000 +y_0=0 ` +
    `+ellps=GRS80 +units=m +no_defs`
  );

  // 执行转换（注意坐标顺序为经度、纬度）
  const [x, y] = proj4("EPSG:4326", projCode, [lng, lat]);
  
  // 输出结果（X加带号前两位）
  const xWithPrefix = (zone * 1000000 + x).toFixed(2);
  document.getElementById('gauss-result').innerHTML = `
    带号：${zone} | 
    X=${xWithPrefix}（${zone}${String(x.toFixed(2)).padStart(8, '0')}），
    Y=${y.toFixed(2)}
  `;
}

// 高斯XY → 经纬度（需指定带号）
function convertToWGS84() {
  const zone = parseInt(document.getElementById('input-zone').value);
  let x = parseFloat(document.getElementById('input-x').value);
  const y = parseFloat(document.getElementById('input-y').value);

  if (isNaN(zone) || isNaN(x) || isNaN(y)) {
    alert("请输入有效的带号和坐标！");
    return;
  }

  // 处理8位X坐标（如50448251 → 448251）
  if (x > 10000000) {
    x = x % 1000000;
    alert(`检测到8位X坐标，已自动提取后6位：${x}`);
  }

  // 定义高斯投影
  const projCode = `EPSG:${zone < 30 ? 2430 + zone : 2380 + zone}`;
  proj4.defs(projCode, 
    `+proj=tmerc +lat_0=0 +lon_0=${3*zone} +k=1 +x_0=500000 +y_0=0 ` +
    `+ellps=GRS80 +units=m +no_defs`
  );

  // 执行转换
  const [lng, lat] = proj4(projCode, "EPSG:4326", [x, y]);
  
  document.getElementById('wgs84-result').innerHTML = `
    纬度=${lat.toFixed(6)}°，经度=${lng.toFixed(6)}°
  `;
}
</script>
