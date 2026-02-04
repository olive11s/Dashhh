# Dashhh
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Seattle Dash - Giselle ❤️</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; touch-action: none; }
        body, html { width: 100%; height: 100%; overflow: hidden; background: #192a56; font-family: -apple-system, sans-serif; position: fixed; }

        #game-container { position: relative; width: 100vw; height: 100vh; background: #2f3640; overflow: hidden; }

        /* Moving Road Lines - Always behind the player */
        .road-layer { position: absolute; inset: 0; z-index: 1; }
        .lane-line { 
            position: absolute; top: -100%; bottom: 0; width: 4px; 
            background: repeating-linear-gradient(to bottom, transparent, transparent 40px, rgba(255,255,255,0.2) 40px, rgba(255,255,255,0.2) 80px);
            animation: moveRoad 0.8s linear infinite;
        }
        @keyframes moveRoad { from { transform: translateY(0); } to { transform: translateY(80px); } }

        /* The Character - 3rd Person View (Always Visible) */
        #character {
            position: absolute; bottom: 22%; width: 70px; height: 70px;
            font-size: 55px; display: flex; justify-content: center; align-items: center;
            transition: left 0.15s cubic-bezier(0.25, 0.46, 0.45, 0.94);
            z-index: 100; transform: translateX(-50%);
        }
        .anim-run { animation: runnerBounce 0.3s infinite alternate ease-in-out; }
        @keyframes runnerBounce { from { margin-bottom: 0; } to { margin-bottom: 15px; } }

        /* HUD - Positioned for Notch and URL Bar Safety */
        #hud {
            position: absolute; top: 12%; width: 100%;
            display: flex; flex-direction: column; align-items: center;
            z-index: 200; color: white; font-weight: bold; text-shadow: 1px 1px 4px #000;
        }
        .progress-box { width: 75%; height: 12px; background: rgba(0,0,0,0.5); border-radius: 6px; margin-top: 10px; overflow: hidden; border: 1px solid rgba(255,255,255,0.5); }
        #fill-bar { width: 0%; height: 100%; background: #00d2d3; transition: width 0.3s; }

        /* Game Items */
        .game-item { position: absolute; font-size: 45px; transform: translateX(-50%); z-index: 80; }
        .heart-sparkle { 
            position: absolute; font-size: 24px; pointer-events: none; z-index: 70;
            animation: heartFade 0.7s ease-out forwards; transform: translateX(-50%);
        }
        @keyframes heartFade { 0% { opacity: 1; top: 0; } 100% { opacity: 0; top: -100px; } }

        /* Overlays */
        .overlay-screen {
            position: fixed; inset: 0; background: rgba(0,0,0,0.95);
            display: flex; flex-direction: column; justify-content: center;
            align-items: center; text-align: center; z-index: 1000; padding: 40px; color: white;
        }
        .go-btn { padding: 20px 50px; border-radius: 50px; background: #00d2d3; color: white; border: none; font-size: 1.3rem; font-weight: bold; margin-top: 30px; }
    </style>
</head>
<body>

<div id="game-container">
    <div class="road-layer">
        <div class="lane-line" style="left: 33.3%;"></div>
        <div class="lane-line" style="left: 66.6%;"></div>
    </div>
    
    <div id="character" class="anim-run">🏃‍♀️</div>

    <div id="hud">
        <div>Tickets Found: <span id="score-val">0</span>/15</div>
        <div class="progress-box"><div id="fill-bar"></div></div>
    </div>
</div>

<div id="start-screen" class="overlay-screen">
    <h1 style="color: #00d2d3; font-size: 2.5rem;">Seattle Dash! 🏔️</h1>
    <p style="margin-top: 15px; font-size: 1.1rem; line-height: 1.4;">Swipe to dodge the study stress and get to Seattle!</p>
    <button class="go-btn" onclick="initGame()">START RUNNING 🎶</button>
</div>

<div id="win-screen" class="overlay-screen" style="display:none; background: white; color: #333;">
    <h1 style="color: #00d2d3;">Vacation Time! ✈️</h1>
    <p style="margin: 20px 0;">You made it to Seattle! Check-in at citizenM is ready.</p>
    <p style="font-size: 1.8rem; font-weight: bold;">I love you, Giselle! ❤️</p>
</div>

<audio id="bg-music" loop preload="auto">
    <source src="https://www.bensound.com/bensound-music/bensound-sunny.mp3" type="audio/mpeg">
</audio>

<script>
    let active = false, tickets = 0, lane = 1, superMode = false;
    const laneX = [16.6, 50, 83.3];
    const hero = document.getElementById('character');
    const scoreText = document.getElementById('score-val');
    const bar = document.getElementById('fill-bar');
    const music = document.getElementById('bg-music');

    function initGame() {
        document.getElementById('start-screen').style.display = 'none';
        active = true;
        music.play().catch(() => {});
        mainLoop();
    }

    // High-Precision Swipe for Mobile
    let startX = 0;
    document.addEventListener('touchstart', (e) => startX = e.touches[0].clientX, {passive: true});
    document.addEventListener('touchend', (e) => {
        if (!active) return;
        const diff = e.changedTouches[0].clientX - startX;
        if (Math.abs(diff) > 30) {
            if (diff > 0 && lane < 2) lane++;
            else if (diff < 0 && lane > 0) lane--;
            hero.style.left = laneX[lane] + '%';
        }
    });

    function mainLoop() {
        if (!active) return;
        spawnObject();
        let speed = Math.max(600, 1000 - (tickets * 30));
        setTimeout(mainLoop, speed);
    }

    function spawnObject() {
        const item = document.createElement('div');
        item.className = 'game-item';
        const rng = Math.random();
        
        // k=key (power), t=ticket, b=book
        item.dataset.type = (rng > 0.93) ? 'k' : (rng > 0.55 ? 't' : 'b');
        item.innerHTML = (item.dataset.type === 'k') ? '🔑' : (item.dataset.type === 't' ? '✈️' : '📚');
        item.style.left = laneX[Math.floor(Math.random() * 3)] + '%';
        item.style.top = '-60px';
        document.getElementById('game-container').appendChild(item);

        let y = -60;
        const speed = superMode ? 14 : (8 + (tickets * 0.2));

        function animate() {
            if (!active) return;
            y += speed;
            item.style.top = y + 'px';

            const pRect = hero.getBoundingClientRect();
            const iRect = item.getBoundingClientRect();

            if (pRect.left < iRect.right - 10 && pRect.right > iRect.left + 10 && 
                pRect.top < iRect.bottom - 10 && pRect.bottom > iRect.top + 10) {
                
                if (item.dataset.type === 'b' && !superMode) {
                    active = false;
                    alert("The study stress was too much! Try again.");
                    location.reload();
                    return;
                } else if (item.dataset.type === 't') {
                    tickets++;
                    scoreText.innerText = tickets;
                    bar.style.width = (tickets / 15 * 100) + '%';
                    if (tickets >= 15) {
                        active = false;
                        document.getElementById('win-screen').style.display = 'flex';
                    }
                } else if (item.dataset.type === 'k') {
                    activatePower();
                }
                item.remove();
                return;
            }

            if (y > window.innerHeight) {
                item.remove();
            } else {
                requestAnimationFrame(animate);
            }
        }
        requestAnimationFrame(animate);
    }

    function activatePower() {
        if (superMode) return;
        superMode = true;
        hero.innerHTML = '✈️';
        hero.classList.remove('anim-run');
        
        const hearts = setInterval(() => {
            if (!superMode) { clearInterval(hearts); return; }
            const h = document.createElement('div');
            h.className = 'heart-sparkle';
            h.innerHTML = '❤️';
            h.style.left = hero.style.left;
            h.style.top = (hero.offsetTop + 40) + 'px';
            document.getElementById('game-container').appendChild(h);
            setTimeout(() => h.remove(), 700);
        }, 120);

        setTimeout(() => {
            superMode = false;
            hero.innerHTML = '🏃‍♀️';
            hero.classList.add('anim-run');
        }, 4500);
    }

    hero.style.left = '50%';
</script>
</body>
</html>
