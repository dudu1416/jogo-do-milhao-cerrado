
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

    <body>
        <!-- 🕒 Cronômetro -->
        <div id="timer" style="
  position: fixed;
  top: 12px;
  right: 20px;
  font-size: 2.4rem;
  font-weight: 900;
  color: #ffd43b;
  text-shadow: 0 0 12px #ffde78;
  font-family: 'Trebuchet MS', sans-serif;
  z-index: 9999;
">35</div>


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

            <footer>Regras: 3 acertos = +1 vida | 0 vidas = eliminação | Uma pergunta por vez para cada dupla.
            </footer>
        </div>

        <script>
            // 🔊 Sistema de som
            // 🎵 Música de fundo contínua (loop)
            const audio = new Audio("assets/opening.mp3");
            audio.loop = true;
            audio.volume = 0.6;

            // toca assim que o usuário interagir
            audio.play().catch(() => {
                console.warn("A música será iniciada após interação do usuário.");
            });


            function soundCorrect() { tone(880, 0.09); tone(1100, 0.09, 0.1); tone(1400, 0.12, 0.22); }
            function soundWrong() { tone(220, 0.14); tone(170, 0.14, 0.12); tone(120, 0.2, 0.3); }

            // 🧠 Perguntas


            // 🔥 Banco de perguntas do Jogo do Milhão Cerrado
            const allQuestions = [
                { q: 'Qual é o segundo maior bioma do Brasil em extensão territorial?', a: ['Cerrado', 'Amazônia', 'Caatinga', 'Mata Atlântica'], c: 0 },
                { q: 'O Cerrado é considerado o berço das águas do Brasil por abrigar nascentes de quais bacias?', a: ['Amazônica e Platina', 'São Francisco e Paraná', 'Paraná, Tocantins e São Francisco', 'Tocantins e Parnaíba'], c: 2 },
                { q: 'Qual o tipo predominante de clima no Cerrado brasileiro?', a: ['Equatorial úmido', 'Tropical sazonal', 'Semiárido', 'Subtropical úmido'], c: 1 },
                { q: 'O solo do Cerrado é naturalmente pobre em nutrientes por ser:', a: ['Raso e arenoso', 'Ácido e rico em alumínio', 'Argiloso e fértil', 'Rico em cálcio e potássio'], c: 1 },
                { q: 'A vegetação do Cerrado apresenta adaptações como cascas grossas e raízes profundas para resistir a:', a: ['Baixas temperaturas', 'Queimadas e seca', 'Ventos fortes', 'Excesso de chuva'], c: 1 },
                { q: 'O bioma Cerrado abrange quantos estados brasileiros, aproximadamente?', a: ['5', '8', '12', '14'], c: 2 },
                { q: 'O Cerrado é classificado, do ponto de vista fitogeográfico, como uma:', a: ['Floresta tropical', 'Savanas tropicais', 'Mata de galeria', 'Caatinga úmida'], c: 1 },
                { q: 'A biodiversidade do Cerrado corresponde a cerca de quantas espécies de plantas conhecidas?', a: ['2 mil', '4 mil', '6 mil', '12 mil'], c: 3 },
                { q: 'O termo “vereda” designa qual tipo de ambiente no Cerrado?', a: ['Campo com árvores baixas', 'Região alagada com buritis', 'Área de solo arenoso e seco', 'Campo limpo sem árvores'], c: 1 },
                { q: 'Que animal é considerado símbolo da fauna do Cerrado?', a: ['Onça-pintada', 'Lobo-guará', 'Tamanduá-bandeira', 'Tatu-canastra'], c: 1 },
                { q: 'As queimadas naturais no Cerrado ocorrem principalmente:', a: ['No período chuvoso', 'Na estação seca', 'Durante o inverno', 'Durante o outono'], c: 1 },
                { q: 'Qual a principal ameaça atual à conservação do Cerrado?', a: ['Expansão agrícola', 'Mudanças climáticas', 'Turismo predatório', 'Caça de subsistência'], c: 0 },
                { q: 'A fisionomia “cerradão” caracteriza-se por:', a: ['Campo com poucas árvores', 'Vegetação mais densa e alta', 'Área úmida com buritis', 'Campo limpo sem arbustos'], c: 1 },
                { q: 'Qual destas plantas é típica do Cerrado?', a: ['Ipê-amarelo', 'Araucária', 'Castanheira', 'Açaizeiro'], c: 0 },
                { q: 'A floresta de galeria é importante porque:', a: ['Protege margens dos rios e nascentes', 'Aumenta o desmatamento', 'Favorece erosão', 'É usada para pasto'], c: 0 },
                { q: 'O Cerrado contribui para a recarga de aquíferos como o:', a: ['Alter do Chão', 'Guarani', 'Paraguaçu', 'Amazonas'], c: 1 },
                { q: 'Qual a média de altitude das chapadas do Cerrado?', a: ['100 a 300m', '500 a 700m', '800 a 1200m', '1500 a 2000m'], c: 2 },
                { q: 'Em termos de área, o Cerrado cobre cerca de que porcentagem do território brasileiro?', a: ['10%', '15%', '22%', '35%'], c: 2 },
                { q: 'A cor avermelhada dos solos do Cerrado é causada pela presença de:', a: ['Cálcio', 'Óxidos de ferro', 'Magnésio', 'Silício'], c: 1 },
                { q: 'O fogo tem papel ecológico importante no Cerrado porque:', a: ['Destrói toda a vegetação', 'Favorece a germinação de espécies adaptadas', 'Aumenta a erosão', 'Reduz a biodiversidade'], c: 1 },
                { q: 'A fauna do Cerrado possui grande número de espécies:', a: ['Marinhas', 'Endêmicas', 'Migratórias', 'Introduzidas'], c: 1 },
                { q: 'As folhas coriáceas (grossas e duras) das plantas do Cerrado servem para:', a: ['Atrair insetos', 'Evitar perda de água', 'Aumentar fotossíntese', 'Facilitar a reprodução'], c: 1 },
                { q: 'Que tipo de vegetação forma o contato entre Cerrado e Floresta Amazônica?', a: ['Ecótono', 'Restinga', 'Mata de araucária', 'Caatinga'], c: 0 },
                { q: 'O lobo-guará tem alimentação considerada:', a: ['Apenas carnívora', 'Onívora', 'Herbívora', 'Insetívora'], c: 1 },
                { q: 'O Cerrado apresenta forte sazonalidade com chuvas concentradas em que estação?', a: ['Verão', 'Outono', 'Inverno', 'Primavera'], c: 0 },
                { q: 'O Cerrado está localizado principalmente em qual região do Brasil?', a: ['Sul', 'Sudeste', 'Centro-Oeste', 'Nordeste'], c: 2 },
                { q: 'O desmatamento no Cerrado favorece a perda de:', a: ['Espécies endêmicas e nascentes', 'Solos férteis', 'Água salina', 'Aumento de biomassa'], c: 0 },
                { q: 'O nome “Cerrado” vem de:', a: ['Vegetação densa e fechada', 'Região aberta com poucas árvores', 'Solos rasos e duros', 'Área montanhosa'], c: 0 },
                { q: 'A presença de espécies como o pequi e o barbatimão indica:', a: ['Solos ricos', 'Áreas úmidas', 'Vegetação típica do Cerrado', 'Ambiente pantanoso'], c: 2 },
                { q: 'O bioma Cerrado é considerado um hotspot mundial de biodiversidade por:', a: ['Baixa diversidade', 'Alta taxa de endemismo e destruição acelerada', 'Ser frio e seco', 'Alta pluviosidade'], c: 1 }
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

                // 🕒 reiniciar cronômetro de 35 segundos
                clearInterval(window.timerInterval);
                let timeLeft = 35;
                const timerDisplay = document.getElementById('timer');
                timerDisplay.textContent = timeLeft;
                timerDisplay.style.color = '#ffd43b';
                timerDisplay.style.textShadow = '0 0 12px #ffde78';

                window.timerInterval = setInterval(() => {
                    timeLeft--;
                    timerDisplay.textContent = timeLeft;

                    // alerta visual
                    if (timeLeft === 10) {
                        timerDisplay.style.color = '#ffcc00';
                        timerDisplay.style.textShadow = '0 0 15px #ffcc00';
                    }
                    if (timeLeft === 5) {
                        timerDisplay.style.color = '#ff0000';
                        timerDisplay.style.textShadow = '0 0 15px #ff3333';
                    }

                    // se o tempo acabar
                    if (timeLeft <= 0) {
                        clearInterval(window.timerInterval);
                        timerDisplay.textContent = '⏰';
                        msgEl.textContent = '⏰ Tempo esgotado! Dupla eliminada!';
                        soundWrong();
                        eliminateCurrentTeam();
                    }
                }, 1000);
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
                        msgEl.textContent = '❌ Errou! 💀 Eliminado!';
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
            // 🔥 Abertura do jogo
            // 🔥 Abertura do jogo
            // 🔥 Abertura do jogo (corrigida: cria overlay em vez de usar .container inexistente)
            function playOpening() {
                // cria overlay full-screen
                const overlay = document.createElement('div');
                overlay.id = 'opening-overlay';
                Object.assign(overlay.style, {
                    position: 'fixed',
                    inset: '0',
                    display: 'flex',
                    alignItems: 'center',
                    justifyContent: 'center',
                    flexDirection: 'column',
                    gap: '18px',
                    background: 'linear-gradient(180deg, rgba(3,4,26,0.95), rgba(3,4,26,0.98))',
                    color: '#fff',
                    zIndex: 9999,
                    textAlign: 'center',
                    padding: '24px'
                });

                overlay.innerHTML = `
    <div style="max-width:900px;">
      <h1 style="font-size:3rem;margin:0 0 8px;color:#ffd43b;text-shadow:0 0 15px #ffd43b;">💰 Jogo do Milhão Cerrado 💰</h1>
      <p style="margin:0 0 18px;font-size:1.15rem;opacity:0.95;">Teste seus conhecimentos — prepare a dupla e boa sorte!</p>
      <div style="display:flex;gap:12px;justify-content:center;flex-wrap:wrap;margin-top:8px;">
        <button id="startGame" style="background:#0bb14a;border:none;padding:12px 26px;border-radius:10px;font-weight:800;cursor:pointer;">Começar ▶</button>
        <button id="skipIntro" style="background:transparent;border:1px solid rgba(255,255,255,0.06);padding:12px 20px;border-radius:10px;cursor:pointer;">Pular Abertura</button>
      </div>
    </div>
  `;

                // adiciona ao body
                document.body.appendChild(overlay);

                // pequena vinheta sonora (usa WebAudio para consistência)
                try {
                    const tnow = audioCtx.currentTime;
                    tone(440, 0.08, 0, 'sine', 0.12);
                    tone(660, 0.08, 0.09, 'sine', 0.12);
                    tone(880, 0.10, 0.18, 'sine', 0.12);
                } catch (e) {
                    // se audioCtx não estiver pronto, ignora
                }

                // animação de fade-out e remoção do overlay
                function closeOverlay() {
                    overlay.style.transition = 'opacity 600ms ease';
                    overlay.style.opacity = '0';
                    setTimeout(() => {
                        if (overlay.parentNode) overlay.parentNode.removeChild(overlay);
                        loadQuestion(); // inicia o jogo
                    }, 600);
                }

                document.getElementById('startGame').addEventListener('click', closeOverlay);
                document.getElementById('skipIntro').addEventListener('click', () => {
                    // simplesmente remove sem som e inicia
                    if (overlay.parentNode) overlay.parentNode.removeChild(overlay);
                    loadQuestion();
                });
            }

            // 🚀 Iniciar a abertura quando o script carregar
            playOpening();
