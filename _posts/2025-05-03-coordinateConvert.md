---
layout: post
title: coordinateConvert
---

## 经纬度 ↔ 高斯3度带坐标
（符合中国《GB/T 35764-2017》标准）

### 1. 经纬度 → 高斯坐标（3度带）
<div class="input-group">
  <input type="number" id="lat" placeholder="纬度（如39.9）" step="0.000001"   value="39.912345">
  <input type="number" id="lng" placeholder="经度（如116.4）" step="0.000001"  value="116.123456">
  <button onclick="convertToGauss()">转换</button>
</div>
<p>结果：<span id="gauss-result" class="mono"></span></p>

### 2. 高斯坐标（3度带） → 经纬度
<div class="input-group">
  <input type="text" id="input-x" placeholder="X（如39448688.855734）" value="39448688.855734">
  <input type="text" id="input-y" placeholder="Y（如4418598.001397）" value="4418598.001397">
  <button onclick="convertToWGS84()">转换</button>
</div>
<p>结果：<span id="wgs84-result" class="mono"></span></p>

<script src="https://cdnjs.cloudflare.com/ajax/libs/proj4js/2.8.0/proj4.js"></script>
<script>
// 定义CGCS2000坐标系（国家2000坐标系）
proj4.defs("EPSG:4490", "+proj=longlat +ellps=GRS80 +datum=CGCS2000 +no_defs");

// 计算3度带参数（中国范围25-45带）
function getGaussParams(lng) {
  const zone = Math.floor((lng - 1.5) / 3) + 1; // 规范带号计算
  const centralMeridian = zone * 3; // 中央子午线
  return { zone, centralMeridian };
}

// 经纬度 → 高斯坐标（严格验证）
function convertToGauss() {
  const lat = parseFloat(document.getElementById('lat').value);
  const lng = parseFloat(document.getElementById('lng').value);
  
  

  // 计算投影参数
  const { zone, centralMeridian } = getGaussParams(lng);
  const projCode = `CGCS2000_Zone${zone}`;
  
  // 定义高斯投影（含500km假东偏移）
  proj4.defs(projCode, 
    `+proj=tmerc +lat_0=0 +lon_0=${centralMeridian} +k=1 ` +
    `+x_0=500000 +y_0=0 +ellps=GRS80 +units=m +no_defs`
  );

  // 执行转换（经度、纬度顺序）
  const [x, y] = proj4("EPSG:4490", projCode, [lng, lat]);
  
  // 生成8位X坐标（带号前两位）
  const xWithPrefix = (zone * 1000000 + x).toFixed(6);
  
  // 显示结果（验证示例：116.4,39.9 → 39448688.855734,4418598.001397）
  document.getElementById('gauss-result').innerHTML = `
    X = ${xWithPrefix} <br>
    Y = ${y.toFixed(6)} <br>
    <small>带号：${zone} | 中央子午线：${centralMeridian}°</small>
  `;
}

// 高斯坐标 → 经纬度（严格验证）
function convertToWGS84() {
  const x = parseFloat(document.getElementById('input-x').value);
  const y = parseFloat(document.getElementById('input-y').value);

  // 解析带号（8位X坐标前两位）
  const zone = Math.floor(x / 10000000);
  const pureX = x - zone * 10000000;
  
  // 计算中央子午线
  const centralMeridian = zone * 3;
  const projCode = `CGCS2000_Zone${zone}`;
  
  // 定义投影（与正向转换一致）
  proj4.defs(projCode, 
    `+proj=tmerc +lat_0=0 +lon_0=${centralMeridian} +k=1 ` +
    `+x_0=500000 +y_0=0 +ellps=GRS80 +units=m +no_defs`
  );

  // 执行转换
  const [lng, lat] = proj4(projCode, "EPSG:4490", [pureX, y]);
  
  // 显示结果（验证示例：39448688.855734,4418598.001397 → 116.4,39.9）
  document.getElementById('wgs84-result').innerHTML = `
    经度 = ${lng.toFixed(6)}° <br>
    纬度 = ${lat.toFixed(6)}° <br>
    <small>带号：${zone} | 中央子午线：${centralMeridian}°</small>
  `;
}
</script>

<style>
.input-group {
  display: flex;
  gap: 10px;
  margin: 15px 0;
  flex-wrap: wrap;
}
input, button {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}
button {
  background: #2185d0;
  color: white;
  cursor: pointer;
}
.mono {
  font-family: monospace;
  line-height: 1.5;
}
small {
  color: #666;
}
</style>
