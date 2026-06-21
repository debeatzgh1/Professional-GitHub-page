<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gist Portal Launcher</title>
    <style>
        :root {
            --notify-red: #ff3e3e;
            --gist-blue: #58a6ff;
            --glass-dark: rgba(13, 17, 23, 0.95);
        }

        /* 1. FLOATING WRAPPER */
        #notification-wrapper {
            position: fixed;
            bottom: 30px;
            right: 30px;
            display: flex;
            align-items: center;
            z-index: 9999;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        /* Shake Animation */
        @keyframes bell-shake {
            0%, 100% { transform: rotate(0); }
            15% { transform: rotate(15deg); }
            30% { transform: rotate(-15deg); }
            45% { transform: rotate(10deg); }
            60% { transform: rotate(-10deg); }
        }
        .shake { animation: bell-shake 0.6s ease-in-out; }

        #gist-notifier {
            width: 40px;
            height: 40px;
            background: var(--glass-dark);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            box-shadow: 0 10px 40px rgba(0,0,0,0.6);
            order: 2;
            position: relative;
        }

        /* 2. TYPEWRITER BUBBLE */
        #notify-bubble {
            background: white;
            color: #0d1117;
            padding: 12px 20px;
            border-radius: 14px;
            margin-right: 15px;
            font-size: 13px;
            font-weight: 800;
            opacity: 0;
            transform: translateX(20px);
            transition: opacity 0.4s ease;
            white-space: nowrap;
            order: 1;
            box-shadow: 0 5px 15px rgba(0,0,0,0.3);
            min-width: 180px;
        }
        #notify-bubble.show { opacity: 1; transform: translateX(0); }
        
        /* Cursor effect */
        #type-text::after {
            content: "|";
            animation: blink 0.7s infinite;
            margin-left: 2px;
            color: var(--gist-blue);
        }
        @keyframes blink { 50% { opacity: 0; } }

        /* 3. PERMANENT LAUNCHER */
        #hub-launcher {
            position: fixed;
            bottom: 25px;
            right: 25px;
            background: var(--glass-dark);
            color: #fff;
            padding: 12px 22px;
            border-radius: 50px;
            font-size: 10px;
            font-weight: 700;
            letter-spacing: 1.5px;
            text-transform: uppercase;
            border: 1px solid rgba(255,255,255,0.1);
            cursor: pointer;
            display: none; 
            z-index: 9998;
            transition: all 0.3s ease;
            box-shadow: 0 5px 15px rgba(0,0,0,0.4);
        }
        #hub-launcher:hover { border-color: var(--gist-blue); color: var(--gist-blue); transform: translateY(-2px); }

        #comment-badge {
            position: absolute;
            top: -2px; right: -2px;
            background: var(--notify-red);
            padding: 4px 7px;
            border-radius: 8px;
            font-size: 9px;
            color: #fff;
            display: none;
            font-weight: 900;
        }

        /* MODAL OVERLAY */
        #gist-overlay {
            position: fixed;
            inset: 0;
            background: rgba(0,0,0,0.92);
            backdrop-filter: blur(8px);
            display: none;
            z-index: 10000;
            justify-content: center;
            align-items: center;
        }
        .modal-container { width: 95%; max-width: 900px; height: 85vh; background: #0d1117; border-radius: 20px; overflow: hidden; display: flex; flex-direction: column; border: 1px solid #30363d;}
        iframe { width: 100%; flex-grow: 1; border: none; }
    </style>
</head>
<body>

    <audio id="notify-sound" src="https://debeatzgh1.github.io/posts//"></audio>

    <div id="notification-wrapper">
        <div id="notify-bubble">
            <span id="type-text"></span>
        </div>
        <div id="gist-notifier" onclick="handleEngagement()">
            <span id="comment-badge">UPDATE</span>
            <svg width="26" height="26" viewbox="0 0 24 24" fill="none" stroke="#58a6ff" stroke-width="2.5"><path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"></path><path d="M13.73 21a2 2 0 0 1-3.46 0"></path></svg>
        </div>
    </div>

    <button id="hub-launcher" onclick="restoreFloatingButton()">
        FAQS
    </button>

    <div id="gist-overlay">
        <div class="modal-container">
            <div class="modal-header" style="padding:15px 25px; color:#8b949e; display:flex; justify-content:space-between; background:#161b22; font-size: 11px; font-weight: bold; border-bottom: 1px solid #30363d;">
                <span><i class="fab fa-github"></i> GIST_DISCUSSION_PORTAL & FAQS</span>
                <button onclick="closeGist()" style="color:white; background:none; border:none; cursor:pointer; font-size:22px;">&times;</button>
            </div>
            <iframe id="gist-frame"></iframe>
        </div>
    </div>

    <script>
        const wrapper = document.getElementById('notification-wrapper');
        const bubble = document.getElementById('notify-bubble');
        const badge = document.getElementById('comment-badge');
        const launcher = document.getElementById('hub-launcher');
        const notifier = document.getElementById('gist-notifier');
        const sound = document.getElementById('notify-sound');
        const typeTarget = document.getElementById('type-text');

        const message = "System: New Gist comment detected...";

        function typeWriter(text, i = 0) {
            if (i < text.length) {
                typeTarget.innerHTML += text.charAt(i);
                setTimeout(() => typeWriter(text, i + 1), 50);
            }
        }

        function checkNotification() {
            const hasSeen = localStorage.getItem('gist_final_engaged');
            if (!hasSeen) {
                // Initial delay before alert
                setTimeout(() => {
                    bubble.classList.add('show');
                    badge.style.display = 'block';
                    notifier.classList.add('shake');
                    typeWriter(message);
                    
                    // Unlock sound on first interaction
                    document.addEventListener('mouseover', () => { sound.play().catch(()=>{}) }, {once: true});

                    // Auto-hide bubble after 7s
                    setTimeout(() => { 
                        bubble.classList.remove('show');
                        notifier.classList.remove('shake');
                    }, 7000);
                }, 2500);
            } else {
                wrapper.style.display = 'none';
                launcher.style.display = 'block';
            }
        }

        function handleEngagement() {
            localStorage.setItem('gist_final_engaged', 'true');
            document.getElementById('gist-frame').src = "https://mailchi.mp/dda1a453c7bd/ai-decoder";
            document.getElementById('gist-overlay').style.display = 'flex';
            
            // Vanish animation
            wrapper.style.opacity = '0';
            wrapper.style.transform = 'translateY(20px)';
            setTimeout(() => {
                wrapper.style.display = 'none';
                launcher.style.display = 'block'; 
            }, 500);
        }

        function restoreFloatingButton() {
            // FIXED: Removed the accidental copy-paste HTML block and targeted the correct portfolio path
            document.getElementById('gist-frame').src = "https://debeatzgh1.github.io/Html-code-for-portfolio-/";
            document.getElementById('gist-overlay').style.display = 'flex';
        }

        function closeGist() {
            document.getElementById('gist-overlay').style.display = 'none';
            document.getElementById('gist-frame').src = "";
        }

        window.onload = checkNotification;
    </script>
</body>
</html>
