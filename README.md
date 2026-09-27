#도봉구 안심지도
모두가 같이 만드는 안심지도
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>우리 동네 안심 안전 지도</title>
  https://cdn.jsdelivr.net/npm/leaflet@1.9.4/dist/leaflet.css
  <style>
    * { box-sizing: border-box; }
    html, body { margin: 0; height: 100%; font-family: sans-serif; }
    .app { height: 100%; display: grid; grid-template-columns: 360px 1fr; }
    .panel { padding: 16px; overflow: auto; border-right: 1px solid #ddd; }
    h1 { font-size: 18px; margin: 0 0 12px; }
    .filters, .row { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px; }
    button, select, input, textarea {
      font: inherit; padding: 8px; border: 1px solid #ccc; border-radius: 8px;
    }
    button { cursor: pointer; background: #fff; }
    button.active, .submit { background: #1f7a5c; color: #fff; border: 0; }
    form { display: grid; gap: 8px; margin-bottom: 16px; }
    #map { height: 100%; }
    .card { border: 1px solid #ddd; border-radius: 10px; padding: 10px; margin-bottom: 8px; cursor: pointer; }
    .hint { color: #666; font-size: 12px; }
    @media (max-width: 800px) {
      .app { grid-template-columns: 1fr; grid-template-rows: 50% 50%; }
    }
  </style>
</head>
<body>
  <div class="app">
    <aside class="panel">
      <h1>우리 동네 안심 안전 지도</h1>
      <div class="row">
        <button id="share-btn" type="button">이 사이트 링크 복사</button>
      </div>
      <div class="filters">
        <button class="chip active" data-filter="all" type="button">전체</button>
        <button class="chip" data-filter="route" type="button">안심 귀갓길</button>
        <button class="chip" data-filter="store" type="button">24시 편의점</button>
        <button class="chip" data-filter="safe" type="button">치안 우수</button>
      </div>
      <form id="report-form">
        <select id="category">
          <option value="route">안심 귀갓길</option>
          <option value="store">24시 편의점</option>
          <option value="safe">치안 우수 지역</option>
        </select>
        <input id="name" required placeholder="장소 이름" />
        <textarea id="address" required placeholder="설명. 지도를 클릭해 위치를 찍으세요."></textarea>
        <input id="nickname" placeholder="닉네임(선택)" />
        <div class="hint" id="coord-hint">지도를 클릭하면 위치가 저장됩니다.</div>
        <input type="hidden" id="lat" />
        <input type="hidden" id="lng" />
        <button class="submit" type="submit">제보하기</button>
      </form>
      <div id="list"></div>
    </aside>
    <div id="map"></div>
  </div>
https://cdn.jsdelivr.net/npm/leaflet@1.9.4/dist/leaflet.js
  <script>
    const CENTER = [37.6686, 127.0466];
    const KEY = "ansim-map-v3";
    const LABELS = { route: "안심 귀갓길", store: "24시 편의점", safe: "치안 우수" };
    const COLORS = { route: "#1f7a5c", store: "#c56a1a", safe: "#1d5f8a" };
    const seed = [
      { id: "1", category: "safe", name: "서울도봉경찰서", address: "노해로 403", nickname: "기본", lat: 37.65305, lng: 127.04755 },
      { id: "2", category: "safe", name: "서울도봉소방서", address: "도봉로 666", nickname: "기본", lat: 37.66365, lng: 127.04310 },
      { id: "3", category: "route", name: "창동역 큰길", address: "불이 밝은 귀갓길", nickname: "기본", lat: 37.6532, lng: 127.0473 },
      { id: "4", category: "store", name: "창동역 24시 편의점", address: "창동역 일대", nickname: "기본", lat: 37.6538, lng: 127.0468 },
      { id: "5", category: "store", name: "쌍문역 24시 편의점", address: "쌍문역 주변", nickname: "기본", lat: 37.6489, lng: 127.0342 }
    ];

    const map = L.map("map").setView(CENTER, 14);
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", { maxZoom: 19 }).addTo(map);
    const markers = L.layerGroup().addTo(map);
    let filter = "all";
    let reports = JSON.parse(localStorage.getItem(KEY) || "null") || seed.slice();

    function render() {
      markers.clearLayers();
      const list = document.getElementById("list");
      list.innerHTML = "";
      reports.filter(r => filter === "all" || r.category === filter).forEach(r => {
        const marker = L.circleMarker([r.lat, r.lng], {
          radius: 8, color: COLORS[r.category], fillColor: COLORS[r.category], fillOpacity: 0.9
        }).bindPopup(`<b>${r.name}</b><br>${LABELS[r.category]}<br>${r.address}`);
        markers.addLayer(marker);
        const card = document.createElement("div");
        card.className = "card";
        card.innerHTML = `<b>${r.name}</b><div class="hint">${LABELS[r.category]} · ${r.address}</div>`;
        card.onclick = () => { map.setView([r.lat, r.lng], 16); marker.openPopup(); };
        list.appendChild(card);
      });
    }

    map.on("click", e => {
      document.getElementById("lat").value = e.latlng.lat;
      document.getElementById("lng").value = e.latlng.lng;
      document.getElementById("coord-hint").textContent = "위치 선택됨";
    });

    document.getElementById("report-form").onsubmit = e => {
      e.preventDefault();
      const lat = parseFloat(document.getElementById("lat").value);
      const lng = parseFloat(document.getElementById("lng").value);
      if (Number.isNaN(lat) || Number.isNaN(lng)) { alert("지도를 먼저 클릭하세요."); return; }
      reports.unshift({
        id: Date.now(),
        category: document.getElementById("category").value,
        name: document.getElementById("name").value,
        address: document.getElementById("address").value,
        nickname: document.getElementById("nickname").value || "익명",
        lat, lng
      });
      localStorage.setItem(KEY, JSON.stringify(reports));
      e.target.reset();
      render();
    };

    document.querySelectorAll(".chip").forEach(btn => {
      btn.onclick = () => {
        document.querySelectorAll(".chip").forEach(b => b.classList.remove("active"));
        btn.classList.add("active");
        filter = btn.dataset.filter;
        render();
      };
    });

    document.getElementById("share-btn").onclick = async () => {
      const url = location.href.split("#")[0];
      try { await navigator.clipboard.writeText(url); alert("링크를 복사했습니다."); }
      catch { prompt("이 주소를 복사하세요", url); }
    };

    render();
  </script>
</body>
</html>