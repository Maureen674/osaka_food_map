osaka food picked by maureen
<html lang="zh">
<head>
  <meta charset="UTF-8" />
  <title>大阪美食地图</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />

  <!-- Leaflet 地图库 -->
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://cdn.jsdelivr.net/npm/leaflet@1.9.4/dist/leaflet.js"></script>

  <style>
    body { margin: 0; font-family: sans-serif; display: flex; height: 100vh; }
    #map { height: 100vh; width: 75%; }
    #sidebar { width: 25%; overflow-y: auto; padding: 10px; box-shadow: -2px 0 5px rgba(0,0,0,0.1); }
    .place { margin: 8px 0; padding: 8px; border-radius: 8px; background: #f4f4f4; cursor: pointer; }
    .place:hover { background: #e0e0e0; }
    #error { color: red; font-size: 14px; margin-top: 10px; }
    a { text-decoration: none; color: #0077cc; }
  </style>
</head>
<body>
  <div id="map"></div>
  <div id="sidebar">
    <h2>我的大阪清单</h2>
    <div id="places"></div>
    <div id="error"></div>
    <button onclick="refreshLocation()">刷新位置</button>
  </div>

  <script>
    // 餐厅/景点数据
    const destinations = [
      { name: "Sashisu", lat: 34.6984792, lng: 135.4997499 },
      { name: "Kibitaki", lat: 34.6712265, lng: 135.5020764 },
      { name: "Tempura Tarojiro", lat: 34.6678979, lng: 135.503683 },
      { name: "宫田面儿", lat: 34.6738237, lng: 135.5042491 },
      { name: "Canelé du Japon", lat: 34.6766352, lng: 135.5065096 }
    ];

    // 初始化地图
    let map = L.map("map").setView([34.6937, 135.5023], 14);
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "© OpenStreetMap"
    }).addTo(map);

    // 在侧边栏和地图上显示清单
    const placesDiv = document.getElementById("places");
    destinations.forEach(d => {
      // Google Maps 导航链接
      const gmapUrl = `https://www.google.com/maps/dir/?api=1&destination=${d.lat},${d.lng}`;

      // 侧边栏列表
      const div = document.createElement("div");
      div.className = "place";
      div.innerHTML = `<b>${d.name}</b><br><a href="${gmapUrl}" target="_blank">➡️ 去这里</a>`;
      div.onclick = () => map.setView([d.lat, d.lng], 16);
      placesDiv.appendChild(div);

      // 地图标记 + popup
      L.marker([d.lat, d.lng]).addTo(map)
        .bindPopup(`<b>${d.name}</b><br><a href="${gmapUrl}" target="_blank">在Google Maps中导航</a>`);
    });

    // 定位功能
    function refreshLocation() {
      if (!navigator.geolocation) {
        showError("❌ 你的浏览器不支持定位功能");
        return;
      }
      navigator.geolocation.getCurrentPosition(
        pos => {
          const { latitude, longitude } = pos.coords;
          L.marker([latitude, longitude], {color: "blue"}).addTo(map).bindPopup("你在这里").openPopup();
          map.setView([latitude, longitude], 15);
        },
        err => {
          showError("⚠️ 定位失败：" + err.message);
        }
      );
    }

    function showError(msg) {
      document.getElementById("error").textContent = msg;
    }
  </script>
</body>
</html>
