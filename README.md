<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GREEN-STREAM PRO | AI Command Center</title>
    
    <!-- External Frameworks -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"></script>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=Inter:wght@300;400;600&display=swap');
        
        :root {
            --neon-green: #00ff88;
            --alert-red: #ff3e3e;
            --bg-dark: #050a0f;
        }

        body { font-family: 'Inter', sans-serif; background-color: var(--bg-dark); color: white; overflow: hidden; }
        .cyber-font { font-family: 'Orbitron', sans-serif; }
        
        #map { height: 400px; width: 100%; border-radius: 12px; border: 1px solid #1e293b; }
        
        .video-container { position: relative; width: 100%; background: #000; border-radius: 12px; overflow: hidden; }
        #webcam { width: 100%; height: auto; transform: scaleX(-1); }
        canvas { position: absolute; top: 0; left: 0; width: 100%; height: 100%; transform: scaleX(-1); }

        .status-pulse {
            width: 10px; height: 10px; background: var(--neon-green);
            border-radius: 50%; box-shadow: 0 0 10px var(--neon-green);
            animation: pulse 1.5s infinite;
        }

        @keyframes pulse { 0% { opacity: 0.4; } 50% { opacity: 1; } 100% { opacity: 0.4; } }

        .glass-panel {
            background: rgba(15, 23, 42, 0.8);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        #log-container::-webkit-scrollbar { width: 4px; }
        #log-container::-webkit-scrollbar-thumb { background: #334155; }
    </style>
</head>
<body class="p-4">

    <!-- Top Navigation Bar -->
    <header class="flex justify-between items-center mb-6 glass-panel p-4 rounded-2xl shadow-2xl">
        <div class="flex items-center gap-4">
            <div class="status-pulse"></div>
            <h1 class="cyber-font text-xl font-bold tracking-tighter">PROJECT <span class="text-green-400">GREEN-STREAM</span> v2.0</h1>
        </div>
        <div class="flex gap-8 text-xs cyber-font">
            <div>NETWORK: <span class="text-green-400">STABLE</span></div>
            <div>LATENCY: <span id="latency">24ms</span></div>
            <div>Uptime: <span id="uptime">00:00:00</span></div>
        </div>
    </header>

    <div class="grid grid-cols-12 gap-6 h-[calc(100vh-120px)]">
        
        <!-- Left Wing: Live AI Vision -->
        <div class="col-span-12 lg:col-span-4 flex flex-col gap-4">
            <div class="glass-panel p-4 rounded-2xl relative flex-1">
                <div class="flex justify-between mb-2">
                    <span class="text-xs font-bold uppercase tracking-widest text-gray-400 italic">Live AI Vision Analytics</span>
                    <span class="text-xs text-red-500 animate-pulse font-bold">REC ●</span>
                </div>
                
                <div class="video-container shadow-inner">
                    <video id="webcam" autoplay playsinline muted></video>
                    <canvas id="canvas"></canvas>
                    
                    <!-- Loading Overlay -->
                    <div id="ai-loader" class="absolute inset-0 flex flex-col items-center justify-center bg-black/80 z-50">
                        <i class="fas fa-microchip text-4xl text-green-400 animate-spin mb-4"></i>
                        <p class="cyber-font text-sm">Initializing TensorFlow AI...</p>
                    </div>
                </div>

                <div class="mt-4 grid grid-cols-2 gap-2">
                    <div class="bg-black/40 p-3 rounded-lg border border-white/5">
                        <p class="text-[10px] text-gray-400 uppercase">Detection Mode</p>
                        <p class="text-sm font-bold text-green-400">Waste / Plastic</p>
                    </div>
                    <div class="bg-black/40 p-3 rounded-lg border border-white/5">
                        <p class="text-[10px] text-gray-400 uppercase">Confidence</p>
                        <p class="text-sm font-bold text-blue-400" id="confidence-lv">0%</p>
                    </div>
                </div>
            </div>

            <!-- Telemetry Data -->
            <div class="glass-panel p-4 rounded-2xl h-1/3 overflow-hidden">
                <h3 class="text-xs font-bold mb-3 cyber-font text-gray-400">LIVE SENSOR TELEMETRY</h3>
                <div class="space-y-4">
                    <div>
                        <div class="flex justify-between text-[10px] mb-1"><span>TURBIDITY (NTU)</span><span id="turb-val">4.2</span></div>
                        <div class="w-full bg-gray-700 h-1.5 rounded-full overflow-hidden">
                            <div id="turb-bar" class="bg-blue-500 h-full transition-all duration-500" style="width: 40%"></div>
                        </div>
                    </div>
                    <div>
                        <div class="flex justify-between text-[10px] mb-1"><span>PH LEVEL</span><span id="ph-val">7.2</span></div>
                        <div class="w-full bg-gray-700 h-1.5 rounded-full overflow-hidden">
                            <div id="ph-bar" class="bg-green-500 h-full transition-all duration-500" style="width: 70%"></div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Center: Real-Time Map -->
        <div class="col-span-12 lg:col-span-5 flex flex-col gap-4">
            <div class="glass-panel p-2 rounded-2xl flex-1 relative overflow-hidden">
                <div id="map"></div>
                <!-- Map Overlay -->
                <div class="absolute bottom-6 left-6 z-[1000] glass-panel p-4 rounded-xl border-green-500/50 max-w-[200px]">
                    <p class="text-[10px] font-bold text-green-400 mb-1">GEO-FENCE ACTIVE</p>
                    <p class="text-[9px] text-gray-300">Monitoring 4.2km of urban drainage network.</p>
                </div>
            </div>
        </div>

        <!-- Right Wing: Incident Log -->
        <div class="col-span-12 lg:col-span-3 flex flex-col gap-4">
            <div class="glass-panel rounded-2xl flex flex-col h-full border-l-2 border-red-500/20">
                <div class="p-4 border-b border-white/5 flex justify-between items-center">
                    <span class="cyber-font text-sm font-bold tracking-widest">ALERTS LOG</span>
                    <i class="fas fa-shield-alt text-red-500 animate-pulse"></i>
                </div>
                <div id="log-container" class="flex-1 overflow-y-auto p-4 space-y-3 font-mono text-[11px]">
                    <!-- Logs will appear here -->
                    <div class="text-blue-400">[SYSTEM] Initialization complete.</div>
                    <div class="text-green-400">[GPS] Satellite link established.</div>
                </div>
                <div class="p-4 bg-black/20 rounded-b-2xl">
                    <button onclick="window.print()" class="w-full py-3 bg-green-600 hover:bg-green-500 rounded-xl font-bold transition flex items-center justify-center gap-2">
                        <i class="fas fa-file-export"></i> GENERATE GC REPORT
                    </button>
                </div>
            </div>
        </div>
    </div>

    <script>
        const video = document.getElementById('webcam');
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        const logContainer = document.getElementById('log-container');
        let model;

        // 1. Initialize the Real-Time Map
        const map = L.map('map').setView([51.505, -0.09], 13);
        L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png').addTo(map);

        // Add a live pulsing marker for the station
        const stationMarker = L.circleMarker([51.505, -0.09], {
            radius: 10, color: '#00ff88', fillOpacity: 0.8
        }).addTo(map).bindPopup('<b>Station #042</b><br>Active Monitoring');

        // 2. Load Real AI Model (TensorFlow COCO-SSD)
        async function initAI() {
            try {
                model = await cocoSsd.load();
                document.getElementById('ai-loader').style.display = 'none';
                startCamera();
            } catch (e) {
                addLog("AI Error: Failed to load model", "text-red-500");
            }
        }

        async function startCamera() {
            const stream = await navigator.mediaDevices.getUserMedia({ video: true });
            video.srcObject = stream;
            video.onloadedmetadata = () => {
                canvas.width = video.videoWidth;
                canvas.height = video.videoHeight;
                detectFrame();
            };
        }

        // 3. Detect Frame Function
        async function detectFrame() {
            const predictions = await model.detect(video);
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            predictions.forEach(prediction => {
                // If it detects common plastic/waste objects (bottles, cups, etc)
                const wasteKeywords = ['bottle', 'cup', 'handbag', 'plastic'];
                const isWaste = wasteKeywords.includes(prediction.class);

                ctx.strokeStyle = isWaste ? "#ff3e3e" : "#00ff88";
                ctx.lineWidth = 4;
                ctx.strokeRect(...prediction.bbox);

                ctx.fillStyle = isWaste ? "#ff3e3e" : "#00ff88";
                ctx.font = "bold 18px Inter";
                ctx.fillText(`${prediction.class.toUpperCase()} ${Math.round(prediction.score * 100)}%`, prediction.bbox[0], prediction.bbox[1] > 20 ? prediction.bbox[1] - 10 : 20);

                if (isWaste && prediction.score > 0.65) {
                    addLog(`CRITICAL: ${prediction.class} detected at Station #042`, "text-red-500 font-bold");
                    document.getElementById('confidence-lv').innerText = `${Math.round(prediction.score * 100)}%`;
                }
            });

            requestAnimationFrame(detectFrame);
        }

        // 4. Utility Functions
        function addLog(msg, colorClass = "text-gray-400") {
            const time = new Date().toLocaleTimeString();
            const div = document.createElement('div');
            div.className = colorClass;
            div.innerHTML = `[${time}] ${msg}`;
            logContainer.prepend(div);
        }

        // 5. Live Telemetry Simulation
        setInterval(() => {
            const turb = (Math.random() * 10).toFixed(1);
            const ph = (6.5 + Math.random()).toFixed(1);
            document.getElementById('turb-val').innerText = turb;
            document.getElementById('turb-bar').style.width = (turb * 10) + '%';
            document.getElementById('ph-val').innerText = ph;
            document.getElementById('latency').innerText = Math.floor(Math.random() * 50) + 'ms';
        }, 3000);

        // Uptime Timer
        let seconds = 0;
        setInterval(() => {
            seconds++;
            const h = Math.floor(seconds / 3600).toString().padStart(2, '0');
            const m = Math.floor((seconds % 3600) / 60).toString().padStart(2, '0');
            const s = (seconds % 60).toString().padStart(2, '0');
            document.getElementById('uptime').innerText = `${h}:${m}:${s}`;
        }, 1000);

        // Start everything
        initAI();
    </script>
</body>
</html>
