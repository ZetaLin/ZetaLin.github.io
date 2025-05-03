---
layout: post
title: CoordinateConvert
---
 

---

## 经纬度 ↔ XY 坐标转换工具
（基于前端 JavaScript 实现）

### 1. 经纬度 → XY（平面坐标）
<input type="number" id="lat" placeholder="纬度（如 39.9）">
<input type="number" id="lng" placeholder="经度（如 116.4）">
<button onclick="convertToXY()">转换</button>
<p>结果：<span id="xy-result"></span></p>

### 2. XY → 经纬度
<input type="number" id="x" placeholder="X">
<input type="number" id="y" placeholder="Y">
<button onclick="convertToLatLng()">转换</button>
<p>结果：<span id="latlng-result"></span></p>

<script>
// 简单墨卡托投影示例（实际应用建议用 proj4js）
function convertToXY() {
  const lat = parseFloat(document.getElementById('lat').value);
  const lng = parseFloat(document.getElementById('lng').value);
  // 简化计算（仅演示，真实场景需复杂公式）
  const x = lng * 100000;
  const y = lat * 100000;
  document.getElementById('xy-result').textContent = `X=${x.toFixed(2)}, Y=${y.toFixed(2)}`;
}

function convertToLatLng() {
  const x = parseFloat(document.getElementById('x').value);
  const y = parseFloat(document.getElementById('y').value);
  const lat = y / 100000;
  const lng = x / 100000;
  document.getElementById('latlng-result').textContent = `纬度=${lat.toFixed(6)}, 经度=${lng.toFixed(6)}`;
}
</script>

<!-- 如需高精度转换，取消下方注释 -->
<!--
<script src="https://cdnjs.cloudflare.com/ajax/libs/proj4js/2.8.0/proj4.js"></script>
<script>
// 专业库示例（如UTM投影）
proj4.defs("EPSG:4326", "+proj=longlat +datum=WGS84 +no_defs");
proj4.defs("EPSG:32650", "+proj=utm +zone=50 +datum=WGS84 +units=m +no_defs");

function convertToXY() {
  const lat = parseFloat(document.getElementById('lat').value);
  const lng = parseFloat(document.getElementById('lng').value);
  const [x, y] = proj4("EPSG:4326", "EPSG:32650", [lng, lat]);
  document.getElementById('xy-result").textContent = `X=${x.toFixed(2)}, Y=${y.toFixed(2)}`;
}
</script>
-->
