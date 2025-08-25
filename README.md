osaka food picked by maureen

<html lang="zh">
<head>
  <meta charset="UTF-8" />
  <title>大阪美食地图 🍜🍦🍣</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />

  <!-- Leaflet 地图库 -->
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://cdn.jsdelivr.net/npm/leaflet@1.9.4/dist/leaflet.js"></script>

  <style>
    html, body {
      margin: 0;
      padding: 0;
      height: 100%;
      width: 100%;
      display: flex;
      flex-direction: row;  /* 默认横向排列 */
      font-family: sans-serif;
    }

    #map {
      flex: 3;
      height: 100%;
    }

    #sidebar {
      flex: 1;
      overflow-y: auto;
      padding: 12px;
      box-shadow: -2px 0 5px rgba(0,0,0,0.1);
      background: #fff;
      min-width: 220px;
    }

    .place {
      margin: 10px 0;
      padding: 10px;
      border-radius: 10px;
      background: #f9f9f9;
      cursor: pointer;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .place:hover { background: #ececec; }
    .emoji { font-size: 20px; margin-right: 6px; }

    h2 { margin-top: 0; }
    #error { color: red; font-size: 14px; margin-top: 10px; }
    a { text-decoration: none; color: #0077cc; }
    button { margin-top: 12px; padding: 6px 10px; border: none; border-radius: 6px; background: #0077cc; color: white; cursor: pointer; }
    button:hover { background: #005fa3; }

    /* 📱 小屏幕优化：上下排列 */
    @media (max-width: 768px) {
      html, body { flex-direction: column; }
      #map { height: 60vh; width: 100%; }
      #sidebar { height: 40vh; width: 100%; box-shadow: 0 -2px 5px rgba(0,0,0,0.1); }
    }

    /* 地图 emoji 样式 */
    .emoji-marker {
      font-size: 24px;
      text-align: center;
      line-height: 30px;
    }
  </style>
</head>
<body>
  <div id="map"></div>
  <div id="sidebar">
    <h2>🍴 我的大阪清单</h2>
    <div id="places"></div>
    <div id="error"></div>
    <button onclick="refreshLocation()">📍 刷新位置</button>
  </div>

  <script>
    const destinations = [
      { name: "Sashisu", lat: 34.6984792, lng: 135.4997499, emoji: "🍣" },
      { name: "Kibitaki", lat: 34.6712265, lng: 135.5020764, emoji: "🍢" },
      { name: "Tempura Tarojiro", lat: 34.6678979, lng: 135.503683, emoji: "🍤" },
      { name: "宫田面儿", lat: 34.6738237, lng: 135.5042491, emoji: "🍜" },
      { name: "Canelé du Japon", lat: 34.6766352, lng: 135.5065096, emoji: "🍰" }
    ];

    // 初始化地图
    let map = L.map("map").setView([34.6937, 135.5023], 14);
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "© OpenStreetMap"
    }).addTo(map);

    const placesDiv = document.getElementById("places");

    destinations.forEach(d => {
      const gmapUrl = `https://www.google.com/maps/dir/?api=1&destination=${d.lat},${d.lng}`;

      // 侧边栏
      const div = document.createElement("div");
      div.className = "place";
      div.innerHTML = `<span class="emoji">${d.emoji}</span><span>${d.name}</span><a href="${gmapUrl}" target="_blank">➡️</a>`;
      div.onclick = () => map.setView([d.lat, d.lng], 16);
      placesDiv.appendChild(div);

      // 地图 emoji 标记
      const emojiIcon = L.divIcon({
        className: "emoji-marker",
        html: d.emoji,
        iconSize: [30, 30],
        iconAnchor: [15, 15]
      });

      L.marker([d.lat, d.lng], { icon: emojiIcon })
        .addTo(map)
        .bindPopup(`${d.emoji} <b>${d.name}</b><br><a href="${gmapUrl}" target="_blank">在Google Maps导航</a>`);
    });

    // 定位功能
    function refreshLocation() {
      if (!navigator.geolocation) {
        showError("❌ 浏览器不支持定位");
        return;
      }
      navigator.geolocation.getCurrentPosition(
        pos => {
          const { latitude, longitude } = pos.coords;
          L.marker([latitude, longitude]).addTo(map).bindPopup("📍 你在这里").openPopup();
          map.setView([latitude, longitude], 15);
        },
        err => showError("⚠️ 定位失败：" + err.message)
      );
    }

    function showError(msg) {
      document.getElementById("error").textContent = msg;
    }
  </script>
</body>
</html>
