<!DOCTYPE html>
<html lang="pt-BR">

<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>Jogo do Milhão Cerrado — Carlos, Arthur, Wendel, Marcos, Guilherme</title>
    <style>
        :root {
            --bg: #03041a;
            --panel: #071032;
            --accent: #ffd43b;
            --correct: #0bb14a;
            --wrong: #e53935;
            --option: #093d86;
            --option-hover: #1252b3;
        }

        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            font-family: Inter, Arial, sans-serif;
            color: #fff;
            background: radial-gradient(circle at 10% 10%, #00224a 0%, #04021a 40%, #050018 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 18px;
        }

        .wrap {
            width: 100%;
            max-width: 1200px;
            border-radius: 14px;
            padding: 14px;
            display: grid;
            grid-template-columns: 1fr 340px;
            gap: 18px;
            background: linear-gradient(180deg, rgba(255, 255, 255, 0.03), rgba(0, 0, 0, 0.05));
            box-shadow: 0 20px 60px rgba(0, 0, 0, 0.6);
        }

        header {
            grid-column: 1/3;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        h1 {
            margin: 0;
            color: var(--accent);
            font-size: 1.2rem;
        }

        .subtitle {
            font-size: .9rem;
            opacity: .85;
        }

        main {
            background: var(--panel);
            padding: 16px;
            border-radius: 10px;
        }

        #roundInfo {
            display: flex;
            gap: 8px;
            align-items: center;
            margin-bottom: 8px;
            flex-wrap: wrap;
        }

        #question {
            font-size: 1.2rem;
            margin-bottom: 12px;
        }

        .options {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 10px;
        }

        .option {
            padding: 12px;
            border-radius: 8px;
            background: var(--option);
            border: 1px solid rgba(255, 255, 255, 0.05);
            cursor: pointer;
            font-weight: 700;
            transition: 0.2s;
        }

        .option:hover:not(.disabled) {
            background: var(--option-hover);
            transform: scale(1.02);
        }

        .option.correct {
            background: var(--correct);
        }

        .option.wrong {
            background: var(--wrong);
        }

        .option.disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }

        .side {
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .teams {
            background: var(--panel);
            padding: 12px;
            border-radius: 10px;
            border: 1px solid rgba(255, 255, 255, 0.05);
        }

        .team {
            display: flex;
            justify-content: space-between;
            margin-bottom: 6px;
            padding: 6px;
            border-radius: 6px;
            background: rgba(255, 255, 255, 0.03);
        }

        .team.active {
            background: rgba(255, 255, 255, 0.1);
            color: var(--accent);
        }

        .team.eliminated {
            opacity: 0.4;
            text-decoration: line-through;
        }

        .helps {
            display: flex;
            gap: 6px;
            margin-top: 12px;
            flex-wrap: wrap;
        }

        .helps button {
            flex: 1;
            padding: 8px;
            border: none;
            border-radius: 6px;
            background: #2a3f6b;
            color: #fff;
            cursor: pointer;
            font-weight: 600;
            transition: 0.2s;
        }

        .helps button:hover:not(:disabled) {
            background: #365b9a;
        }

        .helps button:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }

        footer {
            grid-column: 1/3;
            text-align: center;
            font-size: .8rem;
            opacity: .8;
            margin-top: 10px;
        }
    </style>
</head>

<body>

    <!-- 🕒 Cronômetro -->
    <div id="timer" style="position: fixed; top: 12px; right: 20px; font-size: 2.4rem; font-weight: 900; color: #ffd43b; text-shadow: 0 0 12px #ffde78; font-family: 'Trebuchet MS', sans-serif; z-index: 9999;">35</div>

    <div class="wrap">
        <header>
            <div>
                <h1>Jogo do Milhão Cerrado</h1>
                <div class="subtitle">Criado por Carlos, Arthur, Wendel, Marcos, Guilherme</div>
            </div>
            <div class="subtitle">
                Rodada: <span id="roundNumber">1</span> |
                Pergunta: <span id="questionNumber">1</span>
            </div>
        </header>

        <main>
            <div id="roundInfo">
                <b>Dupla atual:</b> <span id="currentTeam">—</span>
                <div style="flex:1"></div>
                <span>Acertos: <b id="teamCorrects">0</b></span>
                <span>Vidas: <b id="teamLives">0</b></span>
            </div>

            <div id="question">Carregando pergunta...</div>
            <div class="options" id="options"></div>
            <div class="helps">
                <button id="btnUni">🎓 Universitários</button>
                <button id="btn5050">🌓 50/50</button>
                <button id="btnSkip">⏭️ Passar</button>
            </div>
            <div id="message" style="margin-top:10px;min-height:25px;font-weight:700;"></div>
        </main>

        <aside class="side">
            <div class="teams">
                <h3>Duplas</h3>
                <div id="teamList"></div>
            </div>
        </aside>

        <footer>Regras: 3 acertos = +1 vida | 0 vidas = eliminação | Uma pergunta por vez para cada dupla.</footer>
    </div>

    <script>
        // 🎧 CORRIGIDO: cria contexto de áudio
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        function tone(freq, duration, delay = 0, type = 'sine', volume = 0.2) {
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.type = type;
            osc.frequency.value = freq;
            gain.gain.value = volume;
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            osc.start(audioCtx.currentTime + delay);
            osc.stop(audioCtx.currentTime + delay + duration);
        }

        // 🔊 Sistema de som
        const audio = new Audio("assets/opening.mp3");
        audio.loop = true;
        audio.volume = 0.6;
        audio.play().catch(() => {
            console.warn("A música será iniciada após interação do usuário.");
        });

        // 🔥 Banco de perguntas (mantido)
        const allQuestions = [
            { q: 'Qual é o segundo maior bioma do Brasil em extensão territorial?', a: ['Cerrado', 'Amazônia', 'Caatinga', 'Mata Atlântica'], c: 0 },
            { q: 'O Cerrado é considerado o berço das águas do Brasil por abrigar nascentes de quais bacias?', a: ['Amazônica e Platina', 'São Francisco e Paraná', 'Paraná, Tocantins e São Francisco', 'Tocantins e Parnaíba'], c: 2 },
            // ... (restante das perguntas que você já tinha)
        ];

        // ⚙️ Variáveis principais
        let teams = [
            { name: 'Dupla 1', lives: 2, corrects: 0, eliminated: false },
            { name: 'Dupla 2', lives: 2, corrects: 0, eliminated: false },
            { name: 'Dupla 3', lives: 2, corrects: 0, eliminated: false },
            { name: 'Dupla 4', lives: 2, corrects: 0, eliminated: false }
        ];
        let currentTeam = 0, currentQuestion = 0, round = 1;
        let usedHelps = { uni: false, fifty: false, skip: false };

        const qEl = document.getElementById('question');
        const oEl = document.getElementById('options');
        const msgEl = document.getElementById('message');

        function updateUI() {
            document.getElementById('roundNumber').textContent = round;
            document.getElementById('questionNumber').textContent = currentQuestion + 1;
            document.getElementById('currentTeam').textContent = teams[currentTeam].name;
            document.getElementById('teamCorrects').textContent = teams[currentTeam].corrects;
            document.getElementById('teamLives').textContent = teams[currentTeam].lives;
            renderTeams();
        }

        function renderTeams() {
            const list = document.getElementById('teamList');
            list.innerHTML = '';
            teams.forEach((t, i) => {
                const div = document.createElement('div');
                div.className = 'team';
                if (i === currentTeam) div.classList.add('active');
                if (t.eliminated) div.classList.add('eliminated');
                div.innerHTML = `${t.name}<span>❤${t.lives} | ✅${t.corrects}</span>`;
                list.appendChild(div);
            });
        }

        function loadQuestion() {
            clearInterval(window.timerInterval);
            let timeLeft = 35;
            const timerDisplay = document.getElementById('timer');
            timerDisplay.textContent = timeLeft;
            timerDisplay.style.color = '#ffd43b';
            timerDisplay.style.textShadow = '0 0 12px #ffde78';

            window.timerInterval = setInterval(() => {
                timeLeft--;
                timerDisplay.textContent = timeLeft;
                if (timeLeft === 10) timerDisplay.style.color = '#ffcc00';
                if (timeLeft === 5) timerDisplay.style.color = '#ff0000';
                if (timeLeft <= 0) {
                    clearInterval(window.timerInterval);
                    msgEl.textContent = '⏰ Tempo esgotado! Dupla eliminada!';
                    eliminateCurrentTeam();
                }
            }, 1000);

            const q = allQuestions[currentQuestion % allQuestions.length];
            qEl.textContent = q.q;
            oEl.innerHTML = '';
            q.a.forEach((opt, i) => {
                const b = document.createElement('button');
                b.className = 'option';
                b.textContent = opt;
                b.onclick = () => chooseAnswer(i);
                oEl.appendChild(b);
            });
            msgEl.textContent = '';
            usedHelps = { uni: false, fifty: false, skip: false };
            updateUI();
        }

        function chooseAnswer(i) {
            const q = allQuestions[currentQuestion % allQuestions.length];
            const opts = document.querySelectorAll('.option');
            opts.forEach(b => b.classList.add('disabled'));
            if (i === q.c) {
                opts[i].classList.add('correct');
                msgEl.textContent = '✅ Resposta correta!';
                const t = teams[currentTeam];
                t.corrects++;
                if (t.corrects % 3 === 0) t.lives++;
            } else {
                opts[i].classList.add('wrong');
                opts[q.c].classList.add('correct');
                teams[currentTeam].lives--;
                if (teams[currentTeam].lives <= 0) {
                    teams[currentTeam].eliminated = true;
                    msgEl.textContent = '❌ Errou! 💀 Eliminado!';
                } else msgEl.textContent = '❌ Resposta errada!';
            }
            setTimeout(nextTurn, 2500);
        }

        function eliminateCurrentTeam() {
            teams[currentTeam].eliminated = true;
            nextTurn();
        }

        function nextTurn() {
            do {
                currentTeam = (currentTeam + 1) % teams.length;
                if (currentTeam === 0) round++;
            } while (teams[currentTeam].eliminated);
            currentQuestion++;
            loadQuestion();
        }

        // 🔥 Abertura do jogo
        function playOpening() {
            const overlay = document.createElement('div');
            Object.assign(overlay.style, {
                position: 'fixed',
                inset: 0,
                display: 'flex',
                flexDirection: 'column',
                alignItems: 'center',
                justifyContent: 'center',
                background: 'linear-gradient(180deg, rgba(3,4,26,0.95), rgba(3,4,26,0.98))',
                color: '#fff',
                zIndex: 9999,
                textAlign: 'center',
                padding: '24px'
            });

            overlay.innerHTML = `
                <h1 style="font-size:3rem;color:#ffd43b;text-shadow:0 0 15px #ffd43b;">💰 Jogo do Milhão Cerrado 💰</h1>
                <p style="font-size:1.1rem;opacity:0.9;">Prepare-se! O show vai começar!</p>
                <button id="startGame" style="margin-top:20px;background:#0bb14a;border:none;padding:12px 26px;border-radius:10px;font-weight:800;cursor:pointer;">Começar ▶</button>
            `;

            document.body.appendChild(overlay);
            document.getElementById('startGame').onclick = () => {
                overlay.style.transition = 'opacity 0.6s';
                overlay.style.opacity = '0';
                setTimeout(() => {
                    overlay.remove();
                    loadQuestion();
                }, 600);
            };
        }

        playOpening();
    </script>
</body>
</html>
