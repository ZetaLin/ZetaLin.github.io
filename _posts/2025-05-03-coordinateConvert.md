---
layout: post
title: coordinateConvert
---

## CGCS2000 经纬度 ↔ 高斯平面坐标
（符合《GB/T 35764-2017》国家规范）

### 1. 经纬度 → 高斯平面坐标
<div class="input-group">
  <input type="number" id="lat" placeholder="纬度（如39.9）" step="0.000001" min="0" max="90" value="39.9">
  <input type="number" id="lng" placeholder="经度（如116.4）" step="0.000001" min="70" max="140" value="116.4">
  <select id="projection-type">
    <option value="3">3度带</option>
    <option value="6">6度带</option>
  </select>
  <button onclick="convertToGauss()">转换</button>
</div>
<p>结果：<span id="gauss-result" style="font-family: monospace;"></span></p>

### 2. 高斯平面坐标 → 经纬度
<div class="input-group">
  <input type="text" id="input-x" placeholder="X（如39448688.855734）" value="39448688.855734">
  <input type="text" id="input-y" placeholder="Y（如4418598.001397）" value="4418598.001397">
  <button onclick="convertToWGS84()">转换</button>
</div>
<p>结果：<span id="wgs84-result" style="font-family: monospace;"></span></p>

<script src="https://cdnjs.cloudflare.com/ajax/libs/proj4js/2.8.0/proj4.js"></script>
<script>
// 定义CGCS2000坐标系（EPSG:4490）
proj4.defs("EPSG:4490", "+proj=longlat +ellps=GRS80 +datum=CGCS2000 +no_defs");

// 经纬度 → 高斯平面坐标
function convertToGauss() {
  const lat = parseFloat(document.getElementById('lat').value);
  const lng = parseFloat(document.getElementById('lng').value);
  const is3Degree = document.getElementById('projection-type').value === "3";

  // 计算中央子午线（3度带或6度带）
  const centralMeridian = is3Degree ? 
    Math.floor((lng + 1.5) / 3) * 3 : 
    Math.floor(lng / 6) * 6 + 3;

  // 定义高斯投影（CGCS2000椭球）
  const projCode = `CGCS2000_${centralMeridian}`;
  proj4.defs(projCode, 
    `+proj=tmerc +lat_0=0 +lon_0=${centralMeridian} +k=1 ` +
    `+x_0=500000 +y_0=0 +ellps=GRS80 +units=m +no_defs`
  );

  // 执行转换（注意坐标顺序为经度、纬度）
  const [x, y] = proj4("EPSG:4490", projCode, [lng, lat]);
  
  // 输出结果（Y为自然值，X加带号）
  const zoneCode = is3Degree ? 
    Math.floor((lng + 1.5) / 3) : 
    Math.floor(lng / 6) + 30;
  const fullX = is3Degree ? 
    (zoneCode * 10000000 + x).toFixed(6) : 
    (zoneCode * 1000000 + x).toFixed(6);

  document.getElementById('gauss-result').innerHTML = `
    X = ${fullX}（带号${zoneCode}）<br>
    Y = ${y.toFixed(6)}<br>
    <small>投影类型：${is3Degree ? '3' : '6'}度带 | 中央子午线：${centralMeridian}°</small>
  `;
}

// 高斯平面坐标 → 经纬度
function convertToWGS84() {
  const x = parseFloat(document.getElementById('input-x').value);
  const y = parseFloat(document.getElementById('input-y').value);

  // 自动判断带号（3度带8位X坐标/6度带7位X坐标）
  const zoneCode = x > 100000000 ? 
    Math.floor(x / 10000000) : 
    Math.floor(x / 1000000);
  const is3Degree = x > 100000000;
  const centralMeridian = is3Degree ? 
    zoneCode * 3 : 
    (zoneCode - 30) * 6 + 3;
  const pureX = is3Degree ? 
    x - zoneCode * 10000000 : 
    x - zoneCode * 1000000;

  // 定义高斯投影
  const projCode = `CGCS2000_${centralMeridian}`;
  proj4.defs(projCode, 
    `+proj=tmerc +lat_0=0 +lon_0=${centralMeridian} +k=1 ` +
    `+x_0=500000 +y_0=0 +ellps=GRS80 +units=m +no_defs`
  );

  // 执行转换
  const [lng, lat] = proj4(projCode, "EPSG:4490", [pureX, y]);
  
  document.getElementById('wgs84-result').innerHTML = `
    经度 = ${lng.toFixed(9)}°<br>
    纬度 = ${lat.toFixed(9)}°<br>
    <small>带号：${zoneCode}（${is3Degree ? '3' : '6'}度带） | 中央子午线：${centralMeridian}°</small>
  `;
}
</script>

<style>
.input-group {
  display: flex;
  gap: 10px;
  margin: 15px 0;
  align-items: center;
}
input, select {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}
button {
  padding: 8px 15px;
  background: #0366d6;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
</style>
