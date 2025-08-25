# osaka_food_map
osaka food may picked by maureen
<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <title>大阪旅行高级地图助手</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
  <style>
    body { margin:0; font-family: Arial, sans-serif; display: flex; height: 100vh; }
    #list { width: 30%; overflow-y: auto; border-right: 1px solid #ccc; padding: 10px; box-sizing: border-box; }
    #list h2 { margin-top: 0; }
    #map { flex: 1; }
    .place-item { padding: 8px; margin: 6px 0; border-radius: 6px; background:#f0f8ff; cursor:pointer; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
    .place-item:hover { background: #e0f0ff; }
    button { margin-top: 5px; padding: 4px 8px; font-size: 0.9em; }
  </style>
</head>
<body>
  <div id="list">
    <h2>大阪餐厅/景点列表</h2>
    <button onclick="locateUser()">刷新位置</button>
    <div id="places">正在加载...</div>
  </div>
  <div id="map"></div>

  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <script>
    // 想去的地点（可自行添加更多）
    const destinations = [
      { name: "Sashisu", lat: 34.6984792, lng: 135.4997499, type: "餐厅" },
      { name: "Kibitaki", lat: 34.6712265, lng: 135.5020764, type: "餐厅" },
      { name: "Tempura Tarojiro", lat: 34.6678979, lng: 135.503683, type: "餐厅" },
      { name: "宫田面儿", lat: 34.6738237, lng: 135.5042491, type: "餐厅" },
      { name: "Canelé du Japon", lat: 34.6766352, lng: 135.5065096, type: "甜品" }
    ];

    const map = L.map('map').setView([34.675, 135.503], 14);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 19, attribution: '© OpenStreetMap' }).addTo(map);

    let userMarker;
    let markers = [];

    function calcDistance(lat1, lon1, lat2, lon2) {
      const R = 6371;
      const dLat = (lat2 - lat1) * Math.PI / 180;
      const dLon = (lon2 - lon1) * Math.PI / 180;
      const a = Math.sin(dLat/2)**2 + Math.cos(lat1*Math.PI/180) * Math.cos(lat2*Math.PI/180) * Math.sin(dLon/2)**2;
      const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
      return (R * c).toFixed(2);
    }

    function updateListAndMarkers(userLat, userLng) {
      // 清除旧标记
      markers.forEach(m => map.removeLayer(m));
      markers = [];

      // 计算距离
      const results = destinations.map(d => {
        const dist = calcDistance(userLat, userLng, d.lat, d.lng);
        return { ...d, distance: dist };
      });

      // 按距离排序
      results.sort((a,b)=>a.distance - b.distance);

      const container = document.getElementById("places");
      container.innerHTML = "";
      results.forEach((place,i)=>{
        const div = document.createElement("div");
        div.className = "place-item";
        div.innerHTML = `<b>${place.name}</b> (${place.type})<br>距离: ${place.distance} km<br>
          <button onclick="navigateTo(${place.lat},${place.lng})">导航</button>`;
        div.onclick = ()=>{ markers[i].openPopup(); map.setView([place.lat, place.lng], 16); };
        container.appendChild(div);

        // 图标区分类型
        let iconUrl = place.type==="餐厅"?"https://cdn-icons-png.flaticon.com/512/1046/1046784.png":"https://cdn-icons-png.flaticon.com/512/1046/1046786.png";
        const icon = L.icon({ iconUrl, iconSize:[30,30] });
        const marker = L.marker([place.lat, place.lng], {icon}).addTo(map)
          .bindPopup(`<b>${place.name}</b><br>${place.type}<br>距离: ${place.distance} km<br>
            <a href="https://www.google.com/maps/dir/?api=1&destination=${place.lat},${place.lng}" target="_blank">导航</a>`);
        markers.push(marker);
      });
    }

    function navigateTo(lat,lng) { window.open(`https://www.google.com/maps/dir/?api=1&destination=${lat},${lng}`, "_blank"); }

    function locateUser() {
      if(navigator.geolocation){
        navigator.geolocation.getCurrentPosition(pos=>{
          const lat = pos.coords.latitude;
          const lng = pos.coords.longitude;

          if(userMarker) userMarker.setLatLng([lat,lng]);
          else userMarker = L.marker([lat,lng], {icon:L.icon({iconUrl:'https://cdn-icons-png.flaticon.com/512/64/64113.png', iconSize:[30,30]})}).addTo(map).bindPopup("你的位置").openPopup();

          map.setView([lat,lng],15);
          updateListAndMarkers(lat,lng);
        }, err=>{ alert("定位失败："+err.message); });
      } else { alert("你的浏览器不支持定位。"); }
    }

    // 页面加载时先定位
    locateUser();
  </script>
</body>
</html>
