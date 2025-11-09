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
    <div id="timer"
        style="position: fixed; top: 12px; right: 20px; font-size: 2.4rem; font-weight: 900; color: #ffd43b; text-shadow: 0 0 12px #ffde78; font-family: 'Trebuchet MS', sans-serif; z-index: 9999;">
        60</div>

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


            { q: 'Qual é o bioma predominante na região central do Brasil, conhecido por sua vegetação de savana tropical?', a: ['Caatinga', 'Cerrado', 'Mata Atlântica', 'Pantanal'], c: 1 },
            { q: 'Qual é a principal característica que diferencia o Cerrado de uma savana africana?', a: ['Menor densidade de árvores', 'Maior altitude média', 'Presença de solos ricos em ferro e alumínio', 'Maior umidade anual'], c: 2 },
            { q: 'O Cerrado é conhecido como “berço das águas” porque:', a: ['Possui muitas lagoas e lagos', 'Apresenta grande número de nascentes de grandes bacias hidrográficas', 'Tem o maior índice pluviométrico do Brasil', 'Concentra rios de planície'], c: 1 },
            { q: 'A vegetação do Cerrado apresenta adaptações estruturais como:', a: ['Folhas largas e finas', 'Troncos finos e lisos', 'Cascas espessas e raízes profundas', 'Presença de folhas suculentas'], c: 2 },
            { q: 'A estação seca no Cerrado ocorre geralmente entre:', a: ['Novembro e março', 'Janeiro e maio', 'Maio e setembro', 'Agosto e dezembro'], c: 2 },
            { q: 'Em relação à fauna, o Cerrado abriga espécies como:', a: ['Onça-pintada e boto-cor-de-rosa', 'Lobo-guará e tamanduá-bandeira', 'Ariranha e macaco-aranha', 'Peixe-boi e mutum-de-alagoas'], c: 1 },
            { q: 'O processo de lixiviação intensa nos solos do Cerrado resulta em:', a: ['Aumento de sais minerais', 'Perda de nutrientes', 'Acúmulo de matéria orgânica', 'Redução da acidez'], c: 1 },
            { q: 'O bioma Cerrado ocupa cerca de quantos milhões de km² do território brasileiro?', a: ['1,5 milhão', '2 milhões', '2,5 milhões', '3 milhões'], c: 1 },
            { q: 'A principal causa da degradação do Cerrado é:', a: ['O desmatamento para agricultura e pecuária', 'A caça de subsistência', 'A urbanização acelerada', 'A mineração artesanal'], c: 0 },
            { q: 'Qual tipo de vegetação é mais denso dentro do bioma Cerrado?', a: ['Campo limpo', 'Cerradão', 'Vereda', 'Campo rupestre'], c: 1 },
            { q: 'O pequi é uma planta típica do Cerrado usada para:', a: ['Fabricação de tecidos', 'Combustível natural', 'Alimentação e cosméticos', 'Medicamentos alopáticos'], c: 2 },
            { q: 'O clima predominante do Cerrado é:', a: ['Tropical sazonal com inverno seco', 'Equatorial úmido', 'Tropical de altitude', 'Semiárido quente'], c: 0 },
            { q: 'A região que mais perdeu área de Cerrado nas últimas décadas é:', a: ['Centro-Oeste', 'Nordeste', 'Sudeste', 'Sul'], c: 0 },
            { q: 'O que ocorre com a biodiversidade do Cerrado após queimadas frequentes?', a: ['Aumenta rapidamente', 'Permanece estável', 'Diminui por perda de espécies sensíveis', 'Fica mais resistente'], c: 2 },
            { q: 'Qual destas espécies é endêmica do Cerrado?', a: ['Tatu-bola', 'Arara-azul-grande', 'Lobo-guará', 'Tamanduá-mirim'], c: 2 },
            { q: 'O termo “mata de galeria” refere-se a:', a: ['Floresta que acompanha cursos d’água', 'Região de transição para o Pantanal', 'Área de solo arenoso e pobre', 'Vegetação rasteira e aberta'], c: 0 },
            { q: 'A cor avermelhada dos solos do Cerrado é causada por:', a: ['Óxidos de ferro', 'Carbonato de cálcio', 'Sílica', 'Magnésio'], c: 0 },
            { q: 'O Cerrado é um bioma considerado hotspot mundial por:', a: ['Baixa taxa de endemismo', 'Alto nível de desmatamento e alta biodiversidade', 'Baixa pluviosidade e baixa densidade populacional', 'Ser o bioma mais preservado'], c: 1 },
            { q: 'A polinização de muitas plantas do Cerrado depende principalmente de:', a: ['Abelhas nativas', 'Pássaros migratórios', 'Vento', 'Répteis'], c: 0 },
            { q: 'O fogo natural no Cerrado tem importância ecológica porque:', a: ['Destrói completamente a vegetação', 'Promove a germinação de sementes adaptadas', 'Aumenta a erosão', 'Diminui a fertilidade'], c: 1 },
            { q: 'O principal tipo de relevo predominante no Cerrado é:', a: ['Planícies aluviais', 'Chapadas e planaltos', 'Depressões e vales úmidos', 'Regiões montanhosas'], c: 1 },
            { q: 'Qual dos estados abaixo possui maior área de Cerrado?', a: ['Bahia', 'Goiás', 'Paraná', 'Acre'], c: 1 },
            { q: 'O Cerrado contribui diretamente para a recarga de qual grande aquífero?', a: ['Aquífero Alter do Chão', 'Aquífero Guarani', 'Aquífero Serra Geral', 'Aquífero Botucatu'], c: 1 },
            { q: 'As queimadas humanas afetam o Cerrado porque:', a: ['Substituem o fogo natural e alteram ciclos ecológicos', 'Reduzem o número de nascentes', 'Aumentam a quantidade de ferro no solo', 'Facilitam a regeneração'], c: 0 },
            { q: 'A principal forma de conservação do Cerrado atualmente é por meio de:', a: ['Reservas extrativistas e parques nacionais', 'Agronegócio sustentável', 'Plantio de eucalipto', 'Expansão urbana planejada'], c: 0 },
            { q: 'O nome “Cerrado” vem do termo que significa:', a: ['Área fechada e densa', 'Campo aberto', 'Terreno montanhoso', 'Solo fértil'], c: 0 },
            { q: 'As árvores do Cerrado frequentemente apresentam galhos retorcidos porque:', a: ['O solo raso impede o crescimento vertical', 'A adaptação ao vento e à seca molda seu formato', 'São podadas por herbívoros', 'Há excesso de umidade'], c: 1 },
            { q: 'O fruto do barbatimão é conhecido por:', a: ['Ser comestível e usado em doces', 'Ter propriedades medicinais adstringentes', 'Conter óleo essencial aromático', 'Ser venenoso'], c: 1 },
            { q: 'A perda do Cerrado pode afetar diretamente o regime hídrico de quais bacias?', a: ['Amazonas, São Francisco e Paraná', 'Parnaíba, Tocantins e Uruguai', 'São Francisco, Paraná e Tocantins', 'Paraná, Paraguai e Madeira'], c: 2 },
            { q: 'O solo do Cerrado é naturalmente ácido e pobre, mas pode ser utilizado para agricultura após:', a: ['Aeração natural', 'Correção com calcário e adubação', 'Plantio direto', 'Remoção da camada superficial'], c: 1 },
            { q: 'Em termos de biodiversidade vegetal, o Cerrado abriga aproximadamente:', a: ['2 mil espécies', '5 mil espécies', '12 mil espécies', '20 mil espécies'], c: 2 }
        ];


        // ⚙️ Variáveis principais
        let teams = [
            { name: 'Dupla 1', lives: 1 , corrects: 0, eliminated: false },
            { name: 'Dupla 2', lives: 1, corrects: 0, eliminated: false },
            { name: 'Dupla 3', lives: 1, corrects: 0, eliminated: false },
            { name: 'Dupla 4', lives: 1, corrects: 0, eliminated: false }
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
            let timeLeft = 60;
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
