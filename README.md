osaka food picked by maureen

<html lang="zh">
<head>
  <meta charset="UTF-8">
  <title>大阪美食地图 🍣🍤🍰</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <!-- Leaflet -->
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/leaflet@1.9.4/dist/leaflet.css"/>
  <script src="https://cdn.jsdelivr.net/npm/leaflet@1.9.4/dist/leaflet.js"></script>

  <style>
    html, body {
      margin: 0;
      padding: 0;
      height: 100%;
      width: 100%;
      font-family: sans-serif;
      overflow: hidden;
    }

    #map { height: 100%; width: 100%; }

    /* 抽屉样式 */
    #sidebar {
      position: fixed;
      top: 0;
      right: -300px;
      width: 300px;
      height: 100%;
      background: #fff;
      box-shadow: -2px 0 8px rgba(0,0,0,0.2);
      padding: 12px;
      overflow-y: auto;
      transition: right 0.3s ease;
      z-index: 1000;
    }

    #sidebar.open { right: 0; }

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

    button {
      padding: 6px 12px;
      border: none;
      border-radius: 6px;
      background: #0077cc;
      color: white;
      cursor: pointer;
      margin-bottom: 10px;
    }
    button:hover { background: #005fa3; }

    .emoji-marker { font-size: 24px; text-align: center; line-height: 30px; }
  </style>
</head>
<body>

  <div id="map"></div>

  <div id="sidebar">
    <button onclick="toggleSidebar()">关闭 ×</button>
    <h2>🍴 大阪美食清单</h2>
    <div id="places"></div>
    <button onclick="refreshLocation()">📍 刷新位置</button>
    <div id="error" style="color:red;margin-top:10px;"></div>
  </div>

  <button id="openBtn" onclick="toggleSidebar()" style="position: fixed; top: 20px; right: 20px; z-index:1001;">📋 菜单</button>

  <script>
    const destinations = [
      { name: "Sashisu", lat: 34.6984792, lng: 135.4997499, emoji: "🍣" },
      { name: "Kibitaki", lat: 34.6712265, lng: 135.5020764, emoji: "🍢" },
      { name: "Tempura Tarojiro", lat: 34.6678979, lng: 135.503683, emoji: "🍤" },
      { name: "宫田面儿", lat: 34.6738237, lng: 135.5042491, emoji: "🍜" },
      { name: "Canelé du Japon", lat: 34.6766352, lng: 135.5065096, emoji: "🍰" }
    ];

    let map = L.map("map").setView([34.6937, 135.5023], 14);
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "© OpenStreetMap"
    }).addTo(map);

    const placesDiv = document.getElementById("places");
    let currentPosition = null; 

    function calcDistance(lat1, lng1, lat2, lng2) {
      const R = 6371;
      const dLat = (lat2 - lat1) * Math.PI/180;
      const dLng = (lng2 - lng1) * Math.PI/180;
      const a = Math.sin(dLat/2)**2 +
                Math.cos(lat1*Math.PI/180) * Math.cos(lat2*Math.PI/180) *
                Math.sin(dLng/2)**2;
      const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
      return R * c;
    }

    function updateSidebar() {
      placesDiv.innerHTML = "";
      const sorted = destinations.slice();
      if(currentPosition) {
        sorted.forEach(d => {
          d.distance = calcDistance(currentPosition.lat, currentPosition.lng, d.lat, d.lng).toFixed(2);
        });
        sorted.sort((a,b) => a.distance - b.distance);
      }
      sorted.forEach(d => {
        const gmapUrl = `https://www.google.com/maps/dir/?api=1&destination=${d.lat},${d.lng}`;
        const distText = d.distance ? ` (${d.distance} km)` : "";
        const div = document.createElement("div");
        div.className = "place";
        div.innerHTML = `<span class="emoji">${d.emoji}</span><span>${d.name}${distText}</span><a href="${gmapUrl}" target="_blank">➡️</a>`;
        div.onclick = () => map.setView([d.lat, d.lng], 16);
        placesDiv.appendChild(div);
      });
    }

    destinations.forEach(d => {
      const gmapUrl = `https://www.google.com/maps/dir/?api=1&destination=${d.lat},${d.lng}`;
      const emojiIcon = L.divIcon({ className:"emoji-marker", html:d.emoji, iconSize:[30,30], iconAnchor:[15,15] });
      L.marker([d.lat,d.lng], {icon: emojiIcon})
        .addTo(map)
        .bindPopup(`${d.emoji} <b>${d.name}</b><br><a href="${gmapUrl}" target="_blank">在Google Maps导航</a>`);
    });

    function refreshLocation() {
      if(!navigator.geolocation) { showError("❌ 浏览器不支持定位"); return; }
      navigator.geolocation.getCurrentPosition(
        pos => {
          const { latitude, longitude } = pos.coords;
          currentPosition = { lat: latitude, lng: longitude };
          L.marker([latitude, longitude]).addTo(map).bindPopup("📍 你在这里").openPopup();
          map.setView([latitude, longitude], 15);
          updateSidebar();
          map.invalidateSize(); // 修复地图显示
        },
        err => showError("⚠️ 定位失败：" + err.message)
      );
    }

    const sidebar = document.getElementById("sidebar");
    function toggleSidebar() {
      sidebar.classList.toggle("open");
      setTimeout(() => { map.invalidateSize(); }, 300); // 抽屉动画结束后刷新地图
    }

    function showError(msg) { document.getElementById("error").textContent = msg; }

    window.onload = () => { setTimeout(()=>{ map.invalidateSize(); }, 100); };

    updateSidebar();
  </script>

</body>
</html>
