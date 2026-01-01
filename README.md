<<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Green-Stream AI | Solapur Smart City Node</title>
    
    <!-- Core AI & UI Dependencies -->
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Free Map Engine (Leaflet) -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;600;800&family=JetBrains+Mono&display=swap');
        :root { --brand-green: #00f58d; --brand-red: #ff4444; --bg-dark: #080a0f; --card-dark: #12151c; }
        body { background-color: var(--bg-dark); color: #e2e8f0; font-family: 'Plus Jakarta Sans', sans-serif; overflow-x: hidden; }
        .mono { font-family: 'JetBrains+Mono', monospace; }
        .glass-morphism { background: rgba(18, 21, 28, 0.8); backdrop-filter: blur(12px); border: 1px solid rgba(255, 255, 255, 0.05); }
        
        #video-container { 
            position: relative; 
            border-radius: 1.5rem; 
            overflow: hidden; 
            background: #000; 
            border: 1px solid rgba(0, 245, 141, 0.15);
            min-height: 450px;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        
        canvas#detection-overlay { 
            position: absolute; 
            top: 0; 
            left: 0; 
            width: 100%; 
            height: 100%; 
            z-index: 5;
            object-fit: contain;
        }
        
        #webcam { width: 100%; height: 100%; object-fit: cover; z-index: 1; }
        #map { width: 100%; height: 100%; z-index: 1; border-radius: 1.5rem; }
        
        .leaflet-container { background: #080a0f !important; }
        .btn-glow:hover { box-shadow: 0 0 15px var(--brand-green); transform: translateY(-2px); }
        .custom-scrollbar::-webkit-scrollbar { width: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #334155; border-radius: 10px; }
        
        .critical-alert { animation: pulse-red 1.5s infinite; }
        @keyframes pulse-red {
            0% { box-shadow: 0 0 0 0 rgba(255, 68, 68, 0.4); }
            70% { box-shadow: 0 0 0 20px rgba(255, 68, 68, 0); }
            100% { box-shadow: 0 0 0 0 rgba(255, 68, 68, 0); }
        }
    </style>
</head>
<body class="p-4 lg:p-6">

    <!-- Header Section -->
    <header class="max-w-[1600px] mx-auto flex flex-col md:flex-row justify-between items-center mb-6 gap-4">
        <div class="flex items-center gap-4">
            <div class="p-3 bg-[#00f58d] rounded-2xl shadow-[0_0_20px_rgba(0,245,141,0.4)]">
                <svg class="w-8 h-8 text-black" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10"></path></svg>
            </div>
            <div>
                <h1 class="text-2xl font-extrabold tracking-tight text-white uppercase italic">Green-Stream <span class="text-[#00f58d]">AI</span></h1>
                <p class="text-xs text-slate-500 font-semibold uppercase tracking-widest">Solapur Municipal Command Center</p>
            </div>
        </div>
        <div id="connection-status" class="glass-morphism px-6 py-2 rounded-full flex items-center gap-3">
            <div id="status-dot" class="w-2 h-2 bg-yellow-500 rounded-full"></div>
            <span id="status-text" class="text-[10px] font-bold uppercase tracking-widest text-slate-400">Loading AI Engine...</span>
        </div>
    </header>

    <main class="max-w-[1600px] mx-auto grid grid-cols-1 lg:grid-cols-12 gap-6">
        <!-- AI Vision Panel -->
        <div class="lg:col-span-7 space-y-6">
            <div id="video-container" class="glass-morphism shadow-2xl">
                <video id="webcam" autoplay muted playsinline></video>
                <canvas id="detection-overlay"></canvas>
                
                <div class="absolute bottom-6 left-6 right-6 flex justify-between items-center z-20">
                    <div class="flex gap-3">
                        <button onclick="GreenStream.initCamera()" class="btn-glow bg-[#00f58d] text-black px-6 py-3 rounded-xl font-extrabold text-xs shadow-xl transition-all uppercase">Live Monitor</button>
                        <input type="file" id="fileInput" class="hidden" accept="image/*" onchange="GreenStream.handleUpload(event)">
                        <button onclick="document.getElementById('fileInput').click()" class="bg-white/10 backdrop-blur-md text-white px-6 py-3 rounded-xl font-bold text-xs hover:bg-white/20 transition-all uppercase">Analyze Scene</button>
                    </div>
                    <button onclick="GreenStream.exportData()" class="bg-blue-600/20 text-blue-400 border border-blue-500/30 px-4 py-3 rounded-xl font-bold text-[10px] uppercase">Export Analytics</button>
                </div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div id="index-card" class="glass-morphism p-5 rounded-3xl border-l-4 border-[#00f58d] transition-all duration-500">
                    <p class="text-[10px] text-slate-500 font-bold uppercase mb-2">Pollution Index</p>
                    <h2 id="ui-index" class="text-4xl font-black">0%</h2>
                    <div class="w-full bg-slate-800 h-1 mt-3 rounded-full overflow-hidden">
                        <div id="ui-progress" class="bg-[#00f58d] h-full transition-all" style="width: 0%"></div>
                    </div>
                </div>
                <div class="glass-morphism p-5 rounded-3xl">
                    <p class="text-[10px] text-slate-500 font-bold uppercase mb-2">Waste Items</p>
                    <h2 id="ui-count" class="text-4xl font-black text-[#ff4444]">00</h2>
                    <p class="text-[9px] text-slate-500 mt-2 mono uppercase">Density: <span id="ui-density">0.00</span></p>
                </div>
                <div class="glass-morphism p-5 rounded-3xl">
                    <p class="text-[10px] text-slate-500 font-bold uppercase mb-2">Node Priority</p>
                    <h2 id="ui-priority" class="text-2xl font-black text-slate-600 tracking-tighter uppercase">IDLE</h2>
                    <p id="ui-status-text" class="text-[9px] text-slate-400 mt-2 uppercase">Awaiting Input...</p>
                </div>
            </div>

            <div class="glass-morphism p-6 rounded-3xl">
                <h3 class="text-xs font-bold uppercase tracking-widest mb-4 flex justify-between">
                    System Telemetry <span id="inference-speed" class="mono text-blue-400">0ms</span>
                </h3>
                <div id="activity-log" class="h-24 overflow-y-auto space-y-2 mono text-[11px] text-slate-400 custom-scrollbar"></div>
            </div>
        </div>

        <!-- Analytics & Geography -->
        <div class="lg:col-span-5 space-y-6">
            <div class="glass-morphism rounded-3xl overflow-hidden h-[300px]">
                <div id="map"></div>
            </div>

            <div class="glass-morphism rounded-3xl overflow-hidden flex flex-col h-[400px]">
                <div class="p-5 border-b border-white/5 flex justify-between items-center bg-white/5">
                    <h3 class="font-bold text-sm uppercase tracking-widest text-[#00f58d]">Violation Workflow</h3>
                    <button onclick="GreenStream.forceReport()" class="bg-red-600 px-3 py-1.5 rounded-lg text-[9px] font-bold text-white uppercase shadow-lg">Push Report</button>
                </div>
                <div id="incident-feed" class="flex-1 p-4 space-y-3 custom-scrollbar overflow-y-auto">
                    <div class="text-center text-slate-600 text-xs py-10 italic">No Active Violations Stored</div>
                </div>
            </div>

            <div class="glass-morphism p-4 rounded-3xl">
                <p class="text-[10px] text-slate-500 font-bold uppercase mb-2">Geo-Location Lock</p>
                <div class="flex justify-between items-center mono text-[10px]">
                    <span id="geo-lat" class="text-white">LAT: 17.6599</span>
                    <span id="geo-lng" class="text-white">LNG: 75.9064</span>
                    <span class="text-[#00f58d] tracking-tighter uppercase font-bold">SOLAPUR_NODE_01</span>
                </div>
            </div>
        </div>
    </main>

    <script>
        const GreenStream = (() => {
            let model, video, canvas, ctx, leafletMap;
            let isDetecting = false;
            // SET LOCATION TO SOLAPUR
            let userLocation = { lat: 17.6599, lng: 75.9064 }; 
            const WASTE_LABELS = ['bottle', 'cup', 'wine glass', 'backpack', 'handbag', 'suitcase', 'bowl', 'garbage', 'trash', 'can', 'plastic'];

            return {
                async init() {
                    video = document.getElementById('webcam');
                    canvas = document.getElementById('detection-overlay');
                    ctx = canvas.getContext('2d');
                    this.log("Node Initializing at Solapur Hub...", "blue");

                    try {
                        model = await cocoSsd.load();
                        this.log("AI Ready for Solapur Monitoring.", "green");
                        document.getElementById('status-dot').className = "w-2 h-2 bg-[#00f58d] rounded-full";
                        document.getElementById('status-text').innerText = "SYSTEM_ONLINE";
                        document.getElementById('status-text').style.color = "#00f58d";
                    } catch (e) { this.log("AI Load Error.", "red"); }

                    this.initMap();
                },

                initMap() {
                    this.log("Connecting to Solapur Map Node...", "blue");
                    
                    leafletMap = L.map('map', { zoomControl: false, attributionControl: false }).setView([userLocation.lat, userLocation.lng], 13);

                    L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', { maxZoom: 19 }).addTo(leafletMap);

                    L.circle([userLocation.lat, userLocation.lng], {
                        color: '#00f58d', fillColor: '#00f58d', fillOpacity: 0.4, radius: 800
                    }).addTo(leafletMap).bindPopup("<b>Solapur Command Center</b>").openPopup();

                    if (navigator.geolocation) {
                        navigator.geolocation.getCurrentPosition(pos => {
                            userLocation = { lat: pos.coords.latitude, lng: pos.coords.longitude };
                            document.getElementById('geo-lat').innerText = `LAT: ${userLocation.lat.toFixed(4)}`;
                            document.getElementById('geo-lng').innerText = `LNG: ${userLocation.lng.toFixed(4)}`;
                            leafletMap.setView([userLocation.lat, userLocation.lng], 15);
                            L.marker([userLocation.lat, userLocation.lng]).addTo(leafletMap);
                            this.log("GPS Lock Confirmed.", "green");
                        });
                    }
                },

                async initCamera() {
                    isDetecting = true;
                    video.style.display = "block";
                    const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' } });
                    video.srcObject = stream;
                    this.log("Live Sensor Unit Active.", "green");
                    this.detectLoop();
                },

                async handleUpload(e) {
                    const file = e.target.files[0];
                    if (!file) return;
                    isDetecting = false;
                    if (video.srcObject) { video.srcObject.getTracks().forEach(t => t.stop()); video.srcObject = null; }
                    video.style.display = "none"; 

                    const reader = new FileReader();
                    reader.onload = (event) => {
                        const img = new Image();
                        img.onload = async () => {
                            const containerW = document.getElementById('video-container').clientWidth;
                            canvas.width = containerW;
                            canvas.height = (img.height / img.width) * containerW;
                            ctx.clearRect(0, 0, canvas.width, canvas.height);
                            ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                            this.log("Analyzing Static Frame...", "blue");
                            const predictions = await model.detect(img);
                            this.processLogic(predictions, img.width, img.height, true, img);
                        };
                        img.src = event.target.result;
                    };
                    reader.readAsDataURL(file);
                },

                async detectLoop() {
                    if (!isDetecting) return;
                    const start = performance.now();
                    const predictions = await model.detect(video);
                    document.getElementById('inference-speed').innerText = `${Math.round(performance.now() - start)}ms`;
                    this.processLogic(predictions, video.videoWidth, video.videoHeight, false, null);
                    requestAnimationFrame(() => this.detectLoop());
                },

                processLogic(predictions, origW, origH, isStatic, sourceImg) {
                    if (!isStatic) {
                        canvas.width = video.clientWidth;
                        canvas.height = video.clientHeight;
                        ctx.clearRect(0, 0, canvas.width, canvas.height);
                    } else {
                        ctx.clearRect(0, 0, canvas.width, canvas.height);
                        ctx.drawImage(sourceImg, 0, 0, canvas.width, canvas.height);
                    }

                    let wasteCount = 0;
                    let totalArea = 0;
                    const sx = canvas.width / origW;
                    const sy = canvas.height / origH;

                    predictions.forEach(p => {
                        if (WASTE_LABELS.includes(p.class.toLowerCase()) && p.score > 0.4) {
                            wasteCount++;
                            const [x, y, w, h] = p.bbox;
                            totalArea += (w * h);
                            // DRAW RED BOXES
                            ctx.strokeStyle = '#ff4444';
                            ctx.lineWidth = 4;
                            ctx.strokeRect(x * sx, y * sy, w * sx, h * sy);
                            ctx.fillStyle = '#ff4444';
                            ctx.font = 'bold 12px monospace';
                            ctx.fillText(p.class.toUpperCase(), x*sx, (y*sy > 20) ? y*sy - 5 : 10);
                        }
                    });

                    const areaRatio = (totalArea / (origW * origH)) * 100;
                    const pollutionScore = Math.min(100, Math.round((wasteCount * 10) + (areaRatio * 2)));
                    this.updateUI(wasteCount, pollutionScore, areaRatio);
                },

                updateUI(count, score, density) {
                    document.getElementById('ui-index').innerText = score + "%";
                    document.getElementById('ui-progress').style.width = score + "%";
                    document.getElementById('ui-count').innerText = count.toString().padStart(2, '0');
                    document.getElementById('ui-density').innerText = density.toFixed(2);
                    
                    const prio = document.getElementById('ui-priority');
                    const indexCard = document.getElementById('index-card');
                    const statusText = document.getElementById('ui-status-text');

                    if (score > 60) { 
                        prio.innerText = "CRITICAL"; prio.style.color = "#ff4444"; 
                        indexCard.classList.add('critical-alert');
                        statusText.innerText = "Violation Detected";
                    } else if (score > 25) { 
                        prio.innerText = "MODERATE"; prio.style.color = "#ffbb00"; 
                        indexCard.classList.remove('critical-alert');
                        statusText.innerText = "Under Review";
                    } else { 
                        prio.innerText = "CLEAN"; prio.style.color = "#00f58d"; 
                        indexCard.classList.remove('critical-alert');
                        statusText.innerText = "All Clear";
                    }
                },

                forceReport() {
                    const score = parseInt(document.getElementById('ui-index').innerText);
                    const count = document.getElementById('ui-count').innerText;
                    if (count == "00") { alert("No Waste Detected."); return; }

                    const id = Math.random().toString(36).substr(2, 6).toUpperCase();
                    const feed = document.getElementById('incident-feed');
                    if(feed.innerText.includes("No Active Violations")) feed.innerHTML = '';

                    const item = document.createElement('div');
                    item.className = "glass-morphism p-3 rounded-2xl flex justify-between items-center text-left border-l-2 border-red-500 mb-2";
                    item.innerHTML = `
                        <div>
                            <p class="text-[9px] text-slate-500 font-bold">CASE ID: ${id}</p>
                            <p class="text-[11px] font-bold text-white uppercase">${score}% POLLUTION</p>
                        </div>
                        <div class="flex gap-2">
                            <span class="text-[8px] bg-red-600/20 text-red-500 px-2 py-1 rounded-md uppercase">Detected</span>
                            <button onclick="this.parentElement.innerHTML='<span class=\'text-[8px] bg-green-600/20 text-green-500 px-2 py-1 rounded-md uppercase\'>Resolved</span>'" class="text-[8px] bg-white/5 px-2 py-1 rounded-md uppercase">Resolve</button>
                        </div>
                    `;
                    feed.prepend(item);
                    this.log(`Case ${id} pushed to Solapur Node.`, "blue");
                },

                exportData() {
                    const report = {
                        node: "GREEN_STREAM_SOLAPUR_01",
                        timestamp: new Date().toISOString(),
                        current_index: document.getElementById('ui-index').innerText,
                        waste_count: document.getElementById('ui-count').innerText,
                        location: userLocation
                    };
                    const blob = new Blob([JSON.stringify(report)], {type: 'application/json'});
                    const url = URL.createObjectURL(blob);
                    const a = document.createElement('a');
                    a.href = url; a.download = 'solapur_pollution_report.json'; a.click();
                    this.log("Analytical Data Saved.", "green");
                },

                log(msg, color) {
                    const entry = document.createElement('p');
                    const colors = { green: '#00f58d', blue: '#4488ff', red: '#ff4444' };
                    entry.innerHTML = `<span style="color:${colors[color]}">></span> ${msg}`;
                    const logEl = document.getElementById('activity-log');
                    logEl.prepend(entry);
                }
            };
        })();

        window.onload = () => GreenStream.init();
    </script>
</body>
</html>
