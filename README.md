# Boss-number1
World changing 

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="theme-color" content="#071b2c">
<meta name="description" content="AirAlert air quality dashboard for PM2.5, PM10 and AQI.">
<title>AirAlert | Smart Pollution Monitor</title>
<style>
:root {
  --bg:#071b2c; --panel:#102a40; --text:#f1f7fc;
  --muted:#a7bfd0; --accent:#47e0b1; --line:#27445a;
}
* { box-sizing:border-box; }
body {
  margin:0; font-family:Arial,sans-serif; color:var(--text);
  background:linear-gradient(145deg,#071b2c,#0c3043);
  min-height:100vh;
}
header, main, footer { max-width:1000px; margin:auto; padding:20px; }
header { display:flex; justify-content:space-between; align-items:center; gap:12px; }
.brand { font-size:25px; font-weight:800; }
.brand span { color:var(--accent); }
.tag { font-size:12px; color:var(--muted); }
h1 { font-size:clamp(27px,6vw,43px); margin:20px 0 8px; }
p { line-height:1.6; }
.muted { color:var(--muted); }
.panel {
  background:rgba(16,42,64,.94); border:1px solid var(--line);
  border-radius:18px; padding:20px; margin:16px 0;
}
.controls { display:flex; flex-wrap:wrap; gap:10px; margin-top:16px; }
input,button {
  padding:13px; border-radius:10px; font-size:15px;
}
input {
  min-width:0; width:100%; background:#071b2c; color:white;
  border:1px solid var(--line);
}
.field { flex:1 1 130px; }
label { display:block; color:var(--muted); font-size:13px; margin-bottom:7px; }
button {
  border:0; cursor:pointer; font-weight:bold;
  background:var(--accent); color:#06251f;
}
button.secondary { background:#24455c; color:white; }
button:disabled { opacity:.6; cursor:wait; }
.grid {
  display:grid; grid-template-columns:repeat(3,minmax(0,1fr)); gap:14px;
}
.metric { background:#0a2032; border-radius:14px; padding:17px; }
.metric h3 { margin:0 0 14px; font-size:14px; color:var(--muted); }
.value { font-size:clamp(25px,5vw,36px); font-weight:800; overflow-wrap:anywhere; }
.unit { color:var(--muted); font-size:12px; }
.status { font-size:20px; font-weight:bold; margin:10px 0; }
.good { color:#47e0b1; }
.moderate { color:#ffd166; }
.bad { color:#ff9b75; }
.hazard { color:#ff6577; }
.note { font-size:13px; color:var(--muted); }
#message { margin-top:12px; font-size:14px; }
footer { font-size:12px; color:var(--muted); padding-bottom:35px; }
@media(max-width:560px) {
  header { padding:16px; }
  main { padding:12px; }
  .grid { grid-template-columns:1fr; }
  .panel { padding:16px; }
  .controls button { flex:1 1 100%; }
}
</style>
</head>
<body>
<header>
  <div class="brand">🌍 Air<span>Alert</span></div>
  <div class="tag">SMART AIR MONITOR</div>
</header>

<main>
  <section>
    <p class="tag">BREATHE SMARTER • LIVE DATA WHEN AVAILABLE</p>
    <h1>Know the air around you.</h1>
    <p class="muted">Monitor air quality estimates and particulate pollution from one simple dashboard.</p>
  </section>

  <section class="panel">
    <h2>📍 Choose your location</h2>
    <p class="note">Enter latitude and longitude, or allow device location.</p>
    <div class="controls">
      <div class="field">
        <label for="lat">Latitude</label>
        <input id="lat" type="number" step="any" min="-90" max="90" value="25.21">
      </div>
      <div class="field">
        <label for="lon">Longitude</label>
        <input id="lon" type="number" step="any" min="-180" max="180" value="84.98">
      </div>
    </div>
    <div class="controls">
      <button id="loadBtn" onclick="loadAir()">Get Air Quality</button>
      <button class="secondary" onclick="useLocation()">Use My Location</button>
      <button class="secondary" onclick="showDemo()">Try Demo Data</button>
    </div>
    <div id="message" role="status" aria-live="polite">Ready to check air quality.</div>
  </section>

  <section class="panel">
    <div class="tag">AIR QUALITY OVERVIEW</div>
    <h2 id="place">Selected location</h2>
    <div id="status" class="status moderate">Waiting for data</div>
    <p id="advice" class="muted">Load data to see an air quality summary.</p>
    <div class="grid">
      <div class="metric">
        <h3>US AQI</h3>
        <div class="value" id="aqi">--</div>
        <div class="unit">Index (0–500+)</div>
      </div>
      <div class="metric">
        <h3>PM2.5</h3>
        <div class="value" id="pm25">--</div>
        <div class="unit">µg/m³</div>
      </div>
      <div class="metric">
        <h3>PM10</h3>
        <div class="value" id="pm10">--</div>
        <div class="unit">µg/m³</div>
      </div>
    </div>
    <p class="note" id="updated">No data loaded yet.</p>
    <p class="note" id="source">Source: Not connected</p>
  </section>

  <section class="panel">
    <h2>🛡️ How to use this information</h2>
    <p>When the air quality index is high, consider reducing strenuous outdoor activity. People sensitive to air pollution should follow local health guidance.</p>
    <p class="note">AQI categories in this demo use the US AQI scale. Values from an external model may differ from official local monitoring stations.</p>
  </section>
</main>

<footer>
  AirAlert Student Project • Informational use only.<br>
  This website does not measure air directly. Real sensor monitoring requires compatible hardware and integration.
</footer>

<script>
const $ = id => document.getElementById(id);

function setMessage(text) {
  $("message").textContent = text;
}

function classify(aqi) {
  if (aqi <= 50) return ["Good", "good",
    "Air quality is generally satisfactory for most people."];
  if (aqi <= 100) return ["Moderate", "moderate",
    "Some sensitive people may need extra caution."];
  if (aqi <= 150) return ["Unhealthy for Sensitive Groups", "moderate",
    "Sensitive groups should consider reducing strenuous outdoor activity."];
  if (aqi <= 200) return ["Unhealthy", "bad",
    "Consider reducing prolonged or strenuous outdoor activity."];
  if (aqi <= 300) return ["Very Unhealthy", "hazard",
    "Avoid strenuous outdoor activity and follow local health advice."];
  return ["Hazardous", "hazard",
    "Avoid outdoor exertion and follow official local health guidance."];
}

function render(data, source) {
  const aqi = Number(data.aqi);
  const pm25 = Number(data.pm25);
  const pm10 = Number(data.pm10);

  if (![aqi, pm25, pm10].every(Number.isFinite)) {
    throw new Error("Data values are missing or invalid.");
  }

  $("aqi").textContent = Math.round(aqi);
  $("pm25").textContent = pm25.toFixed(1);
  $("pm10").textContent = pm10.toFixed(1);

  const [name, cls, advice] = classify(aqi);
  $("status").textContent = name;
  $("status").className = "status " + cls;
  $("advice").textContent = advice;
  $("place").textContent = data.place || "Selected location";
  $("updated").textContent = "Updated: " + new Date().toLocaleString();
  $("source").textContent = "Source: " + source;
}

async function loadAir() {
  const lat = Number($("lat").value);
  const lon = Number($("lon").value);

  if (!Number.isFinite(lat) || lat < -90 || lat > 90 ||
      !Number.isFinite(lon) || lon < -180 || lon > 180) {
    setMessage("Please enter valid latitude and longitude.");
    return;
  }

  $("loadBtn").disabled = true;
  setMessage("Loading air quality data...");

  try {
    const url = "https://air-quality-api.open-meteo.com/v1/air-quality"
      + "?latitude=" + encodeURIComponent(lat)
      + "&longitude=" + encodeURIComponent(lon)
      + "&current=us_aqi,pm2_5,pm10"
      + "&timezone=auto";

    const response = await fetch(url);
    if (!response.ok) throw new Error("API request failed (" + response.status + ")");

    const result = await response.json();
    const current = result.current;

    if (!current || current.us_aqi == null ||
        current.pm2_5 == null || current.pm10 == null) {
      throw new Error("Current data is unavailable for this location.");
    }

    render({
      aqi: current.us_aqi,
      pm25: current.pm2_5,
      pm10: current.pm10,
      place: lat.toFixed(2) + ", " + lon.toFixed(2)
    }, "Open-Meteo air quality model");

    setMessage("Air quality data loaded successfully.");
  } catch (error) {
    setMessage("Could not load live data: " + error.message +
      " Try again or use Demo Data.");
  } finally {
    $("loadBtn").disabled = false;
  }
}

function showDemo() {
  render({
    aqi: 78,
    pm25: 22.4,
    pm10: 41.8,
    place: "Sample Location (not live)"
  }, "DEMO DATA — NOT A REAL MEASUREMENT");
  setMessage("Demo loaded. These numbers are examples only.");
}

function useLocation() {
  if (!navigator.geolocation) {
    setMessage("Location is not supported by this browser.");
    return;
  }

  setMessage("Requesting location permission...");

  navigator.geolocation.getCurrentPosition(
    position => {
      $("lat").value = position.coords.latitude.toFixed(5);
      $("lon").value = position.coords.longitude.toFixed(5);
      loadAir();
    },
    error => {
      setMessage("Location unavailable or permission denied. Enter coordinates manually.");
    },
    { enableHighAccuracy: false, timeout: 10000, maximumAge: 300000 }
  );
}

// Load a demo first so the dashboard can be tested without an API.
showDemo();
</script>
</body>
</html>
