<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>歯車の伝達比学習アプリ</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            font-family: 'Helvetica Neue', Arial, 'Hiragino Kaku Gothic ProN', 'Hiragino Sans', Meiryo, sans-serif;
            touch-action: none; /* Prevent scroll/zoom on touch devices for better control */
        }
        .canvas-container {
            background-image: radial-gradient(#e5e7eb 1px, transparent 1px);
            background-size: 20px 20px;
        }
        input[type=range] {
            height: 24px;
            -webkit-appearance: none;
            margin: 10px 0;
            width: 100%;
            background: transparent;
        }
        input[type=range]:focus {
            outline: none;
        }
        input[type=range]::-webkit-slider-runnable-track {
            width: 100%;
            height: 8px;
            cursor: pointer;
            animate: 0.2s;
            background: #d1d5db;
            border-radius: 5px;
        }
        input[type=range]::-webkit-slider-thumb {
            height: 20px;
            width: 20px;
            border-radius: 50%;
            background: #3b82f6;
            cursor: pointer;
            -webkit-appearance: none;
            margin-top: -6px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.3);
        }
        .driven-slider::-webkit-slider-thumb {
            background: #ef4444;
        }
        /* Custom Checkbox Style */
        .toggle-checkbox:checked {
            right: 0;
            border-color: #3b82f6;
        }
        .toggle-checkbox:checked + .toggle-label {
            background-color: #3b82f6;
        }
    </style>
</head>
<body class="bg-gray-50 text-gray-800 h-screen flex flex-col overflow-hidden">

    <!-- Header -->
    <header class="bg-white shadow-sm p-4 z-10 flex justify-between items-center shrink-0">
        <div class="flex items-center gap-2">
            <!-- Settings Icon -->
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="w-6 h-6 text-blue-600">
                <path d="M12.22 2h-.44a2 2 0 0 0-2 2v.18a2 2 0 0 1-1 1.73l-.43.25a2 2 0 0 1-2 0l-.15-.08a2 2 0 0 0-2.73.73l-.22.38a2 2 0 0 0 .73 2.73l.15.1a2 2 0 0 1 1 1.72v.51a2 2 0 0 1-1 1.74l-.15.09a2 2 0 0 0-.73 2.73l.22.38a2 2 0 0 0 2.73.73l.15-.08a2 2 0 0 1 2 0l.43.25a2 2 0 0 1 1 1.73V20a2 2 0 0 0 2 2h.44a2 2 0 0 0 2-2v-.18a2 2 0 0 1 1-1.73l.43-.25a2 2 0 0 1 2 0l.15.08a2 2 0 0 0 2.73-.73l.22-.38a2 2 0 0 0-.73-2.73l-.15-.1a2 2 0 0 1-1-1.72v-.51a2 2 0 0 1 1-1.74l.15-.09a2 2 0 0 0 .73-2.73l-.22-.38a2 2 0 0 0-2.73-.73l-.15.08a2 2 0 0 1-2 0l-.43-.25a2 2 0 0 1-1-1.73V4a2 2 0 0 0-2-2z"></path>
                <circle cx="12" cy="12" r="3"></circle>
            </svg>
            <h1 class="text-lg md:text-xl font-bold text-gray-800">歯車マスター：伝達比を学ぼう</h1>
        </div>
        <div class="flex gap-2">
            <button onclick="switchMode('simulation')" id="btn-sim" class="px-3 py-1.5 rounded-md text-sm font-medium bg-blue-100 text-blue-700 hover:bg-blue-200 transition">シミュレーション</button>
            <button onclick="switchMode('quiz')" id="btn-quiz" class="px-3 py-1.5 rounded-md text-sm font-medium text-gray-600 hover:bg-gray-100 transition">クイズ</button>
        </div>
    </header>

    <!-- Main Content Area -->
    <main class="flex-1 flex flex-col md:flex-row overflow-hidden">
        
        <!-- Simulation Canvas -->
        <div class="flex-1 canvas-container relative flex items-center justify-center bg-gray-100 overflow-hidden" id="canvas-wrapper">
            <canvas id="gearCanvas"></canvas>
            
            <!-- Floating Stats -->
            <div class="absolute top-4 left-4 bg-white/90 backdrop-blur p-3 rounded-lg shadow-md text-sm border border-gray-200 max-w-[200px]">
                <div class="flex justify-between items-center mb-1">
                    <span class="text-gray-500">ギア比 (i):</span>
                    <span id="ratio-display" class="font-bold text-lg text-indigo-600">1.00</span>
                </div>
                <div class="flex justify-between items-center mb-1">
                    <span class="text-blue-600 font-medium">入力回転:</span>
                    <span id="input-rpm-display" class="font-mono">60 rpm</span>
                </div>
                <div class="flex justify-between items-center">
                    <span class="text-red-600 font-medium">出力回転:</span>
                    <span id="output-rpm-display" class="font-mono">60 rpm</span>
                </div>
                <div class="mt-2 pt-2 border-t border-gray-200 text-xs text-gray-500">
                    <div>速度比 = 1 / ギア比</div>
                    <div>回転方向: <span id="direction-display" class="font-bold text-gray-700">逆転</span></div>
                </div>
                <!-- Manual Mode Indicator -->
                <div id="manual-indicator" class="hidden mt-2 pt-1 border-t border-gray-200 text-xs font-bold text-orange-500 flex items-center gap-1">
                    <svg xmlns="http://www.w3.org/2000/svg" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 20a8 8 0 1 0 0-16 8 8 0 0 0 0 16Z"/><path d="M12 14 9.1 8l5.8-3"/></svg>
                    手動モード: マウスホイールで回転
                </div>
            </div>

            <!-- Quiz Overlay -->
            <div id="quiz-overlay" class="hidden absolute top-4 right-4 bg-yellow-50/90 backdrop-blur p-4 rounded-lg shadow-lg border-l-4 border-yellow-400 w-64">
                <h3 class="font-bold text-yellow-800 mb-2 flex items-center gap-2">
                    <!-- Help Icon -->
                    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="w-4 h-4"><circle cx="12" cy="12" r="10"></circle><path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3"></path><line x1="12" y1="17" x2="12.01" y2="17"></line></svg>
                    問題
                </h3>
                <p class="text-sm text-yellow-900 mb-3" id="quiz-question">読み込み中...</p>
                <div id="quiz-feedback" class="text-sm font-bold h-6 mb-2"></div>
                <button onclick="checkAnswer()" class="w-full bg-yellow-500 hover:bg-yellow-600 text-white py-1.5 rounded shadow text-sm font-bold transition">回答する</button>
                <button onclick="nextQuestion()" class="w-full mt-2 bg-transparent text-yellow-700 hover:bg-yellow-100 py-1 rounded text-xs transition">別の問題へ</button>
            </div>
        </div>

        <!-- Controls Panel -->
        <div class="w-full md:w-80 bg-white border-t md:border-t-0 md:border-l border-gray-200 p-6 flex flex-col gap-6 overflow-y-auto shrink-0 z-20 shadow-[0_-5px_20px_rgba(0,0,0,0.05)]">
            
            <!-- Driver Gear Controls -->
            <div class="space-y-3">
                <div class="flex justify-between items-center">
                    <h2 class="text-sm font-bold text-blue-700 uppercase tracking-wide flex items-center gap-2">
                        <!-- Play Icon -->
                        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="w-4 h-4"><circle cx="12" cy="12" r="10"></circle><polygon points="10 8 16 12 10 16 10 8"></polygon></svg>
                        駆動歯車 (入力)
                    </h2>
                    <span class="bg-blue-100 text-blue-800 px-2 py-0.5 rounded text-xs font-mono">Z1</span>
                </div>
                <div class="bg-blue-50 p-4 rounded-lg border border-blue-100">
                    <div class="flex justify-between mb-1">
                        <label class="text-xs text-blue-600 font-medium">歯数 (Teeth)</label>
                        <span id="val-z1" class="text-sm font-bold text-blue-900">20</span>
                    </div>
                    <input type="range" id="slider-z1" min="8" max="40" value="20" step="1" class="w-full">
                    
                    <div id="speed-control-group">
                        <div class="flex justify-between mb-1 mt-3">
                            <label class="text-xs text-blue-600 font-medium">回転速度 (RPM)</label>
                            <span id="val-speed" class="text-sm font-bold text-blue-900">60</span>
                        </div>
                        <input type="range" id="slider-speed" min="0" max="120" value="60" step="10" class="w-full">
                    </div>

                    <!-- Manual Rotation Checkbox -->
                    <div class="mt-4 pt-3 border-t border-blue-200 flex items-center gap-3">
                        <input type="checkbox" id="check-manual" class="w-4 h-4 text-blue-600 rounded border-gray-300 focus:ring-blue-500 cursor-pointer">
                        <label for="check-manual" class="text-sm font-bold text-gray-700 cursor-pointer select-none">
                            手動回転 (マウスホイール)
                        </label>
                    </div>
                </div>
            </div>

            <!-- Driven Gear Controls -->
            <div class="space-y-3">
                <div class="flex justify-between items-center">
                    <h2 class="text-sm font-bold text-red-700 uppercase tracking-wide flex items-center gap-2">
                        <!-- Refresh Icon -->
                        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="w-4 h-4"><path d="M3 12a9 9 0 0 1 9-9 9.75 9.75 0 0 1 6.74 2.74L21 8"></path><path d="M21 3v5h-5"></path><path d="M21 12a9 9 0 0 1-9 9 9.75 9.75 0 0 1-6.74-2.74L3 16"></path><path d="M8 16H3v5"></path></svg>
                        被動歯車 (出力)
                    </h2>
                    <span class="bg-red-100 text-red-800 px-2 py-0.5 rounded text-xs font-mono">Z2</span>
                </div>
                <div class="bg-red-50 p-4 rounded-lg border border-red-100">
                    <div class="flex justify-between mb-1">
                        <label class="text-xs text-red-600 font-medium">歯数 (Teeth)</label>
                        <span id="val-z2" class="text-sm font-bold text-red-900">20</span>
                    </div>
                    <input type="range" id="slider-z2" min="8" max="40" value="20" step="1" class="driven-slider w-full">
                </div>
            </div>

            <!-- Formula Explanation -->
            <div class="bg-gray-50 rounded-lg p-4 border border-gray-200">
                <h3 class="text-sm font-bold text-gray-700 mb-2">計算式 (Theory)</h3>
                <div class="text-center font-mono text-lg bg-white p-2 rounded border border-gray-200 mb-2">
                    i = <span class="text-red-600">Z2</span> / <span class="text-blue-600">Z1</span>
                </div>
                <div class="text-center font-mono text-sm text-gray-600">
                    <span id="calc-z2" class="text-red-600 font-bold">20</span> 
                    ÷ 
                    <span id="calc-z1" class="text-blue-600 font-bold">20</span> 
                    = 
                    <span id="calc-result" class="text-indigo-600 font-bold">1.00</span>
                </div>
                <p class="text-xs text-gray-500 mt-2 leading-relaxed">
                    ギア比 (i) が <strong>1より大きい</strong> と「減速」し、力（トルク）が増します。<br>
                    <strong>1より小さい</strong> と「増速」し、回転が速くなります。
                </p>
            </div>

        </div>
    </main>

    <script>
        // --- Canvas Setup ---
        const canvas = document.getElementById('gearCanvas');
        const ctx = canvas.getContext('2d');
        const container = document.getElementById('canvas-wrapper');

        let width, height;
        const MODULE = 12; // Size scale of teeth

        // --- State ---
        let appState = {
            z1: 20, // Driver teeth
            z2: 20, // Driven teeth
            speed1: 60, // Driver RPM
            angle1: 0,
            angle2: 0,
            mode: 'simulation', // 'simulation' or 'quiz'
            isManual: false,    // Manual rotation mode
            quizTargetRatio: 1,
            quizTargetType: 'ratio' // 'ratio' or 'speed'
        };

        // --- DOM Elements ---
        const ui = {
            sliderZ1: document.getElementById('slider-z1'),
            sliderZ2: document.getElementById('slider-z2'),
            sliderSpeed: document.getElementById('slider-speed'),
            speedControlGroup: document.getElementById('speed-control-group'),
            checkManual: document.getElementById('check-manual'),
            manualIndicator: document.getElementById('manual-indicator'),
            
            valZ1: document.getElementById('val-z1'),
            valZ2: document.getElementById('val-z2'),
            valSpeed: document.getElementById('val-speed'),
            ratioDisplay: document.getElementById('ratio-display'),
            inputRpm: document.getElementById('input-rpm-display'),
            outputRpm: document.getElementById('output-rpm-display'),
            direction: document.getElementById('direction-display'),
            calcZ1: document.getElementById('calc-z1'),
            calcZ2: document.getElementById('calc-z2'),
            calcResult: document.getElementById('calc-result'),
            quizOverlay: document.getElementById('quiz-overlay'),
            quizQuestion: document.getElementById('quiz-question'),
            quizFeedback: document.getElementById('quiz-feedback'),
            btnSim: document.getElementById('btn-sim'),
            btnQuiz: document.getElementById('btn-quiz')
        };

        // --- Resize Handler ---
        function resize() {
            width = container.clientWidth;
            height = container.clientHeight;
            canvas.width = width;
            canvas.height = height;
            // High DPI support
            const dpr = window.devicePixelRatio || 1;
            canvas.width = width * dpr;
            canvas.height = height * dpr;
            ctx.scale(dpr, dpr);
            canvas.style.width = `${width}px`;
            canvas.style.height = `${height}px`;
        }
        window.addEventListener('resize', resize);
        resize();

        // --- Gear Drawing Logic ---
        function drawGear(cx, cy, teeth, color, angle, label, isDriver) {
            const pitchRadius = (teeth * MODULE) / 2;
            const rootRadius = pitchRadius - (1.25 * MODULE); 
            const outsideRadius = pitchRadius + (1.0 * MODULE);
            
            ctx.save();
            ctx.translate(cx, cy);
            ctx.rotate(angle);

            // Gear Body
            ctx.beginPath();
            ctx.fillStyle = color;
            ctx.strokeStyle = '#374151'; // Gray-700
            ctx.lineWidth = 2;

            // Draw Teeth
            const toothAngle = (Math.PI * 2) / teeth;
            const toothWidthFactor = 0.48; 
            const getXY = (r, a) => ({ x: r * Math.cos(a), y: r * Math.sin(a) });

            // Define Widths outside the loop
            const halfPitchWidth = (toothAngle * toothWidthFactor) / 2;
            const halfRootWidth = halfPitchWidth * 1.5; 
            const halfTipWidth = halfPitchWidth * 0.6;

            for (let i = 0; i < teeth; i++) {
                const centerAngle = i * toothAngle + toothAngle * 0.5;
                
                const aRootLeft = centerAngle - halfRootWidth;
                const aRootRight = centerAngle + halfRootWidth;
                const aTipLeft = centerAngle - halfTipWidth;
                const aTipRight = centerAngle + halfTipWidth;
                
                const pRootLeft = getXY(rootRadius, aRootLeft);
                const pRootRight = getXY(rootRadius, aRootRight);
                const pTipLeft = getXY(outsideRadius, aTipLeft);
                const pTipRight = getXY(outsideRadius, aTipRight);

                if (i === 0) {
                    ctx.moveTo(pRootLeft.x, pRootLeft.y);
                } else {
                    const prevCenterAngle = (i - 1) * toothAngle + toothAngle * 0.5;
                    const prevRootRightAngle = prevCenterAngle + halfRootWidth;
                    const valleyMidAngle = (prevRootRightAngle + aRootLeft) / 2;
                    const pValleyDeep = getXY(rootRadius * 0.95, valleyMidAngle);
                    ctx.quadraticCurveTo(pValleyDeep.x, pValleyDeep.y, pRootLeft.x, pRootLeft.y);
                }

                const cpFlankLeft = getXY(pitchRadius * 1.05, (aRootLeft + aTipLeft) / 2);
                ctx.quadraticCurveTo(cpFlankLeft.x, cpFlankLeft.y, pTipLeft.x, pTipLeft.y);

                const pTipMid = getXY(outsideRadius, centerAngle);
                ctx.quadraticCurveTo(pTipMid.x, pTipMid.y, pTipRight.x, pTipRight.y);

                const cpFlankRight = getXY(pitchRadius * 1.05, (aRootRight + aTipRight) / 2);
                ctx.quadraticCurveTo(cpFlankRight.x, cpFlankRight.y, pRootRight.x, pRootRight.y);
            }
            
            const firstCenterAngle = 0 * toothAngle + toothAngle * 0.5;
            const firstRootLeftAngle = firstCenterAngle - halfRootWidth;
            const pFirstRootLeft = getXY(rootRadius, firstRootLeftAngle);
            const pLastValleyDeep = getXY(rootRadius * 0.95, 0);
            
            ctx.quadraticCurveTo(pLastValleyDeep.x, pLastValleyDeep.y, pFirstRootLeft.x, pFirstRootLeft.y);

            ctx.closePath();
            ctx.fill();
            ctx.stroke();

            // Inner Circle (Hub)
            ctx.beginPath();
            ctx.arc(0, 0, pitchRadius * 0.3, 0, Math.PI * 2);
            ctx.fillStyle = '#f3f4f6'; // Light gray hub
            ctx.fill();
            ctx.stroke();

            // Pitch Circle
            ctx.beginPath();
            ctx.strokeStyle = 'rgba(0,0,0,0.15)';
            ctx.setLineDash([4, 4]);
            ctx.lineWidth = 1;
            ctx.arc(0, 0, pitchRadius, 0, Math.PI * 2);
            ctx.stroke();
            ctx.setLineDash([]);

            // Visual Marker
            ctx.beginPath();
            ctx.fillStyle = isDriver ? '#1d4ed8' : '#b91c1c';
            ctx.arc(pitchRadius * 0.6, 0, 4, 0, Math.PI * 2);
            ctx.fill();

            // Label
            ctx.rotate(-angle);
            ctx.fillStyle = '#1f2937';
            ctx.font = 'bold 12px sans-serif';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText(`${label} (${teeth}T)`, 0, isDriver ? pitchRadius + 30 : pitchRadius + 30);

            ctx.restore();
        }

        function drawShaft(cx, cy) {
            ctx.beginPath();
            ctx.fillStyle = '#9ca3af';
            ctx.arc(cx, cy, 5, 0, Math.PI * 2);
            ctx.fill();
        }

        // --- Animation Loop ---
        let lastTime = 0;

        function animate(timestamp) {
            if (!lastTime) lastTime = timestamp;
            const deltaTime = (timestamp - lastTime) / 1000; // Seconds
            lastTime = timestamp;

            ctx.clearRect(0, 0, canvas.width / (window.devicePixelRatio || 1), canvas.height / (window.devicePixelRatio || 1));

            // Logic Update
            if (!appState.isManual) {
                // RPM to Radians per second: RPM * 2PI / 60
                const speedRad = (appState.speed1 * Math.PI * 2) / 60;
                appState.angle1 += speedRad * deltaTime;
            }
            // else: angle1 is updated by wheel event
            
            // Gear Ratio i = Z2 / Z1
            const speedRatio = appState.z1 / appState.z2;
            appState.angle2 = -(appState.angle1 * speedRatio);
            
            // Adjust phase to mesh teeth
            const toothPitchAngle = (Math.PI * 2) / appState.z2;
            const meshingOffset = toothPitchAngle / 2;
            
            // Drawing Position
            const r1 = (appState.z1 * MODULE) / 2;
            const r2 = (appState.z2 * MODULE) / 2;
            const totalDist = r1 + r2;
            
            const centerX = width / 2;
            const centerY = height / 2;
            
            // Center gears dynamically
            const gear1X = centerX - (totalDist/2);
            const gear2X = centerX + (totalDist/2);
            
            // Draw Connection Line
            ctx.beginPath();
            ctx.moveTo(gear1X, centerY);
            ctx.lineTo(gear2X, centerY);
            ctx.strokeStyle = '#e5e7eb';
            ctx.lineWidth = 4;
            ctx.stroke();

            // Draw Driver (Blue)
            drawGear(gear1X, centerY, appState.z1, '#bfdbfe', appState.angle1, '駆動', true);
            
            // Draw Driven (Red)
            drawGear(gear2X, centerY, appState.z2, '#fecaca', appState.angle2 + meshingOffset + Math.PI, '被動', false);

            drawShaft(gear1X, centerY);
            drawShaft(gear2X, centerY);

            requestAnimationFrame(animate);
        }

        // --- Logic & Updates ---
        function updateStats() {
            const z1 = parseInt(ui.sliderZ1.value);
            const z2 = parseInt(ui.sliderZ2.value);
            const speed = parseInt(ui.sliderSpeed.value);

            appState.z1 = z1;
            appState.z2 = z2;
            appState.speed1 = speed;

            // Update UI Text
            ui.valZ1.textContent = z1;
            ui.valZ2.textContent = z2;
            ui.valSpeed.textContent = speed;

            // Math
            const ratio = z2 / z1;
            ui.ratioDisplay.textContent = ratio.toFixed(2);
            
            ui.calcZ1.textContent = z1;
            ui.calcZ2.textContent = z2;
            ui.calcResult.textContent = ratio.toFixed(2);

            if (appState.isManual) {
                ui.inputRpm.textContent = "手動 (Manual)";
                ui.outputRpm.textContent = "---";
            } else {
                ui.inputRpm.textContent = `${speed} rpm`;
                const outSpeed = speed * (z1 / z2);
                ui.outputRpm.textContent = `${outSpeed.toFixed(0)} rpm`;
            }
        }

        // --- Event Listeners ---
        ui.sliderZ1.addEventListener('input', updateStats);
        ui.sliderZ2.addEventListener('input', () => {
            updateStats();
            if (appState.mode === 'quiz') checkQuizState(); 
        });
        ui.sliderSpeed.addEventListener('input', updateStats);

        // Manual Rotation Checkbox Logic
        ui.checkManual.addEventListener('change', (e) => {
            appState.isManual = e.target.checked;
            toggleManualModeUI(e.target.checked);
            updateStats();
        });

        function toggleManualModeUI(isManual) {
            if (isManual) {
                ui.speedControlGroup.classList.add('opacity-50', 'pointer-events-none');
                ui.manualIndicator.classList.remove('hidden');
            } else {
                ui.speedControlGroup.classList.remove('opacity-50', 'pointer-events-none');
                ui.manualIndicator.classList.add('hidden');
            }
        }

        // Wheel Event for Manual Rotation
        canvas.addEventListener('wheel', (e) => {
            if (!appState.isManual) return;
            e.preventDefault();
            
            // Adjust sensitivity as needed
            const sensitivity = 0.005; 
            // DeltaY positive is scrolling down -> Increase angle (Clockwise visual)
            appState.angle1 += e.deltaY * sensitivity;
        }, { passive: false });

        // --- Quiz Mode Logic ---
        function switchMode(mode) {
            appState.mode = mode;
            
            // Update Tab Styles
            if (mode === 'simulation') {
                ui.btnSim.className = "px-3 py-1.5 rounded-md text-sm font-medium bg-blue-100 text-blue-700 hover:bg-blue-200 transition";
                ui.btnQuiz.className = "px-3 py-1.5 rounded-md text-sm font-medium text-gray-600 hover:bg-gray-100 transition";
                
                ui.quizOverlay.classList.add('hidden');
                
                // Unlock Controls
                ui.sliderZ1.disabled = false;
                ui.sliderSpeed.disabled = false;
                ui.checkManual.disabled = false; // Enable Manual check
                ui.sliderZ1.parentElement.classList.remove('opacity-50');
                
            } else {
                ui.btnQuiz.className = "px-3 py-1.5 rounded-md text-sm font-medium bg-yellow-100 text-yellow-700 hover:bg-yellow-200 transition";
                ui.btnSim.className = "px-3 py-1.5 rounded-md text-sm font-medium text-gray-600 hover:bg-gray-100 transition";
                
                ui.quizOverlay.classList.remove('hidden');
                
                // Disable Manual Mode if active
                if (appState.isManual) {
                    appState.isManual = false;
                    ui.checkManual.checked = false;
                    toggleManualModeUI(false);
                }
                
                nextQuestion();
            }
        }

        function nextQuestion() {
            // Reset feedback
            ui.quizFeedback.textContent = "";
            ui.quizFeedback.className = "text-sm font-bold h-6 mb-2";

            // Generate Problem
            // Lock Input Gear and Speed to random values
            const qZ1 = [10, 12, 15, 20, 24, 30][Math.floor(Math.random() * 6)];
            const qSpeed = [30, 60, 90, 100][Math.floor(Math.random() * 4)];
            
            ui.sliderZ1.value = qZ1;
            ui.sliderSpeed.value = qSpeed;
            
            // Lock UI for Z1 and Speed
            ui.sliderZ1.disabled = true;
            ui.sliderSpeed.disabled = true;
            ui.checkManual.disabled = true; // Disable manual checkbox in quiz
            ui.sliderZ1.parentElement.classList.add('opacity-50');

            // Determine Goal (Ratio or Speed)
            const type = Math.random() > 0.5 ? 'ratio' : 'speed';
            appState.quizTargetType = type;

            // Pick a valid Z2 that exists in slider range (8-40)
            const possibleZ2s = [];
            for(let i=10; i<=40; i+=2) possibleZ2s.push(i);
            const targetZ2 = possibleZ2s[Math.floor(Math.random() * possibleZ2s.length)];
            
            appState.quizTargetZ2 = targetZ2; // The secret answer

            // Move slider to wrong position initially
            let initialPos = 20;
            if (initialPos === targetZ2) initialPos = 10;
            ui.sliderZ2.value = initialPos;
            updateStats();

            // Formulate Question Text
            if (type === 'ratio') {
                const targetRatio = targetZ2 / qZ1;
                ui.quizQuestion.innerHTML = `
                    駆動歯車 (Z1) は <b>${qZ1}枚</b> です。<br>
                    ギア比を <b>${targetRatio.toFixed(2)}</b> にするには、<br>
                    被動歯車 (Z2) を何枚にすれば良い？
                `;
            } else {
                const targetOutSpeed = qSpeed * (qZ1 / targetZ2);
                ui.quizQuestion.innerHTML = `
                    駆動歯車 (Z1) は <b>${qZ1}枚</b>、入力は <b>${qSpeed}rpm</b> です。<br>
                    出力速度を <b>約${Math.round(targetOutSpeed)}rpm</b> にするには、<br>
                    被動歯車 (Z2) を何枚にすれば良い？
                `;
            }
        }

        function checkAnswer() {
            const currentZ2 = parseInt(ui.sliderZ2.value);
            const correctZ2 = appState.quizTargetZ2;

            if (currentZ2 === correctZ2) {
                ui.quizFeedback.textContent = "正解！素晴らしい！🎉";
                ui.quizFeedback.classList.add("text-green-600");
                
                // Visual celebration (spin faster for a second)
                const origSpeed = appState.speed1;
                appState.speed1 = 300;
                setTimeout(() => { appState.speed1 = origSpeed; }, 1000);
            } else {
                if (currentZ2 < correctZ2) {
                    ui.quizFeedback.textContent = "不正解... もっと歯数を増やそう！";
                } else {
                    ui.quizFeedback.textContent = "不正解... もっと歯数を減らそう！";
                }
                ui.quizFeedback.classList.add("text-red-600");
            }
        }

        function checkQuizState() {
            // Optional: realtime hint
        }

        // Initialize
        updateStats();
        requestAnimationFrame(animate);

    </script>
</body>
</html>
