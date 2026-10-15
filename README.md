<!DOCTYPE html>
<html lang="zh-HK">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>奶蛙無限跳 Creamy Frog Endless Jump</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&family=Noto+Sans+TC:wght@700;900&display=swap" rel="stylesheet">
    <style>
        :root {
            --frog-yellow: #F9D96C;
            --frog-dark-yellow: #E5B942;
            --frog-belly: #F8E9C4;
            --frog-eye-ring: #63A375;
            --frog-feet: #5C5238;
        }
        * {
            box-sizing: border-box;
            user-select: none;
            -webkit-user-select: none;
            -webkit-touch-callout: none;
        }
        body {
            margin: 0;
            padding: 0;
            background-color: #161925;
            background-image: 
                radial-gradient(#272c3f 2px, transparent 2px),
                radial-gradient(#1f2435 2px, #161925 2px);
            background-size: 32px 32px;
            background-position: 0 0, 16px 16px;
            font-family: 'Noto Sans TC', sans-serif;
            overflow: hidden;
            touch-action: none;
        }
        .pixel-font {
            font-family: 'Press Start 2P', 'Noto Sans TC', monospace;
        }
        canvas {
            image-rendering: -moz-crisp-edges;
            image-rendering: -webkit-crisp-edges;
            image-rendering: pixelated;
            image-rendering: crisp-edges;
            display: block;
        }
        /* Retro Arcade Pixel Box Border */
        .pixel-box {
            background: #232733;
            border: 4px solid #F9D96C;
            box-shadow: 
                0 6px 0 #8F6818,
                0 10px 25px rgba(0,0,0,0.6),
                inset 0 0 0 4px #373E52;
        }
        .pixel-btn {
            background: linear-gradient(180deg, #F9D96C 0%, #E5B942 100%);
            border: 4px solid #4A3910;
            box-shadow: 0 6px 0 #4A3910, inset 0 3px 0 #FFF3C4;
            color: #2B2108;
            transition: transform 0.08s ease, box-shadow 0.08s ease;
            cursor: pointer;
        }
        .pixel-btn:hover {
            filter: brightness(1.06);
        }
        .pixel-btn:active {
            transform: translateY(4px);
            box-shadow: 0 2px 0 #4A3910, inset 0 2px 0 #FFF3C4;
        }
        .touch-btn {
            background: rgba(249, 217, 108, 0.22);
            border: 3px solid rgba(249, 217, 108, 0.65);
            box-shadow: 0 4px 0 rgba(92, 82, 56, 0.8);
            backdrop-filter: blur(4px);
        }
        .touch-btn:active, .touch-btn.active {
            background: rgba(249, 217, 108, 0.55);
            transform: translateY(3px);
            box-shadow: 0 1px 0 rgba(92, 82, 56, 0.8);
        }
        @keyframes floatSlow {
            0%, 100% { transform: translateY(0px); }
            50% { transform: translateY(-8px); }
        }
        .animate-float {
            animation: floatSlow 2.4s ease-in-out infinite;
        }
    </style>
</head>
<body class="h-screen w-screen flex flex-col items-center justify-center p-0 sm:p-3">

    <!-- Arcade Cabinet Outer Wrapper -->
    <div class="relative w-full h-full sm:max-w-[440px] sm:max-h-[820px] flex flex-col bg-[#1b1f2b] sm:rounded-2xl sm:border-4 sm:border-[#F9D96C] sm:shadow-[0_12px_40px_rgba(0,0,0,0.85)] overflow-hidden">
        
        <!-- Top HUD Bar -->
        <header class="w-full bg-[#232838] border-b-4 border-[#3c445c] px-3 py-2 flex items-center justify-between z-20 shrink-0">
            <div class="flex items-center space-x-2.5 sm:space-x-3">
                <div>
                    <div class="text-[10px] text-[#9aa5c4] font-bold tracking-wider">分數 SCORE</div>
                    <div id="scoreDisplay" class="pixel-font text-sm sm:text-base text-[#F9D96C] drop-shadow-[0_2px_0_#000]">0m</div>
                </div>
                <div class="h-7 w-[2px] bg-[#3c445c]"></div>
                <div>
                    <div class="text-[10px] text-[#9aa5c4] font-bold tracking-wider">踏死蚊 STOMP</div>
                    <div id="stompDisplay" class="pixel-font text-sm sm:text-base text-[#FF7B7B] drop-shadow-[0_2px_0_#000]">0</div>
                </div>
                <div class="h-7 w-[2px] bg-[#3c445c]"></div>
                <div>
                    <div class="text-[10px] text-[#9aa5c4] font-bold tracking-wider">最高 BEST</div>
                    <div id="bestDisplay" class="pixel-font text-xs sm:text-sm text-[#7CE595] drop-shadow-[0_2px_0_#000]">0m</div>
                </div>
            </div>

            <!-- Status Badges & Audio Controls -->
            <div class="flex items-center space-x-2">
                <div id="shieldIndicator" class="hidden px-2 py-1 rounded bg-[#2c5e7a] border-2 border-[#7ad7ff] text-[11px] text-[#d6f5ff] font-bold flex items-center space-x-1 animate-pulse">
                    <span>🫧</span>
                    <span>護盾</span>
                </div>
                <button id="soundToggleBtn" class="px-2.5 py-1.5 rounded-lg bg-[#32394f] hover:bg-[#3f4863] border-2 border-[#596587] text-xs text-white font-bold flex items-center space-x-1 active:translate-y-0.5 transition" title="Toggle 8-Bit Sound">
                    <span id="soundIcon">🔊</span>
                    <span class="hidden xs:inline">音效</span>
                </button>
            </div>
        </header>

        <!-- Main Game Viewport Container -->
        <div id="canvasContainer" class="relative flex-1 w-full bg-[#87CEEB] overflow-hidden">
            <canvas id="gameCanvas" class="w-full h-full"></canvas>

            <!-- Combo / Milestone Toast Notification -->
            <div id="milestoneToast" class="pointer-events-none absolute top-4 left-1/2 -translate-x-1/2 px-4 py-1.5 rounded-full bg-[#1e2330]/90 border-2 border-[#F9D96C] text-[#F9D96C] font-black text-xs sm:text-sm tracking-wide opacity-0 transition-all duration-300 transform -translate-y-3 z-20 whitespace-nowrap shadow-lg">
                🚀 突破 500m！蚊群加速！
            </div>

            <!-- Start Screen Overlay -->
            <div id="startModal" class="absolute inset-0 bg-[#131722]/85 backdrop-blur-xs flex flex-col items-center justify-center p-4 z-30">
                <div class="pixel-box rounded-2xl p-5 w-full max-w-[360px] flex flex-col items-center text-center">
                    
                    <div class="inline-block px-3 py-1 rounded-full bg-[#353d54] border border-[#F9D96C]/50 text-[#F9D96C] text-xs font-bold mb-2">
                        2D PIXEL ENDLESS JUMPER
                    </div>
                    <h1 class="text-2xl sm:text-3xl font-black text-[#F9D96C] tracking-wider drop-shadow-[0_3px_0_#4A3910] mb-1">
                        奶蛙無限跳
                    </h1>
                    <p class="pixel-font text-[10px] text-[#a7b4d6] mb-3">CREAMY FROG JUMP</p>

                    <!-- Large Preview Canvas of the Creamy Frog & Mosquito -->
                    <div class="relative w-36 h-28 bg-[#2c3449] rounded-xl border-2 border-[#4d5978] flex items-center justify-center mb-3 overflow-hidden shadow-inner">
                        <canvas id="previewCanvas" width="144" height="112" class="w-36 h-28 animate-float"></canvas>
                        <div class="absolute bottom-1 right-2 text-[10px] text-[#8fa0c9] font-bold">LV.99 肌力奶蛙</div>
                    </div>

                    <!-- Game Guide (Bilingual) -->
                    <div class="w-full bg-[#1b1f2a] rounded-xl p-3 border-2 border-[#353d52] text-left text-xs space-y-2 mb-4">
                        <div class="flex items-start space-x-2">
                            <span class="text-base leading-none">🐸</span>
                            <div>
                                <span class="text-[#F9D96C] font-bold">無限向上跳 (Endless Jump)：</span>
                                <span class="text-gray-300">踩中踏板自動向上彈跳，穿過左右邊界可瞬移！</span>
                            </div>
                        </div>
                        <div class="flex items-start space-x-2">
                            <span class="text-base leading-none">🦟</span>
                            <div>
                                <span class="text-[#ff7b7b] font-bold">由上踩死惡蚊 (Stomp Mosquitoes)：</span>
                                <span class="text-gray-300">從下方撞到蚊子會失敗，但<span class="text-[#F9D96C] font-bold">由上往下踩中蚊頭</span>可將牠踏死、高彈跳並獲 <span class="text-[#7CE595] font-bold">+50m</span> 獎勵！亦可拾取 <span class="text-[#7ad7ff] font-bold">🫧護盾</span> 或踩 <span class="text-[#ff9be2] font-bold">🍄蘑菇</span> 衝刺！</span>
                            </div>
                        </div>
                        <div class="flex items-start space-x-2">
                            <span class="text-base leading-none">🎮</span>
                            <div>
                                <span class="text-[#7CE595] font-bold">操作方式 (Controls)：</span>
                                <span class="text-gray-300">鍵盤 <kbd class="px-1 py-0.5 bg-gray-700 rounded text-[10px]">←</kbd> <kbd class="px-1 py-0.5 bg-gray-700 rounded text-[10px]">→</kbd> / <kbd class="px-1 py-0.5 bg-gray-700 rounded text-[10px]">A</kbd> <kbd class="px-1 py-0.5 bg-gray-700 rounded text-[10px]">D</kbd>、滑鼠拖曳、或下方方向鍵。</span>
                            </div>
                        </div>
                    </div>

                    <button id="startBtn" class="pixel-btn w-full py-3.5 rounded-xl font-black text-base sm:text-lg tracking-wider flex items-center justify-center space-x-2">
                        <span>🚀 開始跳躍 START JUMP</span>
                    </button>
                </div>
            </div>

            <!-- Game Over Modal Overlay -->
            <div id="gameOverModal" class="hidden absolute inset-0 bg-[#131722]/85 backdrop-blur-xs flex flex-col items-center justify-center p-4 z-30">
                <div class="pixel-box rounded-2xl p-5 w-full max-w-[340px] flex flex-col items-center text-center">
                    <div id="deathReasonBadge" class="px-3 py-1 rounded-full bg-[#59232a] border border-[#ff6b6b] text-[#ff9e9e] text-xs font-bold mb-2">
                        🦟 被蚊子叮中了！
                    </div>
                    <h2 class="text-2xl font-black text-white drop-shadow-[0_3px_0_#000] mb-1">
                        跳躍結束 GAME OVER
                    </h2>
                    <p class="text-xs text-[#9aa5c4] mb-4">奶蛙還想跳得更高！</p>

                    <!-- Score Card -->
                    <div class="w-full grid grid-cols-3 gap-2 bg-[#181c26] p-3 rounded-xl border-2 border-[#373f54] mb-4">
                        <div class="flex flex-col items-center border-r border-[#373f54] pr-1">
                            <span class="text-[10px] text-[#9aa5c4] font-bold mb-1">總分 SCORE</span>
                            <span id="finalScoreText" class="pixel-font text-sm sm:text-base text-[#F9D96C]">0m</span>
                        </div>
                        <div class="flex flex-col items-center border-r border-[#373f54] px-1">
                            <span class="text-[10px] text-[#9aa5c4] font-bold mb-1">踏死蚊 STOMP</span>
                            <span id="finalStompText" class="pixel-font text-sm sm:text-base text-[#FF7B7B]">0</span>
                        </div>
                        <div class="flex flex-col items-center pl-1">
                            <span class="text-[10px] text-[#9aa5c4] font-bold mb-1">最高 BEST</span>
                            <span id="finalBestText" class="pixel-font text-sm sm:text-base text-[#7CE595]">0m</span>
                        </div>
                    </div>

                    <button id="restartBtn" class="pixel-btn w-full py-3.5 rounded-xl font-black text-base tracking-wider flex items-center justify-center space-x-2">
                        <span>🔄 再跳一次 PLAY AGAIN</span>
                    </button>
                </div>
            </div>

            <!-- On-Screen Touch / Click Controls at Bottom of Canvas -->
            <div id="touchControlsBar" class="absolute bottom-3 inset-x-3 flex items-center justify-between pointer-events-none z-20">
                <button id="btnLeft" class="touch-btn pointer-events-auto w-20 h-14 sm:w-24 sm:h-14 rounded-2xl flex items-center justify-center text-2xl font-black text-[#F9D96C] select-none" aria-label="Move Left">
                    ◀
                </button>
                <div class="text-[11px] font-bold text-white/75 bg-black/35 px-3 py-1 rounded-full backdrop-blur-xs pointer-events-none">
                    按住方向或拖曳畫面移動
                </div>
                <button id="btnRight" class="touch-btn pointer-events-auto w-20 h-14 sm:w-24 sm:h-14 rounded-2xl flex items-center justify-center text-2xl font-black text-[#F9D96C] select-none" aria-label="Move Right">
                    ▶
                </button>
            </div>
        </div>
    </div>

    <script>
        /**
         * 8-Bit Retro Sound Effect Synthesizer (No external audio files required)
         */
        class RetroAudio {
            constructor() {
                this.ctx = null;
                this.enabled = true;
            }

            init() {
                if (!this.ctx) {
                    const AudioCtx = window.AudioContext || window.webkitAudioContext;
                    if (AudioCtx) {
                        this.ctx = new AudioCtx();
                    }
                }
                if (this.ctx && this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            }

            toggle() {
                this.enabled = !this.enabled;
                return this.enabled;
            }

            playJump() {
                if (!this.enabled) return;
                this.init();
                if (!this.ctx) return;

                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'square';
                osc.frequency.setValueAtTime(190, now);
                osc.frequency.exponentialRampToValueAtTime(540, now + 0.13);

                gain.gain.setValueAtTime(0.12, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.14);

                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.14);
            }

            playSuperSpring() {
                if (!this.enabled) return;
                this.init();
                if (!this.ctx) return;

                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'triangle';
                osc.frequency.setValueAtTime(240, now);
                osc.frequency.exponentialRampToValueAtTime(880, now + 0.28);

                gain.gain.setValueAtTime(0.22, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.30);

                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.30);
            }

            playCrack() {
                if (!this.enabled) return;
                this.init();
                if (!this.ctx) return;

                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(140, now);
                osc.frequency.linearRampToValueAtTime(55, now + 0.12);

                gain.gain.setValueAtTime(0.14, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.13);

                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.13);
            }

            playShieldGet() {
                if (!this.enabled) return;
                this.init();
                if (!this.ctx) return;

                const now = this.ctx.currentTime;
                const notes = [523.25, 659.25, 783.99, 1046.5];
                notes.forEach((freq, idx) => {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(freq, now + idx * 0.05);
                    gain.gain.setValueAtTime(0.15, now + idx * 0.05);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + idx * 0.05 + 0.1);
                    osc.connect(gain);
                    gain.connect(this.ctx.destination);
                    osc.start(now + idx * 0.05);
                    osc.stop(now + idx * 0.05 + 0.1);
                });
            }

            playShieldBlock() {
                if (!this.enabled) return;
                this.init();
                if (!this.ctx) return;

                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'square';
                osc.frequency.setValueAtTime(420, now);
                osc.frequency.exponentialRampToValueAtTime(160, now + 0.18);

                gain.gain.setValueAtTime(0.18, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.19);

                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.19);
            }

            playStompMosquito() {
                if (!this.enabled) return;
                this.init();
                if (!this.ctx) return;

                const now = this.ctx.currentTime;
                // Punchy squash impact + high arcade bounce arpeggio
                const osc1 = this.ctx.createOscillator();
                const gain1 = this.ctx.createGain();
                osc1.type = 'square';
                osc1.frequency.setValueAtTime(160, now);
                osc1.frequency.exponentialRampToValueAtTime(680, now + 0.14);
                gain1.gain.setValueAtTime(0.22, now);
                gain1.gain.exponentialRampToValueAtTime(0.001, now + 0.15);
                osc1.connect(gain1);
                gain1.connect(this.ctx.destination);
                osc1.start(now);
                osc1.stop(now + 0.15);

                const osc2 = this.ctx.createOscillator();
                const gain2 = this.ctx.createGain();
                osc2.type = 'triangle';
                osc2.frequency.setValueAtTime(587.33, now + 0.08);
                osc2.frequency.setValueAtTime(880, now + 0.14);
                gain2.gain.setValueAtTime(0.18, now + 0.08);
                gain2.gain.exponentialRampToValueAtTime(0.001, now + 0.24);
                osc2.connect(gain2);
                gain2.connect(this.ctx.destination);
                osc2.start(now + 0.08);
                osc2.stop(now + 0.24);
            }

            playMosquitoHit() {
                if (!this.enabled) return;
                this.init();
                if (!this.ctx) return;

                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(320, now);
                osc.frequency.linearRampToValueAtTime(90, now + 0.45);

                gain.gain.setValueAtTime(0.2, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.46);

                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.46);
            }
        }

        const sfx = new RetroAudio();

        /**
         * Custom Pixel Art Sprite Matrix Renderer
         * Faithfully modeled after the uploaded Creamy Yellow Muscle Frog reference image!
         * Palette Legend:
         * '.' : Transparent
         * 'O' : Dark warm outline (#463814)
         * 'Y' : Main creamy yellow body (#F8D866)
         * 'H' : Creamy highlight top (#FFF099)
         * 'S' : Warm golden shading (#E2B53E)
         * 'D' : Deeper muscle shadow (#C99926)
         * 'C' : Pale cream belly (#F7E7C0)
         * 'G' : Green eye ring (#65A877)
         * 'K' : Pure black pupil / mouth (#191B18)
         * 'W' : White eye catchlight (#FFFFFF)
         * 'F' : Dark olive-brown hands & feet (#584F37)
         * 'f' : Darker toe tip (#3B3423)
         */
        const FROG_PALETTE = {
            'O': '#463814',
            'Y': '#F8D866',
            'H': '#FFF099',
            'S': '#E2B53E',
            'D': '#C99926',
            'C': '#F7E7C0',
            'G': '#65A877',
            'K': '#191B18',
            'W': '#FFFFFF',
            'F': '#584F37',
            'f': '#3B3423'
        };

        // 1. Squat / Wide Muscular Stance (Matches the exact uploaded image pose!)
        // 24 columns x 18 rows
        const FROG_SPRITE_SQUAT = [
            "........OOOOOOOO........",
            "......OOYYYYYYYYOO......",
            ".....OYYGGGGYYYYGGO.....",
            "....OYHGWKKGYYYYGWKGO...",
            "....OYHGKKKGYYYYGKKGO...",
            "...OYYYOOGGOYYYYOGGOYO..",
            "..OYYYYYYYYOKKKKOYYYYYO.",
            ".OYYYYHYYYYYYYYYYYYHYYYO",
            "OYYYYYYYYSSSSSSSSYYYYYYO",
            "OYYYSOOOCCCCCCCCCCOOOSYY",
            "OYYSO.OOCCCCCCCCCCOO.OSY",
            "OYSO..OSSOCCCCCCOSSo..OS",
            "OYS...OSSOCCCCCCOSSo..SY",
            "OFO...OFFO.OOOO.OFFO..OF",
            "OFFO.fFFFO......OFFFfOFF",
            "fFFf.OOOO........OOOOfFF"
        ];

        // 2. Mid-Air Upward Jump Pose (Stretched muscular arms & legs propelling up)
        const FROG_SPRITE_JUMP = [
            ".......OOOOOOOOOO.......",
            "......OYYHHHHHHYYO......",
            ".....OYYGGGGYYYYGGO.....",
            "....OYHGWKKGYYYYGWKGO...",
            "....OYHGKKKGYYYYGKKGO...",
            "....OYYOOGGOYYYYOGGOYO..",
            "...OYYYYYYYOKKKKOYYYYYO.",
            "..OYYYYYYYYYYYYYYYYYYYYO",
            "..OYYSOCCCCCCCCCCCCOSYYO",
            ".OYYSO.CCCCCCCCCCCC.OSYY",
            ".OYSO..OCCCCCCCCCCO..OSY",
            ".OFO...OSSSSSSSSSSO...OF",
            ".OFFO...OYYOOOOYYO...OFF",
            "..OO....OYYO..OYYO....OO",
            "........OFFO..OFFO......",
            ".......fFFFf..fFFFf....."
        ];

        // 3. Falling / Ready-to-Land Wide Stance
        const FROG_SPRITE_FALL = [
            ".......OOOOOOOOOO.......",
            ".....OOYYHHHHHHYYOO.....",
            "....OYYYGGGGYYYYGGO.....",
            "...OYYHGWKKGYYYYGWKGO...",
            "...OYYHGKKKGYYYYGKKGO...",
            "..OYYYYOOGGOYYYYOGGOYO..",
            ".OYYYYYYYYYOKKKKOYYYYYYO",
            "OYYYYHYYYYYYYYYYYYHYYYYO",
            "OYYSOOOCCCCCCCCCCCCOOOSY",
            "OFFO..OCCCCCCCCCCCCO..OF",
            "fFFf..OSSOCCCCCCOSSo..fF",
            "......OFFO.OOOO.OFFO....",
            ".....fFFFO......OFFFf..."
        ];

        // Mosquito Sprite Palette & Frames (16x12)
        const MOSQUITO_PALETTE = {
            'K': '#1E1A24',
            'B': '#494157',
            'L': '#837799',
            'R': '#FF3B4E',
            'W': '#DDF6FF',
            'w': '#9AD4F5',
            'S': '#E8D9B5'
        };

        // Mosquito Frame 1 (Wings Up, Long Proboscis pointing left)
        const MOSQUITO_FRAME_1 = [
            ".....WWWW..WWWW.",
            "....WwwwwWWwwwwW",
            ".....WWWWWWWWWW.",
            "....KKRRKKKK....",
            "KKKKKKRRBBBBKK..",
            "...KKKKKKKKBSBK.",
            ".....K..K..KBSBK",
            "....K..K....KBK.",
            "...K..K....K..K."
        ];

        // Mosquito Frame 2 (Wings Down)
        const MOSQUITO_FRAME_2 = [
            "................",
            "....KKRRKKKK....",
            "KKKKKKRRBBBBKK..",
            "...KKKKKKKKBSBK.",
            "....WWWWWWWWBSBK",
            "...WwwwwwwwwWKBK",
            "....WWWW..WWK..K",
            "...K..K....K....",
            "..K..K.........."
        ];

        // Squashed Mosquito Sprite (When stomped from above!)
        const MOSQUITO_SQUASHED = [
            "..WWWW....WWWW..",
            "KKKKRRRRBBBBKKKK",
            "..KKKKKKKKBSBKK.",
            ".K...K..K...K..K"
        ];

        /**
         * Helper function to draw a pixel matrix onto any 2D Canvas context
         */
        function drawPixelMatrix(ctx, matrix, palette, x, y, pixelSize, flipX = false) {
            const rows = matrix.length;
            const cols = matrix[0].length;
            ctx.save();
            if (flipX) {
                ctx.translate(x + cols * pixelSize, y);
                ctx.scale(-1, 1);
                x = 0;
                y = 0;
            }
            for (let r = 0; r < rows; r++) {
                const rowStr = matrix[r];
                for (let c = 0; c < cols; c++) {
                    const ch = rowStr[c];
                    if (ch !== '.' && palette[ch]) {
                        ctx.fillStyle = palette[ch];
                        ctx.fillRect(
                            Math.floor(x + c * pixelSize),
                            Math.floor(y + r * pixelSize),
                            Math.ceil(pixelSize),
                            Math.ceil(pixelSize)
                        );
                    }
                }
            }
            ctx.restore();
        }

        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const previewCanvas = document.getElementById('previewCanvas');
        const previewCtx = previewCanvas.getContext('2d');

        // UI Elements
        const scoreDisplay = document.getElementById('scoreDisplay');
        const stompDisplay = document.getElementById('stompDisplay');
        const bestDisplay = document.getElementById('bestDisplay');
        const shieldIndicator = document.getElementById('shieldIndicator');
        const soundToggleBtn = document.getElementById('soundToggleBtn');
        const soundIcon = document.getElementById('soundIcon');
        const milestoneToast = document.getElementById('milestoneToast');

        const startModal = document.getElementById('startModal');
        const gameOverModal = document.getElementById('gameOverModal');
        const startBtn = document.getElementById('startBtn');
        const restartBtn = document.getElementById('restartBtn');
        const finalScoreText = document.getElementById('finalScoreText');
        const finalStompText = document.getElementById('finalStompText');
        const finalBestText = document.getElementById('finalBestText');
        const deathReasonBadge = document.getElementById('deathReasonBadge');

        const btnLeft = document.getElementById('btnLeft');
        const btnRight = document.getElementById('btnRight');

        // Virtual logical resolution for crisp pixel art physics
        const GAME_WIDTH = 380;
        let GAME_HEIGHT = 640;

        // Persistent High Score
        let highScore = 0;
        try {
            highScore = parseInt(localStorage.getItem('creamyFrogBestScore') || '0', 10);
        } catch (e) {
            highScore = 0;
        }
        bestDisplay.textContent = `${highScore}m`;

        // Input State
        const keys = {
            left: false,
            right: false,
            pointerActive: false,
            pointerTargetX: null
        };

        // Game State
        let gameState = 'START'; // 'START', 'PLAYING', 'GAMEOVER'
        let score = 0;
        let bonusScore = 0;
        let stompedCount = 0;
        let cameraY = 0;
        let highestWorldY = 0;
        let lastMilestone = 0;
        let animationFrameId = null;
        let frameCount = 0;

        // Entities
        let frog = null;
        let platforms = [];
        let mosquitoes = [];
        let particles = [];
        let floatingTexts = [];
        let bgClouds = [];
        let bgStars = [];

        /**
         * Resize Canvas while keeping crisp virtual pixel ratio
         */
        function resizeCanvas() {
            const container = document.getElementById('canvasContainer');
            const rect = container.getBoundingClientRect();
            const aspect = rect.height / Math.max(1, rect.width);
            GAME_HEIGHT = Math.max(540, Math.min(780, Math.round(GAME_WIDTH * aspect)));
            canvas.width = GAME_WIDTH;
            canvas.height = GAME_HEIGHT;
            ctx.imageSmoothingEnabled = false;
        }

        window.addEventListener('resize', resizeCanvas);

        /**
         * Render animated Start Screen Preview of Creamy Frog & Mosquito
         */
        function renderPreviewFrog(tick) {
            previewCtx.clearRect(0, 0, previewCanvas.width, previewCanvas.height);

            // Soft radial glow behind frog
            const grad = previewCtx.createRadialGradient(72, 62, 10, 72, 62, 55);
            grad.addColorStop(0, 'rgba(249, 217, 108, 0.28)');
            grad.addColorStop(1, 'rgba(249, 217, 108, 0)');
            previewCtx.fillStyle = grad;
            previewCtx.fillRect(0, 0, 144, 112);

            // Draw lilypad platform under frog
            previewCtx.fillStyle = '#3B7A47';
            previewCtx.fillRect(24, 92, 96, 10);
            previewCtx.fillStyle = '#68B86E';
            previewCtx.fillRect(28, 88, 88, 6);

            // Draw Creamy Frog alternating squat and slight bounce
            const isBounce = Math.sin(tick * 0.08) > 0.55;
            const sprite = isBounce ? FROG_SPRITE_JUMP : FROG_SPRITE_SQUAT;
            const yOffset = isBounce ? 14 : 24;
            drawPixelMatrix(previewCtx, sprite, FROG_PALETTE, 24, yOffset, 4);

            // Draw small buzzing mosquito in top right corner
            const mosqFrame = Math.floor(tick / 4) % 2 === 0 ? MOSQUITO_FRAME_1 : MOSQUITO_FRAME_2;
            const mx = 102 + Math.sin(tick * 0.1) * 5;
            const my = 12 + Math.cos(tick * 0.15) * 4;
            drawPixelMatrix(previewCtx, mosqFrame, MOSQUITO_PALETTE, mx, my, 2, false);
        }

        /**
         * Initialize / Reset Game World
         */
        function initGame() {
            resizeCanvas();
            score = 0;
            bonusScore = 0;
            stompedCount = 0;
            cameraY = 0;
            highestWorldY = 0;
            lastMilestone = 0;
            scoreDisplay.textContent = '0m';
            stompDisplay.textContent = '0';
            shieldIndicator.classList.add('hidden');

            // Create Creamy Frog Protagonist
            frog = {
                x: GAME_WIDTH / 2 - 36,
                y: GAME_HEIGHT - 130,
                width: 72,  // 24 cols * 3px
                height: 48, // 16 rows * 3px
                vx: 0,
                vy: -12.5,
                speed: 6.2,
                gravity: 0.42,
                jumpForce: -12.8,
                facingLeft: false,
                squatTimer: 8,
                superBoostTimer: 0,
                hasShield: false,
                dead: false,
                deathReason: ''
            };

            platforms = [];
            mosquitoes = [];
            particles = [];
            floatingTexts = [];
            bgClouds = [];
            bgStars = [];

            // Generate background pixel clouds & stars
            for (let i = 0; i < 8; i++) {
                bgClouds.push({
                    x: Math.random() * GAME_WIDTH,
                    y: Math.random() * GAME_HEIGHT,
                    w: 48 + Math.floor(Math.random() * 44),
                    h: 16 + Math.floor(Math.random() * 12),
                    speed: 0.15 + Math.random() * 0.25
                });
            }
            for (let i = 0; i < 45; i++) {
                bgStars.push({
                    x: Math.random() * GAME_WIDTH,
                    y: Math.random() * GAME_HEIGHT,
                    size: Math.random() > 0.7 ? 3 : 2,
                    blinkOffset: Math.random() * 100
                });
            }

            // Starting wide lilypad right under the frog
            platforms.push({
                x: GAME_WIDTH / 2 - 55,
                y: GAME_HEIGHT - 60,
                w: 110,
                h: 16,
                type: 'NORMAL',
                vx: 0,
                broken: false,
                hasSpring: false,
                hasShieldItem: false,
                bounceAnim: 0
            });

            // Procedurally populate platforms upwards
            let currentY = GAME_HEIGHT - 135;
            while (currentY > -GAME_HEIGHT) {
                spawnPlatformAt(currentY);
                currentY -= 62 + Math.random() * 26;
            }
        }

        /**
         * Spawn a platform (and potentially a mosquito above it) at world Y
         */
        function spawnPlatformAt(yPos) {
            const altitudeMeters = Math.max(0, Math.floor(-yPos / 12));
            const difficulty = Math.min(1, altitudeMeters / 2500); // Scales 0.0 -> 1.0 up to 2500m

            const w = 68;
            const h = 15;
            const x = 14 + Math.random() * (GAME_WIDTH - w - 28);

            // Choose platform type based on altitude
            const roll = Math.random();
            let type = 'NORMAL'; // Green Lilypad / Moss Stone
            let vx = 0;

            if (roll < 0.16 + difficulty * 0.22) {
                type = 'MOVING'; // Moving Cloud / Branch
                const spd = 1.2 + Math.random() * 1.4 + difficulty * 1.1;
                vx = Math.random() < 0.5 ? spd : -spd;
            } else if (roll < 0.26 + difficulty * 0.28) {
                type = 'CRACKED'; // Cracked Mud Brick (breaks in 1 jump)
            }

            // Spring Mushroom or Bubble Shield Item on non-cracked platforms
            let hasSpring = false;
            let hasShieldItem = false;
            if (type !== 'CRACKED') {
                const itemRoll = Math.random();
                if (itemRoll < 0.11) {
                    hasSpring = true;
                } else if (itemRoll < 0.155 && (!frog || !frog.hasShield)) {
                    hasShieldItem = true;
                }
            }

            platforms.push({
                x,
                y: yPos,
                w,
                h,
                type,
                vx,
                broken: false,
                hasSpring,
                hasShieldItem,
                bounceAnim: 0
            });

            // Spawn extra safety Normal platform occasionally if gap has cracked ones
            if (type === 'CRACKED' && Math.random() < 0.65) {
                const safeX = (x + GAME_WIDTH / 2) % (GAME_WIDTH - w - 24) + 12;
                platforms.push({
                    x: safeX,
                    y: yPos - 10 + Math.random() * 20,
                    w: 66,
                    h: 15,
                    type: 'NORMAL',
                    vx: 0,
                    broken: false,
                    hasSpring: false,
                    hasShieldItem: false,
                    bounceAnim: 0
                });
            }

            // Spawn Mosquito Obstacle (避開蚊)
            // Mosquitoes start appearing after 40m and increase frequency with height
            if (altitudeMeters > 35) {
                const mosquitoChance = Math.min(0.46, 0.14 + difficulty * 0.30);
                // Ensure distance from previous mosquito so paths aren't impossible
                const lastMosq = mosquitoes[mosquitoes.length - 1];
                const farEnough = !lastMosq || Math.abs(lastMosq.y - (yPos - 42)) > 125;

                if (Math.random() < mosquitoChance && farEnough) {
                    const mWidth = 48; // 16 cols * 3px
                    const mHeight = 27; // 9 rows * 3px
                    const mx = 20 + Math.random() * (GAME_WIDTH - mWidth - 40);
                    const baseSpeed = 1.1 + Math.random() * 1.5 + difficulty * 1.6;
                    mosquitoes.push({
                        x: mx,
                        y: yPos - 42,
                        baseY: yPos - 42,
                        w: mWidth,
                        h: mHeight,
                        vx: Math.random() < 0.5 ? baseSpeed : -baseSpeed,
                        bobPhase: Math.random() * Math.PI * 2,
                        bobAmp: 6 + Math.random() * 10,
                        knockedOut: false,
                        squashed: false,
                        vyKnock: 0
                    });
                }
            }
        }

        function createJumpParticles(x, y, color = '#FFF099', count = 8) {
            for (let i = 0; i < count; i++) {
                particles.push({
                    x: x + (Math.random() - 0.5) * 32,
                    y: y + (Math.random() - 0.5) * 8,
                    vx: (Math.random() - 0.5) * 4.5,
                    vy: (Math.random() - 0.2) * 3.5,
                    size: 4 + Math.floor(Math.random() * 4),
                    color,
                    alpha: 1,
                    life: 18 + Math.floor(Math.random() * 12)
                });
            }
        }

        function createFloatingText(x, y, text, color = '#F9D96C') {
            floatingTexts.push({
                x,
                y,
                text,
                color,
                vy: -1.6,
                life: 40,
                maxLife: 40
            });
        }

        function recalculateTotalScore() {
            const climbMeters = Math.max(0, Math.floor(-highestWorldY / 12));
            const total = climbMeters + bonusScore;
            if (total > score) {
                score = total;
                scoreDisplay.textContent = `${score}m`;
                if (score > highScore) {
                    highScore = score;
                    bestDisplay.textContent = `${highScore}m`;
                }

                // Milestone announcements every 500m
                if (score >= lastMilestone + 500) {
                    lastMilestone = Math.floor(score / 500) * 500;
                    showMilestoneToast(`🔥 突破 ${lastMilestone}m！奶蛙衝入更高空域！`);
                }
            }
        }

        function showMilestoneToast(text) {
            milestoneToast.textContent = text;
            milestoneToast.classList.remove('opacity-0', '-translate-y-3');
            milestoneToast.classList.add('opacity-100', 'translate-y-0');
            clearTimeout(milestoneToast._hideTimer);
            milestoneToast._hideTimer = setTimeout(() => {
                milestoneToast.classList.add('opacity-0', '-translate-y-3');
                milestoneToast.classList.remove('opacity-100', 'translate-y-0');
            }, 2200);
        }

        function updateGame() {
            if (gameState !== 'PLAYING') return;

            // 1. Horizontal Movement (Keyboard + Touch Buttons + Pointer Drag)
            if (keys.left && !keys.right) {
                frog.vx = -frog.speed;
                frog.facingLeft = true;
            } else if (keys.right && !keys.left) {
                frog.vx = frog.speed;
                frog.facingLeft = false;
            } else if (keys.pointerActive && keys.pointerTargetX !== null) {
                const frogCenter = frog.x + frog.width / 2;
                const diff = keys.pointerTargetX - frogCenter;
                if (Math.abs(diff) > 6) {
                    frog.vx = Math.max(-frog.speed, Math.min(frog.speed, diff * 0.22));
                    frog.facingLeft = diff < 0;
                } else {
                    frog.vx *= 0.6;
                }
            } else {
                frog.vx *= 0.78;
                if (Math.abs(frog.vx) < 0.1) frog.vx = 0;
            }

            frog.x += frog.vx;

            // Horizontal screen wrap-around
            if (frog.x + frog.width < 0) {
                frog.x = GAME_WIDTH;
            } else if (frog.x > GAME_WIDTH) {
                frog.x = -frog.width;
            }

            // 2. Vertical Physics & Super Boost Trail
            frog.vy += frog.gravity;
            frog.y += frog.vy;

            if (frog.squatTimer > 0) frog.squatTimer--;
            if (frog.superBoostTimer > 0) {
                frog.superBoostTimer--;
                if (frameCount % 2 === 0) {
                    createJumpParticles(
                        frog.x + frog.width / 2,
                        frog.y + frog.height,
                        frameCount % 4 === 0 ? '#F9D96C' : '#7CE595',
                        2
                    );
                }
            }

            // 3. Camera Scroll Upwards when Frog climbs above 44% of screen
            const cameraThreshold = cameraY + GAME_HEIGHT * 0.44;
            if (frog.y < cameraThreshold) {
                const delta = cameraThreshold - frog.y;
                cameraY -= delta;

                // Scroll ambient clouds and stars gently for parallax
                bgClouds.forEach(c => {
                    c.y += delta * 0.35;
                    if (c.y > GAME_HEIGHT + 30) {
                        c.y = -30;
                        c.x = Math.random() * GAME_WIDTH;
                    }
                });
                bgStars.forEach(s => {
                    s.y += delta * 0.15;
                    if (s.y > GAME_HEIGHT + 10) {
                        s.y = -10;
                        s.x = Math.random() * GAME_WIDTH;
                    }
                });
            }

            // Update Score (1 meter per 12 pixels climbed)
            if (frog.y < highestWorldY) {
                highestWorldY = frog.y;
                const currentMeters = Math.max(0, Math.floor(-highestWorldY / 12));
                if (currentMeters > score) {
                    score = currentMeters;
                    scoreDisplay.textContent = `${score}m`;
                    if (score > highScore) {
                        highScore = score;
                        bestDisplay.textContent = `${highScore}m`;
                    }

                    // Milestone announcements every 500m
                    if (score >= lastMilestone + 500) {
            // Update Score (1 meter per 12 pixels climbed + stomp bonuses)
            if (frog.y < highestWorldY) {
                highestWorldY = frog.y;
            }
            recalculateTotalScore();

            // 4. Update Platforms & Check Landing Collisions (Only when falling vy > 0)
            for (let i = platforms.length - 1; i >= 0; i--) {
                const p = platforms[i];

                // Move horizontal platforms
                if (p.type === 'MOVING') {
                    p.x += p.vx;
                    if (p.x < 10 || p.x + p.w > GAME_WIDTH - 10) {
                        p.vx *= -1;
                    }
                }
                if (p.bounceAnim > 0) p.bounceAnim--;

                // Landing detection (Feet hitbox)
                if (!frog.dead && frog.vy > 0 && !p.broken) {
                    const frogLeft = frog.x + 10;
                    const frogRight = frog.x + frog.width - 10;
                    const frogBottom = frog.y + frog.height;
                    const prevFrogBottom = frogBottom - frog.vy;

                    if (
                        frogRight >= p.x &&
                        frogLeft <= p.x + p.w &&
                        prevFrogBottom <= p.y + 10 &&
            // 5. Update Mosquitoes & Check Stomp / Hazard Collisions
            for (let i = mosquitoes.length - 1; i >= 0; i--) {
                const m = mosquitoes[i];

                if (m.knockedOut) {
                    m.y += m.vyKnock;
                    m.vyKnock += 0.55;
                } else {
                    m.x += m.vx;
                    if (m.x < 8 || m.x + m.w > GAME_WIDTH - 8) {
                        m.vx *= -1;
                    }
                    m.bobPhase += 0.09;
                    m.y = m.baseY + Math.sin(m.bobPhase) * m.bobAmp;

                    // Check interaction with Creamy Frog
                    if (!frog.dead) {
                        const frogLeft = frog.x + 8;
                        const frogRight = frog.x + frog.width - 8;
                        const frogBottom = frog.y + frog.height;
                        const prevFrogBottom = frogBottom - frog.vy;

                        const mosqLeft = m.x + 2;
                        const mosqRight = m.x + m.w - 2;
                        const mosqTop = m.y;
                        const mosqStompZoneBottom = m.y + m.h * 0.65;

                        // A. STOMP FROM ABOVE (蚊由上面踏下去是可以被踏死!)
                        // When frog is falling downwards (vy > 0) and its feet land on the upper part of the mosquito
                        const isStompingFromTop =
                            frog.vy > 0 &&
                            frogRight >= mosqLeft &&
                            frogLeft <= mosqRight &&
                            prevFrogBottom <= mosqStompZoneBottom + 8 &&
                            frogBottom >= mosqTop - 4;

                        if (isStompingFromTop) {
                            m.knockedOut = true;
                            m.squashed = true;
                            m.vyKnock = 3.5;

                            // Strong upward trampoline bounce off the mosquito!
                            frog.vy = frog.jumpForce * 1.32;
                            frog.y = m.y - frog.height;
                            frog.squatTimer = 12;

                            // Update Stomp Counter & Bonus Score (+50m)
                            stompedCount++;
                            bonusScore += 50;
                            stompDisplay.textContent = String(stompedCount);
                            recalculateTotalScore();

                            sfx.playStompMosquito();
                            createJumpParticles(m.x + m.w / 2, m.y + 8, '#FF4757', 14);
                            createJumpParticles(m.x + m.w / 2, m.y + 4, '#F9D96C', 10);
                            createFloatingText(m.x + m.w / 2, m.y - 6, '+50m 踏死蚊!', '#F9D96C');
                        } else {
                            // B. Body Collision (When moving upwards or hitting from underneath/side)
                            const fx1 = frog.x + 13;
                            const fy1 = frog.y + 10;
                            const fx2 = frog.x + frog.width - 13;
                            const fy2 = frog.y + frog.height - 6;

                            const mx1 = m.x + 6;
                            const my1 = m.y + 5;
                            const mx2 = m.x + m.w - 6;
                            const my2 = m.y + m.h - 4;

                            if (fx1 < mx2 && fx2 > mx1 && fy1 < my2 && fy2 > my1) {
                                // If frog is in Super Rocket Leap OR has Bubble Shield, knock mosquito away!
                                if (frog.superBoostTimer > 0) {
                                    m.knockedOut = true;
                                    m.squashed = true;
                                    m.vyKnock = -6;
                                    stompedCount++;
                                    bonusScore += 50;
                                    stompDisplay.textContent = String(stompedCount);
                                    recalculateTotalScore();
                                    sfx.playStompMosquito();
                                    createJumpParticles(m.x + m.w / 2, m.y + m.h / 2, '#FF5C77', 14);
                                    createFloatingText(m.x + m.w / 2, m.y - 6, '+50m 衝撞擊破!', '#7CE595');
                                } else if (frog.hasShield) {
                                    frog.hasShield = false;
                                    shieldIndicator.classList.add('hidden');
                                    m.knockedOut = true;
                                    m.vyKnock = -5;
                                    frog.vy = frog.jumpForce * 0.9;
                                    sfx.playShieldBlock();
                                    createJumpParticles(frog.x + frog.width / 2, frog.y + frog.height / 2, '#7ad7ff', 18);
                                    showMilestoneToast('🫧 水泡護盾抵擋了蚊子攻擊！');
                                } else {
                                    // Hit by Mosquito from below/side -> Knocked out!
                                    frog.dead = true;
                                    frog.vy = -7.5;
                                    frog.deathReason = '🦟 被惡蚊叮中了！(由上方踩下可踏死蚊子)';
                                    sfx.playMosquitoHit();
                                    createJumpParticles(frog.x + frog.width / 2, frog.y + frog.height / 2, '#FF4757', 22);
                                }
                            }
                        }
                    }
                }

                if (m.y > cameraY + GAME_HEIGHT + 100) {
                    mosquitoes.splice(i, 1);
                }
            }

            // 6. Update Particles & Floating Popup Texts
            for (let i = particles.length - 1; i >= 0; i--) {
                const pt = particles[i];
                pt.x += pt.vx;
                pt.y += pt.vy;
                pt.life--;
                pt.alpha = Math.max(0, pt.life / 24);
                if (pt.life <= 0) {
                    particles.splice(i, 1);
                }
            }

            for (let i = floatingTexts.length - 1; i >= 0; i--) {
                const ft = floatingTexts[i];
                ft.y += ft.vy;
                ft.life--;
                if (ft.life <= 0) {
                    floatingTexts.splice(i, 1);
                }
            }

            // 7. Check Fall Below Screen -> Game Over
            if (frog.y > cameraY + GAME_HEIGHT + 40) {
                if (!frog.deathReason) {
                    frog.deathReason = '💧 奶蛙跌落池塘了！(Fell Down)';
                    sfx.playMosquitoHit();
                }
                triggerGameOver();
            }
        }

        function interpolateHex(hex1, hex2, factor) {
            const f = Math.max(0, Math.min(1, factor));
            const c1 = parseInt(hex1.slice(1), 16);
            const c2 = parseInt(hex2.slice(1), 16);
            const r = Math.round(((c1 >> 16) & 255) * (1 - f) + ((c2 >> 16) & 255) * f);
            const g = Math.round(((c1 >> 8) & 255) * (1 - f) + ((c2 >> 8) & 255) * f);
            const b = Math.round((c1 & 255) * (1 - f) + (c2 & 255) * f);
            return `rgb(${r},${g},${b})`;
        }

        function drawBackground() {
            // Sky transitions based on height: Pond Day -> Sunset -> Night -> Deep Cosmos
            let topColor = '#4FA8E8';
            let botColor = '#B3E5FC';

            if (score < 600) {
                const t = score / 600;
                topColor = interpolateHex('#4FA8E8', '#E06D53', t);
                botColor = interpolateHex('#B3E5FC', '#F9C784', t);
            } else if (score < 1400) {
                const t = (score - 600) / 800;
                topColor = interpolateHex('#E06D53', '#191D3A', t);
                botColor = interpolateHex('#F9C784', '#394375', t);
            } else {
                const t = Math.min(1, (score - 1400) / 1200);
                topColor = interpolateHex('#191D3A', '#090A14', t);
                botColor = interpolateHex('#394375', '#1B1438', t);
            }

            const skyGrad = ctx.createLinearGradient(0, 0, 0, GAME_HEIGHT);
            skyGrad.addColorStop(0, topColor);
            skyGrad.addColorStop(1, botColor);
            ctx.fillStyle = skyGrad;
            ctx.fillRect(0, 0, GAME_WIDTH, GAME_HEIGHT);

            // Twinkling Pixel Stars when altitude > 450m
            if (score > 450) {
                const starAlpha = Math.min(1, (score - 450) / 500);
                bgStars.forEach(s => {
                    const twinkle = 0.4 + 0.6 * Math.abs(Math.sin((frameCount + s.blinkOffset) * 0.05));
                    ctx.fillStyle = `rgba(255, 250, 220, ${starAlpha * twinkle})`;
                    ctx.fillRect(Math.floor(s.x), Math.floor(s.y), s.size, s.size);
                });
            }

            // Pixel Clouds
            bgClouds.forEach(c => {
                c.x += c.speed;
                if (c.x > GAME_WIDTH + 20) c.x = -c.w - 20;

                ctx.fillStyle = score > 900 ? 'rgba(180, 195, 235, 0.16)' : 'rgba(255, 255, 255, 0.45)';
                const cx = Math.floor(c.x);
                const cy = Math.floor(c.y);
                ctx.fillRect(cx + 8, cy, c.w - 16, c.h);
                ctx.fillRect(cx, cy + 4, c.w, c.h - 6);
            });
        }

        function drawPlatforms() {
            platforms.forEach(p => {
                if (p.broken) return;
                const screenY = Math.floor(p.y - cameraY + (p.bounceAnim > 0 ? 3 : 0));
                const screenX = Math.floor(p.x);

                if (p.type === 'NORMAL') {
                    // Green Pond Lilypad / Moss Stone Platform
                    ctx.fillStyle = '#244A2C';
                    ctx.fillRect(screenX, screenY + 3, p.w, p.h - 2);
                    ctx.fillStyle = '#438A4E';
                    ctx.fillRect(screenX + 2, screenY + 1, p.w - 4, p.h - 4);
                    ctx.fillStyle = '#7CE595';
                    ctx.fillRect(screenX + 4, screenY, p.w - 8, 5);
                    // Little notch in lilypad
                    ctx.fillStyle = '#244A2C';
                    ctx.fillRect(screenX + p.w - 14, screenY, 3, 5);
                } else if (p.type === 'MOVING') {
                    // Sky Winged Cloud Platform
                    ctx.fillStyle = '#3B5C85';
                    ctx.fillRect(screenX, screenY + 3, p.w, p.h - 2);
                    ctx.fillStyle = '#73B9FF';
                    ctx.fillRect(screenX + 2, screenY + 1, p.w - 4, p.h - 4);
                    ctx.fillStyle = '#E3F4FF';
                    ctx.fillRect(screenX + 4, screenY, p.w - 8, 5);
                } else if (p.type === 'CRACKED') {
                    // Cracked Mud / Dry Clay Brick
                    ctx.fillStyle = '#4A2E1B';
                    ctx.fillRect(screenX, screenY + 3, p.w, p.h - 2);
                    ctx.fillStyle = '#9C633B';
                    ctx.fillRect(screenX + 2, screenY + 1, p.w - 4, p.h - 4);
                    ctx.fillStyle = '#C7895C';
                    ctx.fillRect(screenX + 4, screenY, p.w - 8, 4);
                    // Pixel crack lines in center
                    ctx.fillStyle = '#2E1A0D';
                    ctx.fillRect(screenX + p.w / 2 - 2, screenY, 3, 6);
                    ctx.fillRect(screenX + p.w / 2 + 1, screenY + 5, 3, 6);
                }

                // Draw Spring Bouncy Mushroom on platform
                if (p.hasSpring) {
                    const sx = screenX + Math.floor(p.w / 2) - 10;
                    const sy = screenY - 12;
                    // Stem
                    ctx.fillStyle = '#F7E7C0';
                    ctx.fillRect(sx + 6, sy + 6, 8, 6);
                    // Red/Pink Mushroom Cap with white pixel spots
                    ctx.fillStyle = '#E83B68';
                    ctx.fillRect(sx + 2, sy + 2, 16, 5);
                    ctx.fillRect(sx + 4, sy, 12, 3);
                    ctx.fillStyle = '#FFFFFF';
                    ctx.fillRect(sx + 5, sy + 2, 3, 2);
                    ctx.fillRect(sx + 12, sy + 2, 3, 2);
                }

                // Draw Water Bubble Shield Collectible
                if (p.hasShieldItem) {
                    const bx = screenX + Math.floor(p.w / 2) - 9;
                    const by = screenY - 22 + Math.floor(Math.sin(frameCount * 0.1) * 3);
                    ctx.fillStyle = 'rgba(84, 204, 255, 0.35)';
                    ctx.fillRect(bx + 2, by + 2, 14, 14);
                    ctx.strokeStyle = '#99E6FF';
                    ctx.lineWidth = 2;
                    ctx.strokeRect(bx + 2, by + 2, 14, 14);
                    ctx.fillStyle = '#FFFFFF';
                    ctx.fillRect(bx + 5, by + 5, 3, 3);
                }
            });
        }

        function drawMosquitoes() {
            const mosqFrame = Math.floor(frameCount / 3) % 2 === 0 ? MOSQUITO_FRAME_1 : MOSQUITO_FRAME_2;
            mosquitoes.forEach(m => {
                const screenY = Math.floor(m.y - cameraY);
                const screenX = Math.floor(m.x);
                if (screenY < -60 || screenY > GAME_HEIGHT + 60) return;

                const flip = m.vx > 0;

                if (m.squashed) {
                    // Draw flattened mosquito falling down after being stomped!
                    drawPixelMatrix(ctx, MOSQUITO_SQUASHED, MOSQUITO_PALETTE, screenX, screenY + 10, 3, flip);
                    return;
                }

                // Subtle danger warning aura around body
                ctx.fillStyle = 'rgba(255, 59, 78, 0.13)';
                ctx.beginPath();
                ctx.arc(screenX + m.w / 2, screenY + m.h / 2, 22, 0, Math.PI * 2);
                ctx.fill();

                // Top Stompable Target Indicator (Subtle golden bounce line above mosquito head)
                ctx.fillStyle = 'rgba(249, 217, 108, 0.75)';
                ctx.fillRect(screenX + 12, screenY - 4, m.w - 24, 2);

                // Flip sprite depending on movement direction
                drawPixelMatrix(ctx, mosqFrame, MOSQUITO_PALETTE, screenX, screenY, 3, flip);
            });
        }

        function drawFloatingTexts() {
            floatingTexts.forEach(ft => {
                const screenY = Math.floor(ft.y - cameraY);
                const alpha = Math.max(0, ft.life / ft.maxLife);
                ctx.save();
                ctx.globalAlpha = alpha;
                ctx.font = '900 13px "Noto Sans TC", monospace';
                ctx.textAlign = 'center';
                ctx.fillStyle = '#191B18';
                ctx.fillText(ft.text, Math.floor(ft.x) + 1, screenY + 2);
                ctx.fillStyle = ft.color;
                ctx.fillText(ft.text, Math.floor(ft.x), screenY);
                ctx.restore();
            });
        }

        function drawCreamyFrog() {
            if (!frog) return;
            const screenX = Math.floor(frog.x);
            const screenY = Math.floor(frog.y - cameraY);

            // Choose pixel animation frame based on vertical state
            let sprite = FROG_SPRITE_FALL;
            if (frog.squatTimer > 0) {
                sprite = FROG_SPRITE_SQUAT;
            } else if (frog.vy < -1.5) {
                sprite = FROG_SPRITE_JUMP;
            }

            // Super Rocket Boost Golden Aura
            if (frog.superBoostTimer > 0) {
                ctx.fillStyle = 'rgba(249, 217, 108, 0.35)';
                ctx.beginPath();
                ctx.arc(screenX + frog.width / 2, screenY + frog.height / 2, 40, 0, Math.PI * 2);
                ctx.fill();
            }

            // Draw the Creamy Frog (scale 3px per matrix cell)
            drawPixelMatrix(ctx, sprite, FROG_PALETTE, screenX, screenY, 3, frog.facingLeft);

            // Draw Water Bubble Shield around Frog if active
            if (frog.hasShield) {
                const cx = screenX + frog.width / 2;
                const cy = screenY + frog.height / 2 + 2;
                ctx.save();
                ctx.strokeStyle = '#7AD7FF';
                ctx.lineWidth = 3;
                ctx.fillStyle = 'rgba(122, 215, 255, 0.18)';
                ctx.beginPath();
                ctx.arc(cx, cy, 40, 0, Math.PI * 2);
                ctx.fill();
                ctx.stroke();

                // Pixel highlight on bubble
                ctx.fillStyle = '#FFFFFF';
                ctx.fillRect(cx - 22, cy - 24, 8, 4);
                ctx.fillRect(cx - 24, cy - 20, 4, 6);
                ctx.restore();
            }
        }

        function drawParticles() {
            particles.forEach(pt => {
                ctx.save();
                ctx.globalAlpha = pt.alpha;
                ctx.fillStyle = pt.color;
                ctx.fillRect(
                    Math.floor(pt.x),
                    Math.floor(pt.y - cameraY),
                    pt.size,
                    pt.size
                );
                ctx.restore();
            });
        }

        function renderGame() {
            drawBackground();
            drawPlatforms();
            drawParticles();
            drawMosquitoes();
            drawCreamyFrog();
            drawFloatingTexts();
        }

        function gameLoop() {
            frameCount++;

            if (gameState === 'START') {
                renderPreviewFrog(frameCount);
            } else if (gameState === 'PLAYING') {
                updateGame();
                renderGame();
            }

            animationFrameId = requestAnimationFrame(gameLoop);
        }

        function startGame() {
            sfx.init();
            sfx.playJump();
            startModal.classList.add('hidden');
            gameOverModal.classList.add('hidden');
            initGame();
            gameState = 'PLAYING';
        }

        function triggerGameOver() {
            gameState = 'GAMEOVER';
            try {
                localStorage.setItem('creamyFrogBestScore', String(highScore));
            } catch (e) {}

            deathReasonBadge.textContent = frog.deathReason || '🦟 跳躍結束！';
            finalScoreText.textContent = `${score}m`;
            finalStompText.textContent = String(stompedCount);
            finalBestText.textContent = `${highScore}m`;
            gameOverModal.classList.remove('hidden');
        }

        // Keyboard Listeners
        window.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') {
                keys.left = true;
            }
            if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') {
                keys.right = true;
            }
            if ((e.key === ' ' || e.key === 'Enter') && gameState !== 'PLAYING') {
                e.preventDefault();
                startGame();
            }
        });

        window.addEventListener('keyup', (e) => {
            if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') {
                keys.left = false;
            }
            if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') {
                keys.right = false;
            }
        });

        // On-Screen Touch Direction Buttons
        function bindDirectionButton(btn, dirKey) {
            const activate = (e) => {
                e.preventDefault();
                e.stopPropagation();
                keys[dirKey] = true;
                btn.classList.add('active');
            };
            const deactivate = (e) => {
                e.preventDefault();
                e.stopPropagation();
                keys[dirKey] = false;
                btn.classList.remove('active');
            };
            btn.addEventListener('mousedown', activate);
            btn.addEventListener('mouseup', deactivate);
            btn.addEventListener('mouseleave', deactivate);
            btn.addEventListener('touchstart', activate, { passive: false });
            btn.addEventListener('touchend', deactivate, { passive: false });
            btn.addEventListener('touchcancel', deactivate, { passive: false });
        }

        bindDirectionButton(btnLeft, 'left');
        bindDirectionButton(btnRight, 'right');

        // Direct Canvas Pointer / Touch Dragging Support
        function updatePointerX(clientX) {
            const rect = canvas.getBoundingClientRect();
            const relX = ((clientX - rect.left) / rect.width) * GAME_WIDTH;
            keys.pointerTargetX = Math.max(0, Math.min(GAME_WIDTH, relX));
        }

        canvas.addEventListener('mousedown', (e) => {
            if (gameState !== 'PLAYING') return;
            keys.pointerActive = true;
            updatePointerX(e.clientX);
        });
        window.addEventListener('mousemove', (e) => {
            if (!keys.pointerActive) return;
            updatePointerX(e.clientX);
        });
        window.addEventListener('mouseup', () => {
            keys.pointerActive = false;
            keys.pointerTargetX = null;
        });

        canvas.addEventListener('touchstart', (e) => {
            if (gameState !== 'PLAYING') return;
            if (e.touches.length > 0) {
                keys.pointerActive = true;
                updatePointerX(e.touches[0].clientX);
            }
        }, { passive: true });

        canvas.addEventListener('touchmove', (e) => {
            if (!keys.pointerActive || e.touches.length === 0) return;
            updatePointerX(e.touches[0].clientX);
        }, { passive: true });

        canvas.addEventListener('touchend', () => {
            keys.pointerActive = false;
            keys.pointerTargetX = null;
        });

        // Sound Toggle & Start/Restart Buttons
        soundToggleBtn.addEventListener('click', () => {
            const on = sfx.toggle();
            soundIcon.textContent = on ? '🔊' : '🔇';
        });

        startBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', startGame);

        // Initialize on Window Load
        window.onload = function () {
            initGame();
            renderGame();
            gameLoop();
        };
    </script>
</body>
</html>
