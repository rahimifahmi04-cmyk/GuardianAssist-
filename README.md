<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover">
    
    <title>Guardian Assistant Pro</title>
    
    <script src="https://cdn.jsdelivr.net/npm/@ericblade/quagga2/dist/quagga.min.js"></script>

    <style>
        :root { --g-green: #00754a; --g-orange: #ffc107; --danger: #d32f2f; --safe: #388e3c; --bg: #f8f9fa; }
        body { font-family: 'Segoe UI', Roboto, sans-serif; margin: 0; background: var(--bg); padding-bottom: 80px; -webkit-tap-highlight-color: transparent; }
        
        /* App UI Sections */
        .header { background: var(--g-green); color: white; padding: 45px 15px 15px; text-align: center; position: sticky; top: 0; z-index: 100; font-weight: bold; }
        .search-area { padding: 15px; background: white; position: sticky; top: 85px; z-index: 99; border-bottom: 1px solid #eee; }
        #searchInput { width: 100%; padding: 15px 20px; border: 2px solid #eee; border-radius: 30px; box-sizing: border-box; font-size: 16px; outline: none; }

        /* Category Scroll */
        .category-bar { display: flex; overflow-x: auto; background: white; padding: 10px; gap: 10px; scrollbar-width: none; }
        .category-bar::-webkit-scrollbar { display: none; }
        .cat-btn { padding: 10px 20px; background: #eee; border-radius: 25px; white-space: nowrap; border: none; font-size: 13px; font-weight: bold; color: #555; }
        .cat-btn.active { background: var(--g-green); color: white; }

        /* Product Cards */
        .list { padding: 15px; }
        .card { background: white; border-radius: 15px; padding: 15px; margin-bottom: 12px; display: flex; align-items: center; box-shadow: 0 2px 8px rgba(0,0,0,0.05); transition: 0.2s; }
        .info { flex-grow: 1; }
        .brand { font-size: 11px; color: var(--g-green); font-weight: 800; text-transform: uppercase; }
        .name { font-size: 15px; font-weight: 600; margin: 4px 0; }
        .price { color: #e60000; font-weight: 800; font-size: 18px; }
        .aisle { font-size: 11px; color: #888; margin-top: 5px; display: block; }

        /* Enhanced Scanner Modal */
        #scanner-ui { 
            display: none; 
            position: fixed; 
            top: 0; 
            left: 0; 
            width: 100%; 
            height: 100%; 
            background: #000; 
            z-index: 1000; 
        }
        .scanner-container { 
            width: 100%; 
            height: 100%; 
            position: relative;
            overflow: hidden;
        }
        #interactive { 
            width: 100%; 
            height: 100%; 
            position: relative;
        }
        .scanner-viewport {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }
        .overlay-box { 
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 280px;
            height: 180px;
            border: 3px solid #00ff00;
            border-radius: 15px;
            box-shadow: 0 0 0 2000px rgba(0,0,0,0.85);
            z-index: 1001;
            animation: pulse 2s infinite;
            pointer-events: none;
        }
        @keyframes pulse {
            0% { border-color: #00ff00; box-shadow: 0 0 0 2000px rgba(0,0,0,0.85), 0 0 20px rgba(0,255,0,0.5); }
            50% { border-color: #00cc00; box-shadow: 0 0 0 2000px rgba(0,0,0,0.85), 0 0 40px rgba(0,255,0,0.8); }
            100% { border-color: #00ff00; box-shadow: 0 0 0 2000px rgba(0,0,0,0.85), 0 0 20px rgba(0,255,0,0.5); }
        }
        .scanner-guide { 
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: white;
            text-align: center;
            z-index: 1002;
            width: 300px;
            padding-top: 220px;
            font-weight: bold;
            text-shadow: 0 2px 4px rgba(0,0,0,0.9);
            font-size: 15px;
            pointer-events: none;
        }
        .focus-help {
            position: absolute;
            bottom: 100px;
            left: 0;
            width: 100%;
            text-align: center;
            color: white;
            font-size: 14px;
            padding: 10px;
            background: rgba(0,0,0,0.6);
            z-index: 1002;
        }

        /* Manual Input Overlay */
        #manual-input {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.9);
            z-index: 3000;
            justify-content: center;
            align-items: center;
        }
        .manual-box {
            background: white;
            border-radius: 20px;
            padding: 30px;
            width: 90%;
            max-width: 400px;
        }
        .manual-box input {
            width: 100%;
            padding: 15px;
            font-size: 18px;
            border: 2px solid #ddd;
            border-radius: 10px;
            margin: 15px 0;
            text-align: center;
            letter-spacing: 3px;
        }

        /* Loading State */
        .loading { 
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.95);
            z-index: 3000;
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
        }
        .spinner { 
            width: 50px;
            height: 50px;
            border: 5px solid #f3f3f3;
            border-top: 5px solid var(--g-green);
            border-radius: 50%;
            animation: spin 1s linear infinite;
            margin-bottom: 20px;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        /* Result Card Overlay */
        #result-card { 
            display: none; 
            position: fixed; 
            bottom: 0; 
            left: 0; 
            width: 100%; 
            height: 85vh; 
            background: white; 
            border-radius: 25px 25px 0 0; 
            padding: 25px; 
            box-sizing: border-box; 
            box-shadow: 0 -5px 25px rgba(0,0,0,0.2); 
            z-index: 2000; 
            overflow-y: auto;
            animation: slideUp 0.3s ease-out;
        }
        @keyframes slideUp {
            from { transform: translateY(100%); }
            to { transform: translateY(0); }
        }
        .status-badge { display: inline-block; padding: 6px 14px; border-radius: 15px; font-weight: bold; font-size: 11px; margin-bottom: 10px; }
        .status-danger { background: #ffebee; color: var(--danger); }
        .status-safe { background: #e8f5e9; color: var(--safe); }
        .highlight { color: var(--danger); text-decoration: underline; font-weight: bold; }
        .ingredient-list { 
            background: #f9f9f9; 
            padding: 15px; 
            border-radius: 10px; 
            margin-top: 15px;
            max-height: 200px;
            overflow-y: auto;
        }

        /* Scanner Control Bar */
        .scanner-controls {
            position: absolute;
            bottom: 20px;
            left: 0;
            width: 100%;
            display: flex;
            justify-content: center;
            gap: 15px;
            z-index: 1003;
        }
        .control-btn {
            padding: 12px 25px;
            border-radius: 25px;
            border: none;
            background: rgba(255,255,255,0.9);
            font-weight: bold;
            font-size: 14px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.3);
            display: flex;
            align-items: center;
            gap: 8px;
        }

        /* Navigation Bar */
        .nav { 
            position: fixed; 
            bottom: 0; 
            width: 100%; 
            background: white; 
            display: flex; 
            border-top: 1px solid #eee; 
            height: 75px; 
            align-items: center; 
            z-index: 500; 
            padding: 0 5px;
        }
        .nav-btn { 
            flex: 1; 
            text-align: center; 
            border: none; 
            background: none; 
            color: #888; 
            font-size: 11px; 
            font-weight: bold; 
            cursor: pointer; 
            padding: 10px 5px;
        }
        .scan-circle { 
            width: 65px; 
            height: 65px; 
            background: var(--g-green); 
            border-radius: 50%; 
            display: flex; 
            align-items: center; 
            justify-content: center; 
            color: white; 
            margin-top: -40px; 
            border: 8px solid var(--bg); 
            font-size: 24px;
            box-shadow: 0 4px 12px rgba(0,117,74,0.3);
        }
        
        /* Exit Button */
        .exit-btn {
            position: absolute;
            top: 20px;
            right: 20px;
            z-index: 1005;
            padding: 12px 24px;
            border-radius: 25px;
            border: none;
            background: rgba(255,255,255,0.9);
            font-weight: bold;
            font-size: 14px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.2);
        }
        
        /* Scanner Status */
        .scanner-status {
            position: absolute;
            top: 20px;
            left: 20px;
            z-index: 1005;
            padding: 10px 15px;
            border-radius: 20px;
            background: rgba(0,0,0,0.7);
            color: white;
            font-size: 12px;
            display: none;
        }
        
        /* Focus Adjustment */
        .focus-slider-container {
            position: absolute;
            top: 80px;
            right: 20px;
            z-index: 1005;
            background: rgba(0,0,0,0.7);
            padding: 15px;
            border-radius: 15px;
            display: none;
        }
        .focus-slider {
            width: 200px;
            height: 5px;
            -webkit-appearance: none;
            background: #ddd;
            border-radius: 5px;
            outline: none;
        }
        .focus-slider::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background: var(--g-green);
            cursor: pointer;
        }
    </style>
</head>
<body>

<div class="header">GUARDIAN HEALTH-GUARD PRO</div>

<div id="main-ui">
    <div class="search-area">
        <input type="text" id="searchInput" placeholder="Search brands (e.g. Wardah, Panadol)..." onkeyup="search()">
    </div>
    <div class="category-bar" id="catBar"></div>
    <div class="list" id="productList"></div>
</div>

<div id="scanner-ui">
    <div class="scanner-container">
        <div id="interactive" class="viewport"></div>
        <div class="overlay-box"></div>
        <div class="scanner-guide">Hold steady 6-12 inches from barcode</div>
        <div class="focus-help" id="focus-help">Move closer or further for better focus</div>
        
        <div class="scanner-status" id="scanner-status">Ready</div>
        
        <div class="focus-slider-container" id="focus-slider">
            <div style="color: white; margin-bottom: 10px;">Manual Focus</div>
            <input type="range" min="0.1" max="1" step="0.1" value="0.5" class="focus-slider" id="focus-range">
        </div>
        
        <button class="exit-btn" onclick="stopScanner()">✕ EXIT</button>
        
        <div class="scanner-controls">
            <button class="control-btn" onclick="toggleManualInput()">📝 MANUAL</button>
            <button class="control-btn" onclick="triggerFocus()">🎯 FOCUS</button>
            <button class="control-btn" onclick="toggleTorch()">⚡ TORCH</button>
        </div>
    </div>
</div>

<!-- Manual Barcode Input -->
<div id="manual-input">
    <div class="manual-box">
        <h3 style="margin: 0 0 15px 0;">Enter Barcode Manually</h3>
        <p style="color: #666; margin-bottom: 10px;">If scanner can't read, enter 12-13 digit barcode:</p>
        <input type="text" id="manual-barcode" placeholder="123456789012" maxlength="13" pattern="[0-9]{12,13}">
        <div style="display: flex; gap: 10px;">
            <button onclick="submitManualBarcode()" style="flex: 1; background: var(--g-green); color: white; border: none; padding: 15px; border-radius: 10px; font-weight: bold;">ANALYZE</button>
            <button onclick="closeManualInput()" style="flex: 1; background: #ddd; color: #333; border: none; padding: 15px; border-radius: 10px; font-weight: bold;">CANCEL</button>
        </div>
    </div>
</div>

<!-- Result Card -->
<div id="result-card">
    <div id="safety-badge" class="status-badge">Analyzing...</div>
    <h2 id="res-name" style="margin: 0; font-size: 20px;">Product Name</h2>
    <div style="font-size: 28px; color: #e60000; font-weight: bold; margin: 10px 0;" id="res-price">RM --.--</div>
    
    <div class="ingredient-list">
        <span style="font-weight: bold; font-size: 13px;">🔎 INGREDIENT ANALYSIS</span>
        <div id="warning-container" style="color:var(--danger); font-weight:bold; font-size:14px; margin: 10px 0;"></div>
        <div id="res-contents" style="font-size: 13px; color: #666; line-height: 1.4; border-top: 1px solid #ddd; padding-top: 10px;"></div>
    </div>
    
    <div style="margin-top: 20px; display: flex; gap: 10px;">
        <button onclick="document.getElementById('result-card').style.display='none'; startScanner();" style="flex: 1; background:var(--g-green); color:white; border:none; padding:15px; border-radius:10px; font-weight:bold;">SCAN NEXT</button>
        <button onclick="document.getElementById('result-card').style.display='none';" style="flex: 1; background:#eee; color:#333; border:none; padding:15px; border-radius:10px; font-weight:bold;">CLOSE</button>
    </div>
</div>

<!-- Loading Overlay -->
<div class="loading" id="loading">
    <div class="spinner"></div>
    <div id="loading-text">Initializing scanner...</div>
</div>

<!-- Navigation -->
<div class="nav">
    <button class="nav-btn" onclick="stopScanner()">🏠<br>HOME</button>
    <div class="nav-btn" onclick="startScanner()"><div class="scan-circle">🔍</div>SCAN</div>
    <button class="nav-btn" onclick="alert('Browse database active')">📂<br>ITEMS</button>
</div>

<script>
    // Database and red flags
    const redFlags = [
        "methylparaben", "propylparaben", "butylparaben", "ethylparaben",
        "alcohol denat", "sd alcohol", "isopropyl alcohol",
        "sodium lauryl sulfate", "sls", "sodium laureth sulfate", "sles",
        "fragrance", "parfum", "perfume",
        "phthalate", "dibutyl phthalate", "dep", "dehp",
        "dimethicone", "cyclomethicone", "simethicone"
    ];
    
    const rawBrands = {
        "Skincare": ["Cetaphil", "Hada Labo", "Bio-Essence", "Wardah", "Loreal", "Eucerin", "Simple", "Garnier", "Neutrogena", "Sunsilk", "Olay", "Cosrx", "Nivea", "Clinelle", "Aiken"],
        "Health": ["Panadol", "Gaviscon", "Blackmores", "Hurix's", "Flavettes", "Brands", "Eno", "Tiger Balm", "Vicks", "Strepsils", "Eye Mo", "Betadine", "Dettol Antiseptic", "Woods", "Cap Ibu & Anak"],
        "Personal Care": ["Dettol Shower", "Colgate", "Pantene", "Dove", "Lifebuoy", "Rexona", "Sunplay", "Sensodyne", "Oral-B", "Shokubutsu", "May", "Ginvera", "Kotex", "Libresse", "Carefree"],
        "Cosmetics": ["Maybelline", "Silkygirl", "Revlon", "Wardah Colorfit", "In2It", "Kate", "Peripera", "Essence", "Catrice", "Elianto"],
        "Baby": ["Johnson's", "Huggies", "MamyPoko", "Pureen", "Pigeon", "Cetaphil Baby", "Sebamed", "Biolane"]
    };

    let fullDb = [];
    Object.keys(rawBrands).forEach(cat => {
        rawBrands[cat].forEach(brand => {
            for(let i=1; i<=3; i++) {
                fullDb.push({
                    b: brand,
                    n: `${brand} ${cat} Series ${i}`,
                    p: (Math.random() * 85 + 5).toFixed(2),
                    c: cat,
                    a: `Aisle ${Math.floor(Math.random() * 10) + 1}`,
                    code: Math.floor(1000000000000 + Math.random() * 9000000000000).toString()
                });
            }
        });
    });

    // Scanner Variables
    let scannerActive = false;
    let torchOn = false;
    let currentStream = null;
    let scanAttempts = 0;
    const MAX_SCAN_ATTEMPTS = 30;
    let scanTimeout = null;

    // --- Search & UI Logic ---
    function render(data) {
        document.getElementById('productList').innerHTML = data.map(i => `
            <div class="card" onclick="showProductDetails('${i.code}')">
                <div class="info">
                    <div class="brand">${i.b}</div>
                    <div class="name">${i.n}</div>
                    <p class="price">RM ${i.p}</p>
                    <span class="aisle">📍 ${i.c} | ${i.a}</span>
                    <div style="font-size: 10px; color: #999; margin-top: 5px;">Barcode: ${i.code}</div>
                </div>
            </div>
        `).join('');
    }

    function showProductDetails(barcode) {
        document.getElementById('main-ui').style.display = 'none';
        analyzeProduct(barcode);
    }

    function search() {
        const q = document.getElementById('searchInput').value.toLowerCase();
        render(fullDb.filter(i => i.n.toLowerCase().includes(q) || i.b.toLowerCase().includes(q)));
    }

    function filter(cat, btn) {
        document.querySelectorAll('.cat-btn').forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        render(cat === "All" ? fullDb : fullDb.filter(i => i.c === cat));
    }

    // --- Enhanced Scanner Logic with Better Focus ---
    async function startScanner() {
        document.getElementById('main-ui').style.display = 'none';
        document.getElementById('scanner-ui').style.display = 'block';
        document.getElementById('result-card').style.display = 'none';
        document.getElementById('loading').style.display = 'flex';
        document.getElementById('loading-text').innerText = 'Initializing camera...';
        
        try {
            // First get camera access
            const stream = await navigator.mediaDevices.getUserMedia({
                video: {
                    facingMode: "environment",
                    width: { ideal: 1920, max: 2560 },
                    height: { ideal: 1080, max: 1440 },
                    frameRate: { ideal: 30, max: 60 }
                },
                audio: false
            });
            
            currentStream = stream;
            
            // Configure Quagga with optimized settings
            const config = {
                inputStream: {
                    name: "Live",
                    type: "LiveStream",
                    target: document.querySelector('#interactive'),
                    constraints: {
                        facingMode: "environment",
                        width: { min: 1280, ideal: 1920 },
                        height: { min: 720, ideal: 1080 },
                        aspectRatio: { min: 1.3, max: 1.7 }
                    },
                    area: {
                        top: "25%",
                        right: "25%",
                        left: "25%",
                        bottom: "25%"
                    },
                    singleChannel: false
                },
                decoder: {
                    readers: [
                        "ean_reader",
                        "ean_8_reader",
                        "upc_reader",
                        "upc_e_reader",
                        "code_128_reader",
                        "code_39_reader",
                        "codabar_reader"
                    ],
                    multiple: false
                },
                locator: {
                    patchSize: "medium",
                    halfSample: true
                },
                locate: true,
                numOfWorkers: navigator.hardwareConcurrency || 4,
                frequency: 15,
                debug: {
                    drawBoundingBox: false,
                    showFrequency: false,
                    drawScanline: true,
                    showPattern: false
                }
            };
            
            Quagga.init(config, function(err) {
                document.getElementById('loading').style.display = 'none';
                
                if (err) {
                    console.error("Scanner initialization failed:", err);
                    document.getElementById('scanner-status').innerText = "Error: " + err.message;
                    document.getElementById('scanner-status').style.display = 'block';
                    document.getElementById('scanner-status').style.background = "rgba(211, 47, 47, 0.8)";
                    
                    // Show manual input as fallback
                    setTimeout(() => {
                        toggleManualInput();
                    }, 1000);
                    return;
                }
                
                console.log("Scanner initialized successfully");
                Quagga.start();
                scannerActive = true;
                scanAttempts = 0;
                
                // Update status
                document.getElementById('scanner-status').innerText = "Ready - Focus on barcode";
                document.getElementById('scanner-status').style.display = 'block';
                document.getElementById('scanner-status').style.background = "rgba(0, 117, 74, 0.8)";
                
                // Start scan timeout
                scanTimeout = setTimeout(() => {
                    if (scannerActive) {
                        document.getElementById('focus-help').innerText = "Try moving closer or adjusting angle";
                        document.getElementById('scanner-status').innerText = "Having trouble? Try manual input";
                        document.getElementById('scanner-status').style.background = "rgba(255, 193, 7, 0.8)";
                    }
                }, 10000);
            });
            
            Quagga.onDetected(async function(result) {
                if (!result || !result.codeResult || !result.codeResult.code) return;
                
                const code = result.codeResult.code;
                console.log("Barcode detected:", code);
                
                // Stop scanner immediately
                Quagga.offDetected();
                Quagga.stop();
                scannerActive = false;
                clearTimeout(scanTimeout);
                
                // Show loading
                document.getElementById('loading').style.display = 'flex';
                document.getElementById('loading-text').innerText = 'Analyzing product...';
                
                // Add slight delay for UX
                await new Promise(resolve => setTimeout(resolve, 500));
                
                await analyzeProduct(code);
                
                document.getElementById('loading').style.display = 'none';
            });
            
            Quagga.onProcessed(function(result) {
                if (!result) return;
                
                scanAttempts++;
                
                // Update status based on attempts
                if (scanAttempts > MAX_SCAN_ATTEMPTS / 2) {
                    document.getElementById('scanner-status').innerText = "Adjust distance for better focus";
                    document.getElementById('scanner-status').style.background = "rgba(255, 152, 0, 0.8)";
                }
                
                // Draw debug info
                const drawingCtx = Quagga.canvas.ctx.overlay;
                const drawingCanvas = Quagga.canvas.dom.overlay;
                
                drawingCtx.clearRect(0, 0, parseInt(drawingCanvas.getAttribute("width")), parseInt(drawingCanvas.getAttribute("height")));
                
                if (result.boxes) {
                    result.boxes.filter(function(box) {
                        return box !== result.box;
                    }).forEach(function(box) {
                        Quagga.ImageDebug.drawPath(box, {x: 0, y: 1}, drawingCtx, {color: "rgba(0, 255, 0, 0.3)", lineWidth: 1});
                    });
                }
                
                if (result.box) {
                    Quagga.ImageDebug.drawPath(result.box, {x: 0, y: 1}, drawingCtx, {color: "#00F", lineWidth: 2});
                }
                
                if (result.codeResult && result.codeResult.code) {
                    Quagga.ImageDebug.drawPath(result.line, {x: 'x', y: 'y'}, drawingCtx, {color: 'red', lineWidth: 3});
                }
            });
            
        } catch (error) {
            console.error("Camera access error:", error);
            document.getElementById('loading').style.display = 'none';
            document.getElementById('scanner-status').innerText = "Camera access denied";
            document.getElementById('scanner-status').style.display = 'block';
            document.getElementById('scanner-status').style.background = "rgba(211, 47, 47, 0.8)";
            
            // Show manual input
            setTimeout(() => {
                toggleManualInput();
            }, 1000);
        }
    }

    function triggerFocus() {
        // Try to trigger autofocus
        if (currentStream) {
            const videoTrack = currentStream.getVideoTracks()[0];
            if (videoTrack && videoTrack.getCapabilities && videoTrack.getSettings) {
                try {
                    const capabilities = videoTrack.getCapabilities();
                    if (capabilities.focusMode && capabilities.focusMode.includes('continuous')) {
                        videoTrack.applyConstraints({
                            advanced: [{ focusMode: 'continuous' }]
                        });
                        document.getElementById('scanner-status').innerText = "Auto-focus triggered";
                        document.getElementById('scanner-status').style.display = 'block';
                        setTimeout(() => {
                            document.getElementById('scanner-status').style.display = 'none';
                        }, 2000);
                    }
                } catch (e) {
                    console.log("Focus not supported");
                    document.getElementById('focus-slider').style.display = 'block';
                }
            }
        }
    }

    function toggleTorch() {
        if (!currentStream) return;
        
        const videoTrack = currentStream.getVideoTracks()[0];
        if (videoTrack && videoTrack.getCapabilities && videoTrack.getSettings) {
            try {
                const capabilities = videoTrack.getCapabilities();
                if (capabilities.torch) {
                    torchOn = !torchOn;
                    videoTrack.applyConstraints({
                        advanced: [{ torch: torchOn }]
                    });
                    document.getElementById('scanner-status').innerText = torchOn ? "Torch ON" : "Torch OFF";
                    document.getElementById('scanner-status').style.display = 'block';
                    setTimeout(() => {
                        document.getElementById('scanner-status').style.display = 'none';
                    }, 2000);
                }
            } catch (e) {
                alert("Torch not supported on this device");
            }
        }
    }

    // Manual Input Functions
    function toggleManualInput() {
        const manualInput = document.getElementById('manual-input');
        if (scannerActive) {
            Quagga.stop();
            scannerActive = false;
        }
        manualInput.style.display = 'flex';
    }

    function closeManualInput() {
        document.getElementById('manual-input').style.display = 'none';
        document.getElementById('manual-barcode').value = '';
        startScanner();
    }

    function submitManualBarcode() {
        const barcode = document.getElementById('manual-barcode').value.trim();
        if (barcode.length >= 12 && /^\d+$/.test(barcode)) {
            document.getElementById('manual-input').style.display = 'none';
            document.getElementById('loading').style.display = 'flex';
            document.getElementById('loading-text').innerText = 'Analyzing product...';
            
            setTimeout(() => {
                analyzeProduct(barcode);
                document.getElementById('manual-barcode').value = '';
            }, 500);
        } else {
            alert("Please enter a valid 12-13 digit barcode");
        }
    }

    // Product Analysis
    async function analyzeProduct(barcode) {
        document.getElementById('scanner-ui').style.display = 'none';
        document.getElementById('result-card').style.display = 'block';
        document.getElementById('loading').style.display = 'none';
        
        try {
            // Check local database first
            let localProduct = fullDb.find(p => p.code === barcode);
            if (!localProduct) {
                // Try to find by brand prefix
                const prefix = barcode.substring(0, 3);
                localProduct = fullDb.find(p => p.b.toLowerCase().includes(prefix));
            }
            
            // Try OpenFoodFacts API
            let apiData = null;
            try {
                const response = await fetch(`https://world.openfoodfacts.org/api/v0/product/${barcode}.json`, {
                    signal: AbortSignal.timeout(5000)
                });
                if (response.ok) {
                    apiData = await response.json();
                }
            } catch (apiError) {
                console.log("API call failed, using fallback");
            }
            
            let productName = "Scanned Product";
            let ingredients = "No ingredient information available.";
            let price = "RM " + (Math.random() * 40 + 10).toFixed(2);
            let brand = "Unknown Brand";
            
            if (apiData && apiData.status === 1 && apiData.product) {
                const p = apiData.product;
                productName = p.product_name || p.generic_name || "Unknown Product";
                ingredients = p.ingredients_text || "Ingredients not specified.";
                brand = p.brands || "";
            }
            
            if (localProduct) {
                productName = localProduct.n;
                brand = localProduct.b;
                ingredients = `Standard ${localProduct.b} formula. Check packaging for full ingredient list.`;
                price = "RM " + localProduct.p;
            }
            
            document.getElementById('res-name').innerText = productName;
            document.getElementById('res-price').innerText = price;
            
            // Enhanced ingredient analysis
            let foundFlags = [];
            let highlightedText = ingredients;
            
            redFlags.forEach(flag => {
                const regex = new RegExp(`\\b${flag.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')}\\b`, "gi");
                if (regex.test(ingredients)) {
                    foundFlags.push(flag.toUpperCase());
                    highlightedText = highlightedText.replace(regex, `<span class="highlight">${flag}</span>`);
                }
            });
            
            const badge = document.getElementById('safety-badge');
            const warning = document.getElementById('warning-container');
            
            if (foundFlags.length > 2) {
                badge.className = "status-badge status-danger";
                badge.innerText = "⚠️ HIGH RISK - MULTIPLE IRRITANTS";
                warning.innerHTML = `<span style="color: ${danger}">Detected ${foundFlags.length} potential irritants</span><br>${foundFlags.slice(0, 3).join(", ")}${foundFlags.length > 3 ? '...' : ''}`;
            } else if (foundFlags.length > 0) {
                badge.className = "status-badge status-danger";
                badge.innerText = "⚠️ CONTAINS POTENTIAL IRRITANTS";
                warning.innerHTML = `<span style="color: ${danger}">Detected:</span> ${foundFlags.join(", ")}`;
            } else {
                badge.className = "status-badge status-safe";
                badge.innerText = "✅ LOW RISK FORMULATION";
                warning.innerHTML = `<span style="color: ${safe}">No common irritants detected.</span><br><small>Always patch test new products.</small>`;
            }
            
            document.getElementById('res-contents').innerHTML = highlightedText || "No ingredient data available.";
            
        } catch (error) {
            console.error("Analysis error:", error);
            document.getElementById('res-name').innerText = "Scan Result";
            document.getElementById('res-price').innerText = "RM --.--";
            document.getElementById('safety-badge').className = "status-badge status-danger";
            document.getElementById('safety-badge').innerText = "⚠️ ANALYSIS ERROR";
            document.getElementById('warning-container').innerText = "Unable to retrieve product data. Please try again.";
            document.getElementById('res-contents').innerText = "Network or database
