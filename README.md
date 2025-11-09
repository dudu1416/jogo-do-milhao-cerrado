# jogo-do-milhao-cerrado
<!DOCTYPE html>
<html lang="pt-BR">


<head>
    <title>Jogo do Milhão Cerrado</title>
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
            /* Azul mais claro */
            --option-hover: #1252b3;
        }

        * {
            box-sizing: border-box
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

        .teams,
        .prizes {
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
            <div class="prizes">
                <h3>Ranking</h3>
                <div id="prizeList"></div>
            </div>
        </aside>

        <footer>Regras: 3 acertos = +1 vida | 0 vidas = eliminação | Uma pergunta por vez para cada dupla.</footer>
    </div>

    <script>
        // 🔊 Sistema de som
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        function ensureAudio() { if (audioCtx.state === 'suspended') audioCtx.resume(); }
        function tone(freq, dur = 0.12, when = 0, type = 'sine', gain = 0.25) {
            const now = audioCtx.currentTime + when;
            const o = audioCtx.createOscillator(), g = audioCtx.createGain();
            o.type = type; o.frequency.value = freq; o.connect(g); g.connect(audioCtx.destination);
            g.gain.setValueAtTime(0.0001, now);
            g.gain.exponentialRampToValueAtTime(gain, now + 0.01);
            o.start(now); g.gain.exponentialRampToValueAtTime(0.0001, now + dur);
            o.stop(now + dur + 0.02);
        }
        function soundCorrect() { tone(880, 0.09); tone(1100, 0.09, 0.1); tone(1400, 0.12, 0.22); }
        function soundWrong() { tone(220, 0.14); tone(170, 0.14, 0.12); tone(120, 0.2, 0.3); }

        // 🧠 Perguntas
        const allQuestions = [
            { q: 'Qual é o bioma predominante no Estado de Goiás?', a: ['Caatinga', 'Cerrado', 'Amazônia', 'Mata Atlântica'], c: 1, hint: 'O bioma típico de Goiás é uma savana brasileira.' },
            { q: 'Qual fruto típico do cerrado e rico em vitamina C?', a: ['Caju', 'Açaí', 'Pequi', 'Jabuticaba'], c: 2, hint: 'É usado em arroz e tem cheiro forte.' },
            { q: 'Relação entre Cerrado e aquífero Guarani?', a: ['Nenhuma', 'Cerrado contribui para recarga de aquíferos', "Cerrado é corpo d'água", 'Cerrado é impermeável'], c: 1, hint: 'O Cerrado é fundamental para as nascentes e recarga dos aquíferos.' },
            { q: 'Qual animal é símbolo do cerrado brasileiro?', a: ['Onça-pintada', 'Lobo-guará', 'Capivara', 'Mico-leão-dourado'], c: 1, hint: 'É um canídeo de pernas longas e pelagem avermelhada.' },
            { q: 'Qual é a vegetação típica do cerrado?', a: ['Mata densa', 'Campos úmidos', 'Savana/arbustos esparsos', 'Manguezal'], c: 2, hint: 'Tem árvores tortas e espaçadas.' }
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

        // 🔧 UI
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
            document.getElementById('btnUni').disabled = usedHelps.uni;
            document.getElementById('btn5050').disabled = usedHelps.fifty;
            document.getElementById('btnSkip').disabled = usedHelps.skip;
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
            if (teams.filter(t => !t.eliminated).length === 1) {
                const winner = teams.find(t => !t.eliminated);
                qEl.textContent = `🏆 ${winner.name} venceu o jogo!`;
                oEl.innerHTML = ''; msgEl.textContent = '';
                return;
            }
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
            ensureAudio();
            const q = allQuestions[currentQuestion % allQuestions.length];
            const opts = document.querySelectorAll('.option');
            opts.forEach(b => b.classList.add('disabled'));
            if (i === q.c) {
                opts[i].classList.add('correct');
                msgEl.textContent = '✅ Resposta correta!';
                soundCorrect();
                const t = teams[currentTeam];
                t.corrects++;
                if (t.corrects % 3 === 0) t.lives++;
            } else {
                opts[i].classList.add('wrong');
                opts[q.c].classList.add('correct');
                teams[currentTeam].lives--;
                if (teams[currentTeam].lives <= 0) {
                    teams[currentTeam].eliminated = true;
                    msgEl.textContent = '❌ Errou e ficou sem vidas! 💀 Eliminado!';
                } else {
                    msgEl.textContent = '❌ Resposta errada!';
                }
                soundWrong();
            }
            setTimeout(nextTurn, 2500);
        }

        function nextTurn() {
            do {
                currentTeam = (currentTeam + 1) % teams.length;
                if (currentTeam === 0) round++;
            } while (teams[currentTeam].eliminated);
            currentQuestion++;
            loadQuestion();
        }

        // 💡 Ajudas
        document.getElementById('btnUni').onclick = function () {
            const q = allQuestions[currentQuestion % allQuestions.length];
            msgEl.textContent = '🎓 Universitários: ' + q.hint;
            usedHelps.uni = true;
            updateUI();
        };

        document.getElementById('btn5050').onclick = function () {
            const q = allQuestions[currentQuestion % allQuestions.length];
            const opts = document.querySelectorAll('.option');
            let wrongs = [0, 1, 2, 3].filter(x => x !== q.c);
            wrongs.sort(() => Math.random() - 0.5);
            wrongs.slice(0, 2).forEach(i => opts[i].disabled = true);
            msgEl.textContent = '🌓 Duas opções foram removidas!';
            usedHelps.fifty = true;
            updateUI();
        };

        document.getElementById('btnSkip').onclick = function () {
            msgEl.textContent = '⏭️ Pergunta passada para a próxima dupla!';
            usedHelps.skip = true;
            setTimeout(nextTurn, 1500);
            updateUI();
        };

        // 🚀 Inicialização
        renderTeams();
        loadQuestion();
    </script>
</body>

</html>
