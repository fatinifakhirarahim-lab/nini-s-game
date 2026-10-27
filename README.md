<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Celebrations - ESL Pronunciation Runner</title>
    <!-- Tailwind CSS for modern responsive styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Font Awesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Canvas Confetti -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Fredoka:wght@400;600;700&family=Nunito:wght@700;800;900&display=swap');

        body {
            font-family: 'Fredoka', sans-serif;
            user-select: none;
            -webkit-user-select: none;
            touch-action: manipulation;
        }

        /* Speech visualizer pulse animation */
        @keyframes pulse-ring {
            0% { transform: scale(0.95); opacity: 0.8; }
            50% { transform: scale(1.15); opacity: 0.4; }
            100% { transform: scale(0.95); opacity: 0.8; }
        }

        .mic-active {
            animation: pulse-ring 1.2s infinite ease-in-out;
        }

        /* Screen shake animation on car crash */
        @keyframes crash-shake {
            0% { transform: translate(0, 0) rotate(0deg); }
            20% { transform: translate(-10px, 5px) rotate(-3deg); }
            40% { transform: translate(10px, -5px) rotate(3deg); }
            60% { transform: translate(-8px, -2px) rotate(-2deg); }
            80% { transform: translate(8px, 2px) rotate(1deg); }
            100% { transform: translate(0, 0) rotate(0deg); }
        }

        .crash-effect {
            animation: crash-shake 0.5s ease-in-out;
        }

        /* Floating text animation */
        @keyframes float-up {
            0% { opacity: 1; transform: translateY(0); }
            100% { opacity: 0; transform: translateY(-40px); }
        }

        .float-text {
            animation: float-up 1.2s ease-out forwards;
        }

        /* Custom Scrollbar for review modal */
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f1f1;
            border-radius: 10px;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 10px;
        }
    </style>
</head>
<body class="bg-slate-900 text-slate-800 h-screen w-screen overflow-hidden flex flex-col justify-between items-center select-none">

    <!-- Top Navigation Bar / Scoreboard -->
    <header class="w-full max-w-4xl px-4 pt-3 pb-1 flex justify-between items-center z-20">
        <!-- Level Tag -->
        <div class="bg-amber-400 border-4 border-amber-600 rounded-2xl px-4 py-1.5 shadow-lg flex items-center gap-2">
            <i class="fa-solid fa-flag-checkered text-amber-900 text-lg"></i>
            <span class="text-amber-950 font-black text-lg md:text-xl uppercase tracking-wider">
                Level <span id="levelDisplay" class="text-2xl text-red-700">1</span>/10
            </span>
        </div>

        <!-- Progress Bar & Mini Map -->
        <div class="flex-1 mx-4 max-w-md hidden sm:block">
            <div class="flex justify-between text-xs font-bold text-slate-300 mb-1 px-1">
                <span>START</span>
                <span>FINISH 🏆</span>
            </div>
            <div class="relative w-full bg-slate-800 h-5 rounded-full border-2 border-slate-600 overflow-visible p-0.5 shadow-inner">
                <div id="progressBar" class="bg-gradient-to-r from-amber-400 to-emerald-400 h-full rounded-full w-0 transition-all duration-500 ease-out"></div>
                <!-- Mini Car Marker -->
                <div id="miniCarMarker" class="absolute -top-2.5 left-0 transition-all duration-500 ease-out transform -translate-x-1/2 drop-shadow-md">
                    <span class="text-xl">🚗</span>
                </div>
            </div>
        </div>

        <!-- Score Badge -->
        <div class="bg-emerald-500 border-4 border-emerald-700 rounded-2xl px-4 py-1.5 shadow-lg flex items-center gap-2">
            <i class="fa-solid fa-star text-yellow-300 text-xl animate-bounce"></i>
            <div class="text-right">
                <span class="text-emerald-950 font-black text-xs block leading-none">SCORE</span>
                <span id="scoreDisplay" class="text-white font-black text-xl md:text-2xl leading-none">0</span>
            </div>
        </div>
    </header>

    <!-- Main Game Scene Display Canvas Container -->
    <main class="relative w-full max-w-4xl flex-1 my-2 px-2 flex flex-col justify-center items-center">
        
        <!-- Interactive Canvas Stage -->
        <div class="relative w-full h-64 sm:h-80 md:h-96 rounded-3xl overflow-hidden border-4 border-slate-700 shadow-2xl bg-sky-300" id="gameStage">
            <!-- HTML5 Canvas for Animated Cartoon World -->
            <canvas id="gameCanvas" class="w-full h-full block"></canvas>

            <!-- Overlay Feedback Banner -->
            <div id="feedbackBanner" class="hidden absolute top-4 left-1/2 transform -translate-x-1/2 px-6 py-2 rounded-2xl border-4 font-black text-center text-lg sm:text-2xl shadow-2xl transition-all duration-300 z-30">
                <span id="feedbackText">Great Job!</span>
            </div>

            <!-- Floating Audio Waveform Indicator when Mic is Active -->
            <div id="recordingIndicator" class="hidden absolute bottom-4 right-4 bg-red-600 text-white px-4 py-2 rounded-full font-bold flex items-center gap-2 shadow-lg z-30 animate-pulse">
                <div class="w-3 h-3 bg-white rounded-full"></div>
                <span>Listening...</span>
            </div>
        </div>

        <!-- Target Sentence Card -->
        <div class="w-full mt-3 bg-white border-4 border-indigo-500 rounded-2xl p-4 sm:p-5 shadow-xl text-center relative z-20">
            <div class="text-xs uppercase font-extrabold tracking-widest text-indigo-500 mb-1 flex justify-center items-center gap-2">
                <span>Read Aloud sentence</span>
                <button id="ttsBtn" title="Listen to sentence" class="text-indigo-600 hover:text-indigo-800 transition-colors p-1 text-base">
                    <i class="fa-solid fa-volume-high"></i>
                </button>
            </div>
            
            <h2 id="targetSentence" class="text-xl sm:text-2xl md:text-3xl font-black text-slate-800 leading-tight">
                "When is Teacher's Day?"
            </h2>

            <!-- Speech Recognition Live Transcript & Feedback -->
            <div id="speechOutputContainer" class="mt-2 min-h-[1.75rem] flex flex-col items-center justify-center">
                <p id="speechResult" class="text-sm sm:text-base font-bold text-slate-500 italic">
                    Tap the microphone button and read sentence clearly...
                </p>
                <p id="accuracyScore" class="text-xs font-extrabold text-indigo-600 mt-1 hidden"></p>
            </div>
        </div>

    </main>

    <!-- Bottom Controls & Interaction Buttons -->
    <footer class="w-full max-w-4xl px-4 pb-4 pt-1 z-20 flex flex-col sm:flex-row gap-3 justify-center items-center">
        
        <!-- Push-To-Talk Microphone Button -->
        <button id="micBtn" class="w-full sm:w-auto flex-1 max-w-xs bg-gradient-to-r from-red-500 to-rose-600 hover:from-red-600 hover:to-rose-700 active:scale-95 text-white border-b-4 border-red-800 rounded-2xl py-3 px-6 font-black text-lg sm:text-xl shadow-lg flex items-center justify-center gap-3 transition-all">
            <div class="relative flex items-center justify-center">
                <span id="micPulse" class="absolute w-10 h-10 bg-red-400 rounded-full opacity-0"></span>
                <i id="micIcon" class="fa-solid fa-microphone text-2xl z-10"></i>
            </div>
            <span id="micText">TAP TO SPEAK</span>
        </button>

        <!-- Retry Button (Visible on incorrect answer or manually) -->
        <button id="retryBtn" class="hidden w-full sm:w-auto bg-amber-500 hover:bg-amber-600 active:scale-95 text-white border-b-4 border-amber-700 rounded-2xl py-3 px-6 font-black text-lg shadow-lg flex items-center justify-center gap-2 transition-all">
            <i class="fa-solid fa-rotate-right"></i>
            <span>TRY AGAIN</span>
        </button>

        <!-- Next Level Button (Hidden until correct pronunciation achieved) -->
        <button id="nextBtn" class="hidden w-full sm:w-auto bg-gradient-to-r from-emerald-500 to-green-600 hover:from-emerald-600 hover:to-green-700 active:scale-95 text-white border-b-4 border-emerald-800 rounded-2xl py-3 px-8 font-black text-xl shadow-2xl flex items-center justify-center gap-3 animate-bounce transition-all">
            <span>NEXT LEVEL</span>
            <i class="fa-solid fa-circle-arrow-right"></i>
        </button>
    </footer>

    <!-- Game Complete Modal -->
    <div id="gameCompleteModal" class="fixed inset-0 bg-slate-900/90 backdrop-blur-md z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-white border-8 border-amber-400 rounded-3xl p-6 sm:p-8 max-w-md w-full text-center shadow-2xl relative transform transition-all scale-100">
            
            <div class="w-24 h-24 bg-amber-100 border-4 border-amber-400 rounded-full flex items-center justify-center mx-auto -mt-16 shadow-xl">
                <span class="text-5xl">🏆</span>
            </div>

            <h1 class="text-3xl sm:text-4xl font-black text-slate-800 mt-4 mb-1">CONGRATULATIONS!</h1>
            <p class="text-slate-500 font-bold text-base mb-4">You completed all celebration levels!</p>

            <!-- Star Rating Display -->
            <div class="flex justify-center gap-2 text-3xl sm:text-4xl text-amber-400 mb-6">
                <i class="fa-solid fa-star transform -rotate-12"></i>
                <i class="fa-solid fa-star transform scale-125"></i>
                <i class="fa-solid fa-star transform rotate-12"></i>
            </div>

            <!-- Score Summary Card -->
            <div class="bg-slate-100 rounded-2xl p-4 mb-6 border-2 border-slate-200">
                <div class="text-sm font-extrabold text-slate-400 uppercase">Final Score</div>
                <div id="finalScore" class="text-4xl sm:text-5xl font-black text-emerald-600">100</div>
                <p class="text-xs font-bold text-slate-500 mt-1">10 / 10 Sentences Pronounced Correctly!</p>
            </div>

            <button id="restartGameBtn" class="w-full bg-gradient-to-r from-indigo-500 to-purple-600 hover:from-indigo-600 hover:to-purple-700 active:scale-95 text-white border-b-4 border-indigo-900 rounded-2xl py-3 font-black text-xl shadow-xl flex items-center justify-center gap-2">
                <i class="fa-solid fa-play"></i>
                <span>PLAY AGAIN</span>
            </button>
        </div>
    </div>

    <!-- Main Game Logic JavaScript -->
    <script>
        // Target Sentences Array (As specified in prompt)
        const TARGET_SENTENCES = [
            "When is Teacher's Day?",
            "Teacher's Day is on the sixteenth of May.",
            "When is Malaysia Day?",
            "Malaysia Day is on the sixteenth of September.",
            "When is National Day?",
            "National Day is on the thirty-first of August.",
            "Really? That's interesting!",
            "Oh, I see. Thank you for telling me.",
            "Sorry, could you say that again, please?",
            "Do you mean Malaysia Day is on the sixteenth of September?"
        ];

        // State variables
        let currentLevel = 0; // 0 to 9 index (Level 1 to 10)
        let score = 0;
        let isRecording = false;
        let speechStartTime = 0;
        let speechDurationMs = 0;
        let carPassedObstacle = false;
        let levelPassed = false;
        let carCrashed = false;

        // Canvas & Animation Engine variables
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        let animationFrameId;
        let worldX = 0;
        let carX = 80;
        let carY = 0;
        let carSpeed = 0;
        let obstacleX = 0;
        let barrierLiftY = 0;
        let wheelRotation = 0;
        let smokeParticles = [];
        let confettiParticles = [];

        // Synthesized Audio Engine (Web Audio API)
        class SoundEffects {
            constructor() {
                this.ctx = null;
            }

            init() {
                if (!this.ctx) {
                    this.ctx = new (window.AudioContext || window.webkitAudioContext)();
                }
                if (this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            }

            playSuccess() {
                this.init();
                const now = this.ctx.currentTime;
                // Upward cheerful arpeggio notes
                const notes = [523.25, 659.25, 783.99, 1046.50]; // C5, E5, G5, C6
                notes.forEach((freq, idx) => {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(freq, now + idx * 0.08);
                    gain.gain.setValueAtTime(0.3, now + idx * 0.08);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + idx * 0.08 + 0.25);
                    osc.connect(gain);
                    gain.connect(this.ctx.destination);
                    osc.start(now + idx * 0.08);
                    osc.stop(now + idx * 0.08 + 0.25);
                });
            }

            playCrash() {
                this.init();
                const now = this.ctx.currentTime;
                // Noise buffer crash sound
                const bufferSize = this.ctx.sampleRate * 0.4;
                const buffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
                const data = buffer.getChannelData(0);
                for (let i = 0; i < bufferSize; i++) {
                    data[i] = Math.random() * 2 - 1;
                }
                const noise = this.ctx.createBufferSource();
                noise.buffer = buffer;

                const filter = this.ctx.createBiquadFilter();
                filter.type = 'lowpass';
                filter.frequency.setValueAtTime(800, now);
                filter.frequency.exponentialRampToValueAtTime(50, now + 0.4);

                const gain = this.ctx.createGain();
                gain.gain.setValueAtTime(0.6, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.4);

                noise.connect(filter);
                filter.connect(gain);
                gain.connect(this.ctx.destination);

                noise.start(now);

                // Add a low boing tone
                const osc = this.ctx.createOscillator();
                const oscGain = this.ctx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(160, now);
                osc.frequency.exponentialRampToValueAtTime(40, now + 0.3);
                oscGain.gain.setValueAtTime(0.4, now);
                oscGain.gain.exponentialRampToValueAtTime(0.01, now + 0.3);
                osc.connect(oscGain);
                oscGain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.3);
            }

            playClick() {
                this.init();
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(600, now);
                osc.frequency.exponentialRampToValueAtTime(200, now + 0.05);
                gain.gain.setValueAtTime(0.2, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.05);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.05);
            }
        }

        const audioFX = new SoundEffects();

        // Web Speech API Initialization
        let recognition = null;
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;

        if (SpeechRecognition) {
            recognition = new SpeechRecognition();
            recognition.continuous = false;
            recognition.interimResults = true;
            recognition.lang = 'en-US';
        }

        // Setup DOM Element References
        const levelDisplay = document.getElementById('levelDisplay');
        const scoreDisplay = document.getElementById('scoreDisplay');
        const progressBar = document.getElementById('progressBar');
        const miniCarMarker = document.getElementById('miniCarMarker');
        const targetSentence = document.getElementById('targetSentence');
        const speechResult = document.getElementById('speechResult');
        const accuracyScore = document.getElementById('accuracyScore');
        const micBtn = document.getElementById('micBtn');
        const micIcon = document.getElementById('micIcon');
        const micText = document.getElementById('micText');
        const micPulse = document.getElementById('micPulse');
        const retryBtn = document.getElementById('retryBtn');
        const nextBtn = document.getElementById('nextBtn');
        const feedbackBanner = document.getElementById('feedbackBanner');
        const feedbackText = document.getElementById('feedbackText');
        const recordingIndicator = document.getElementById('recordingIndicator');
        const ttsBtn = document.getElementById('ttsBtn');
        const gameCompleteModal = document.getElementById('gameCompleteModal');
        const finalScore = document.getElementById('finalScore');
        const restartGameBtn = document.getElementById('restartGameBtn');
        const gameStage = document.getElementById('gameStage');

        // Responsive Canvas Sizing
        function resizeCanvas() {
            canvas.width = gameStage.clientWidth;
            canvas.height = gameStage.clientHeight;
            carY = canvas.height - 75; // Align car with road line
            obstacleX = canvas.width - 150;
        }

        window.addEventListener('resize', resizeCanvas);

        // Helper: Text Normalizer & Number-to-Words Converter for robust speech matching
        function normalizeText(text) {
            if (!text) return "";
            let str = text.toLowerCase().trim();
            
            // Replace common punctuation
            str = str.replace(/[.,\/#!$%\^&\*;:{}=\-_`~()?'"]/g, "");
            
            // Map ordinal/cardinal digit strings to full spoken words
            const numberMap = {
                "16th": "sixteenth", "16": "sixteenth",
                "31st": "thirty first", "31": "thirty first",
                "1st": "first", "2nd": "second", "3rd": "third"
            };

            for (const [key, val] of Object.entries(numberMap)) {
                str = str.replace(new RegExp(`\\b${key}\\b`, 'g'), val);
            }

            return str.replace(/\s+/g, ' ');
        }

        // Fuzzy String Distance Algorithm (Levenshtein Distance Similarity)
        function calculateSimilarity(str1, str2) {
            const s1 = normalizeText(str1);
            const s2 = normalizeText(str2);

            if (s1 === s2) return 1.0;
            if (s1.length === 0 || s2.length === 0) return 0.0;

            const matrix = [];

            for (let i = 0; i <= s2.length; i++) {
                matrix[i] = [i];
            }
            for (let j = 0; j <= s1.length; j++) {
                matrix[0][j] = j;
            }

            for (let i = 1; i <= s2.length; i++) {
                for (let j = 1; j <= s1.length; j++) {
                    if (s2.charAt(i - 1) === s1.charAt(j - 1)) {
                        matrix[i][j] = matrix[i - 1][j - 1];
                    } else {
                        matrix[i][j] = Math.min(
                            matrix[i - 1][j - 1] + 1, // substitution
                            matrix[i][j - 1] + 1,     // insertion
                            matrix[i - 1][j] + 1      // deletion
                        );
                    }
                }
            }

            const distance = matrix[s2.length][s1.length];
            const maxLength = Math.max(s1.length, s2.length);
            return (maxLength - distance) / maxLength;
        }

        // Text-To-Speech Pronunciation Helper for ESL students
        function speakTargetSentence() {
            if ('speechSynthesis' in window) {
                window.speechSynthesis.cancel();
                const utterance = new SpeechSynthesisUtterance(TARGET_SENTENCES[currentLevel]);
                utterance.rate = 0.85; // Slightly slower for clear ESL learning
                utterance.lang = 'en-US';
                window.speechSynthesis.speak(utterance);
            }
        }

        ttsBtn.addEventListener('click', () => {
            audioFX.playClick();
            speakTargetSentence();
        });

        // Initialize / Load Level State
        function loadLevel(levelIdx) {
            currentLevel = levelIdx;
            carPassedObstacle = false;
            levelPassed = false;
            carCrashed = false;
            carX = 80;
            barrierLiftY = 0;
            smokeParticles = [];
            
            // UI Updates
            levelDisplay.textContent = currentLevel + 1;
            targetSentence.textContent = `"${TARGET_SENTENCES[currentLevel]}"`;
            speechResult.textContent = "Tap microphone and read sentence clearly...";
            speechResult.className = "text-sm sm:text-base font-bold text-slate-500 italic";
            accuracyScore.classList.add('hidden');
            
            // Hide Next Level button and Retry
            nextBtn.classList.add('hidden');
            retryBtn.classList.add('hidden');
            micBtn.classList.remove('hidden');

            // Feedback reset
            feedbackBanner.classList.add('hidden');

            // Progress bar update
            const progressPercent = (currentLevel / TARGET_SENTENCES.length) * 100;
            progressBar.style.width = `${progressPercent}%`;
            miniCarMarker.style.left = `${progressPercent}%`;
        }

        // Speech Recognition setup & Audio constraints handler
        if (recognition) {
            recognition.onstart = () => {
                isRecording = true;
                speechStartTime = Date.now();
                micBtn.classList.add('bg-red-700');
                micPulse.classList.add('mic-active');
                micText.textContent = "LISTENING...";
                recordingIndicator.classList.remove('hidden');
                speechResult.textContent = "Listening to your voice...";
                speechResult.className = "text-sm sm:text-base font-bold text-indigo-600 animate-pulse";
            };

            recognition.onresult = (event) => {
                let transcript = '';
                for (let i = event.resultIndex; i < event.results.length; i++) {
                    transcript += event.results[i][0].transcript;
                }
                speechResult.textContent = `"${transcript}"`;
            };

            recognition.onerror = (event) => {
                console.warn("Speech recognition error:", event.error);
                stopRecordingUI();
                if (event.error !== 'no-speech') {
                    showFeedback("Please speak clearly and try again", "warning");
                }
            };

            recognition.onend = () => {
                stopRecordingUI();
                speechDurationMs = Date.now() - speechStartTime;
                evaluateSpeech();
            };
        }

        function stopRecordingUI() {
            isRecording = false;
            micBtn.classList.remove('bg-red-700');
            micPulse.classList.remove('mic-active');
            micText.textContent = "TAP TO SPEAK";
            recordingIndicator.classList.add('hidden');
        }

        // Toggle Microphone Push-to-Talk
        micBtn.addEventListener('click', async () => {
            audioFX.playClick();

            if (!SpeechRecognition) {
                alert("Speech recognition is not supported in this browser. Please try Chrome or Safari.");
                return;
            }

            if (isRecording) {
                recognition.stop();
                return;
            }

            // Request Audio Stream with strict Noise Reduction Constraints
            try {
                await navigator.mediaDevices.getUserMedia({
                    audio: {
                        noiseSuppression: true,
                        echoCancellation: true,
                        autoGainControl: true
                    }
                });
                recognition.start();
            } catch (err) {
                console.error("Microphone access denied:", err);
                speechResult.textContent = "Microphone access is required to play!";
                speechResult.className = "text-sm font-bold text-red-600";
            }
        });

        // Speech Evaluation Engine
        function evaluateSpeech() {
            const spokenText = speechResult.textContent.replace(/^"|"$/g, '').trim();

            // Noise Reduction & Duration Handling:
            // Check if speech duration is too short (< 1.2s) or no meaningful speech detected
            if (!spokenText || spokenText === "Listening to your voice..." || speechDurationMs < 1000) {
                showFeedback("Please speak clearly and try again", "warning");
                speechResult.textContent = "Audio too short or quiet. Try speaking standard and clearly.";
                speechResult.className = "text-sm font-bold text-amber-600";
                retryBtn.classList.remove('hidden');
                return;
            }

            const targetText = TARGET_SENTENCES[currentLevel];
            const similarity = calculateSimilarity(spokenText, targetText);
            const similarityPercent = Math.round(similarity * 100);

            accuracyScore.textContent = `Match Accuracy: ${similarityPercent}%`;
            accuracyScore.classList.remove('hidden');

            // Tolerance threshold set to approx 75-80% for friendly ESL tolerance
            if (similarity >= 0.75) {
                // Correct Pronunciation Logic
                handleSuccess(similarityPercent);
            } else {
                // Incorrect Pronunciation Logic
                handleFailure(similarityPercent);
            }
        }

        // Handle Correct Pronunciation
        function handleSuccess(percent) {
            if (levelPassed) return;
            levelPassed = true;
            
            // Award points
            score += 10;
            scoreDisplay.textContent = score;

            // Audio & FX
            audioFX.playSuccess();
            showFeedback(`Correct! +10 Points (${percent}% match)`, "success");

            // Car animation status
            carPassedObstacle = true;

            // Trigger Confetti Celebration
            confetti({
                particleCount: 75,
                spread: 70,
                origin: { y: 0.6 }
            });

            // Reveal Next Level Button (DO NOT AUTO ADVANCE)
            micBtn.classList.add('hidden');
            retryBtn.classList.add('hidden');
            nextBtn.classList.remove('hidden');
        }

        // Handle Failure / Crash
        function handleFailure(percent) {
            carCrashed = true;
            audioFX.playCrash();
            
            // Apply crash screen shake
            gameStage.classList.add('crash-effect');
            setTimeout(() => gameStage.classList.remove('crash-effect'), 500);

            // Generate smoke particles
            createSmokeParticles();

            showFeedback("Try Again!", "error");
            retryBtn.classList.remove('hidden');
        }

        // Feedback Banner display
        function showFeedback(msg, type) {
            feedbackText.textContent = msg;
            feedbackBanner.classList.remove('hidden', 'bg-emerald-500', 'bg-red-500', 'bg-amber-500', 'text-white', 'border-emerald-700', 'border-red-700', 'border-amber-700');

            if (type === 'success') {
                feedbackBanner.classList.add('bg-emerald-500', 'text-white', 'border-emerald-700');
            } else if (type === 'error') {
                feedbackBanner.classList.add('bg-red-500', 'text-white', 'border-red-700');
            } else {
                feedbackBanner.classList.add('bg-amber-500', 'text-white', 'border-amber-700');
            }
        }

        // Retry Button Handler
        retryBtn.addEventListener('click', () => {
            audioFX.playClick();
            carCrashed = false;
            speechResult.textContent = "Tap microphone and read sentence clearly...";
            speechResult.className = "text-sm sm:text-base font-bold text-slate-500 italic";
            accuracyScore.classList.add('hidden');
            feedbackBanner.classList.add('hidden');
            retryBtn.classList.add('hidden');
        });

        // Next Level Button Handler
        nextBtn.addEventListener('click', () => {
            audioFX.playClick();
            if (currentLevel < TARGET_SENTENCES.length - 1) {
                loadLevel(currentLevel + 1);
            } else {
                // Final Level Passed - Complete Game
                showGameComplete();
            }
        });

        function showGameComplete() {
            finalScore.textContent = score;
            progressBar.style.width = '100%';
            miniCarMarker.style.left = '100%';
            gameCompleteModal.classList.remove('hidden');
            
            // Big Confetti
            confetti({
                particleCount: 150,
                spread: 100,
                origin: { y: 0.5 }
            });
        }

        restartGameBtn.addEventListener('click', () => {
            audioFX.playClick();
            score = 0;
            scoreDisplay.textContent = 0;
            gameCompleteModal.classList.add('hidden');
            loadLevel(0);
        });

        // -------------------------------------------------------------
        // Canvas Rendering Engine (Cartoon City, Car & Animated Obstacles)
        // -------------------------------------------------------------

        function createSmokeParticles() {
            smokeParticles = [];
            for (let i = 0; i < 20; i++) {
                smokeParticles.push({
                    x: carX + 60 + (Math.random() * 20 - 10),
                    y: carY + 20,
                    vx: (Math.random() - 0.5) * 3,
                    vy: -Math.random() * 3 - 1,
                    radius: Math.random() * 8 + 6,
                    alpha: 1
                });
            }
        }

        function drawBackground() {
            // Sky Gradient
            const skyGradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
            skyGradient.addColorStop(0, '#38bdf8');
            skyGradient.addColorStop(0.7, '#bae6fd');
            skyGradient.addColorStop(1, '#e0f2fe');
            ctx.fillStyle = skyGradient;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Sun
            ctx.fillStyle = '#fde047';
            ctx.beginPath();
            ctx.arc(60, 50, 30, 0, Math.PI * 2);
            ctx.fill();

            // Scrolling Clouds
            ctx.fillStyle = 'rgba(255, 255, 255, 0.85)';
            const cloudPositions = [100, 320, 580, 800];
            cloudPositions.forEach(cx => {
                let x = (cx - worldX * 0.2) % (canvas.width + 200) - 100;
                ctx.beginPath();
                ctx.arc(x, 40, 18, 0, Math.PI * 2);
                ctx.arc(x + 20, 35, 25, 0, Math.PI * 2);
                ctx.arc(x + 45, 40, 18, 0, Math.PI * 2);
                ctx.fill();
            });

            // City Buildings Silhouette Layer
            const buildingColors = ['#f472b6', '#38bdf8', '#fbbf24', '#a78bfa', '#34d399'];
            const buildingHeights = [110, 140, 95, 130, 105, 150, 120];
            let bx = -(worldX * 0.4) % 120;
            
            let bIdx = 0;
            while (bx < canvas.width + 120) {
                const height = buildingHeights[bIdx % buildingHeights.length];
                ctx.fillStyle = buildingColors[bIdx % buildingColors.length];
                ctx.fillRect(bx, canvas.height - 60 - height, 75, height);
                
                // Building Windows
                ctx.fillStyle = '#fef08a';
                for (let wy = canvas.height - 50 - height; wy < canvas.height - 70; wy += 22) {
                    for (let wx = bx + 10; wx < bx + 60; wx += 20) {
                        ctx.fillRect(wx, wy, 12, 14);
                    }
                }
                bx += 75;
                bIdx++;
            }

            // Road & Ground
            ctx.fillStyle = '#1e293b'; // Asphalt
            ctx.fillRect(0, canvas.height - 60, canvas.width, 60);

            // Curb
            ctx.fillStyle = '#ef4444';
            ctx.fillRect(0, canvas.height - 65, canvas.width, 5);

            // Animated Road Line Stripes
            ctx.strokeStyle = '#facc15';
            ctx.lineWidth = 4;
            ctx.setLineDash([25, 20]);
            ctx.lineDashOffset = -worldX;
            ctx.beginPath();
            ctx.moveTo(0, canvas.height - 30);
            ctx.lineTo(canvas.width, canvas.height - 30);
            ctx.stroke();
            ctx.setLineDash([]); // reset line dash
        }

        function drawObstacle() {
            const bx = obstacleX;
            const by = canvas.height - 60;

            // Barrier Posts
            ctx.fillStyle = '#475569';
            ctx.fillRect(bx, by - 70, 12, 70);

            // Barrier Lift Gate Arm
            ctx.save();
            ctx.translate(bx + 6, by - 60);

            // If correct pronunciation achieved, lift barrier arm smoothly
            if (carPassedObstacle && barrierLiftY < 75) {
                barrierLiftY += 2;
            }

            ctx.rotate((-barrierLiftY * Math.PI) / 180);

            // Striped Toll Barrier Arm
            ctx.fillStyle = '#f87171';
            ctx.fillRect(0, -8, -120, 16);
            ctx.fillStyle = '#ffffff';
            for (let i = -110; i < 0; i += 30) {
                ctx.fillRect(i, -8, 15, 16);
            }

            ctx.restore();

            // STOP Sign
            ctx.fillStyle = '#dc2626';
            ctx.beginPath();
            ctx.arc(bx + 6, by - 85, 18, 0, Math.PI * 2);
            ctx.fill();
            ctx.lineWidth = 3;
            ctx.strokeStyle = '#ffffff';
            ctx.stroke();

            ctx.fillStyle = '#ffffff';
            ctx.font = '900 11px Fredoka, sans-serif';
            ctx.textAlign = 'center';
            ctx.fillText('STOP', bx + 6, by - 81);
        }

        function drawCar(x, y) {
            ctx.save();
            ctx.translate(x, y);

            // Slight driving bounce
            const bounce = (!carCrashed && carSpeed > 0) ? Math.sin(Date.now() * 0.015) * 2 : 0;
            ctx.translate(0, bounce);

            // Crash Tilt effect
            if (carCrashed) {
                ctx.rotate(-0.15);
            }

            // Car Shadow
            ctx.fillStyle = 'rgba(0,0,0,0.25)';
            ctx.beginPath();
            ctx.ellipse(35, 38, 40, 8, 0, 0, Math.PI * 2);
            ctx.fill();

            // Car Body Base
            ctx.fillStyle = '#2563eb'; // Royal Blue Body
            ctx.beginPath();
            ctx.roundRect(0, 10, 70, 25, 10);
            ctx.fill();

            // Car Roof / Cabin
            ctx.fillStyle = '#60a5fa';
            ctx.beginPath();
            ctx.roundRect(15, -6, 40, 20, 8);
            ctx.fill();

            // Car Windows
            ctx.fillStyle = '#e0f2fe';
            ctx.fillRect(38, -2, 14, 12);
            ctx.fillRect(18, -2, 16, 12);

            // Headlight
            ctx.fillStyle = '#fef08a';
            ctx.beginPath();
            ctx.arc(68, 18, 5, 0, Math.PI * 2);
            ctx.fill();

            // Wheels
            const drawWheel = (wx, wy) => {
                ctx.save();
                ctx.translate(wx, wy);
                ctx.rotate(wheelRotation);
                ctx.fillStyle = '#1e293b';
                ctx.beginPath();
                ctx.arc(0, 0, 10, 0, Math.PI * 2);
                ctx.fill();
                ctx.fillStyle = '#94a3b8';
                ctx.beginPath();
                ctx.arc(0, 0, 4, 0, Math.PI * 2);
                ctx.fill();
                // Wheel spokes
                ctx.strokeStyle = '#ffffff';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.moveTo(-7, 0); ctx.lineTo(7, 0);
                ctx.moveTo(0, -7); ctx.lineTo(0, 7);
                ctx.stroke();
                ctx.restore();
            };

            drawWheel(18, 32);
            drawWheel(54, 32);

            ctx.restore();
        }

        function drawSmoke() {
            smokeParticles.forEach((p, index) => {
                ctx.fillStyle = `rgba(100, 116, 139, ${p.alpha})`;
                ctx.beginPath();
                ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
                ctx.fill();

                p.x += p.vx;
                p.y += p.vy;
                p.alpha -= 0.02;
                p.radius += 0.3;

                if (p.alpha <= 0) {
                    smokeParticles.splice(index, 1);
                }
            });

            // Continuously generate smoke if crashed
            if (carCrashed && Math.random() < 0.3) {
                smokeParticles.push({
                    x: carX + 60,
                    y: carY + 15,
                    vx: (Math.random() - 0.5) * 2,
                    vy: -Math.random() * 2 - 1,
                    radius: Math.random() * 6 + 4,
                    alpha: 0.9
                });
            }
        }

        // Game Animation Loop
        function gameLoop() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Car Motion Physics Logic
            if (carPassedObstacle) {
                // Drive past the barrier
                carSpeed = 4;
                carX += carSpeed;
                worldX += carSpeed;
                wheelRotation += 0.2;
            } else if (!carCrashed) {
                // Drive up towards the obstacle stop line
                const stopX = obstacleX - 90;
                if (carX < stopX) {
                    carSpeed = 2.5;
                    carX += carSpeed;
                    worldX += carSpeed;
                    wheelRotation += 0.15;
                } else {
                    carSpeed = 0;
                }
            } else {
                carSpeed = 0;
            }

            // Render Layers
            drawBackground();
            drawObstacle();
            drawCar(carX, carY);
            drawSmoke();

            animationFrameId = requestAnimationFrame(gameLoop);
        }

        // Initialize Game Canvas & First Level
        window.onload = () => {
            resizeCanvas();
            loadLevel(0);
            gameLoop();
        };
    </script>
</body>
</html>
