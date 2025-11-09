<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Jogo do Milhão Cerrado</title>
  <link rel="icon" href="favicon.ico" />
  <style>
    :root{
      --bg1:#03041a;
      --panel: rgba(7,16,50,0.88);
      --accent:#ffd43b;
      --accent-2:#ffb84d;
      --option:#093d86;
      --option-hover:#1252b3;
      --correct:#0bb14a;
      --wrong:#e53935;
      --glass: rgba(255,255,255,0.03);
    }

    html,body{height:100%;margin:0;font-family:Inter,Arial,sans-serif;background:#000;}
    body{
      background-image: url('assets/cerrado.jpg');
      background-size: cover;
      background-position: center;
      background-attachment: fixed;
      color:#fff;
      display:flex;
      align-items:center;
      justify-content:center;
      padding:18px;
    }

    .wrap{
      width:100%;
      max-width:1200px;
      border-radius:14px;
      padding:14px;
      display:grid;
      grid-template-columns:1fr 360px;
      gap:18px;
      background: linear-gradient(180deg, rgba(0,0,0,0.45), rgba(0,0,0,0.6));
      box-shadow:0 20px 60px rgba(0,0,0,0.6);
      backdrop-filter: blur(6px);
    }

    header{grid-column:1/3;display:flex;justify-content:space-between;align-items:center;padding:6px 12px}
    h1{margin:0;color:var(--accent);font-size:1.1rem}
    .subtitle{font-size:0.85rem;opacity:.9}

    main{background:var(--panel);padding:16px;border-radius:12px}
    #roundInfo{display:flex;gap:10px;align-items:center;margin-bottom:10px}
    #question{font-size:1.25rem;padding:14px;border-radius:10px;background:linear-gradient(90deg, rgba(255,255,255,0.02), rgba(0,0,0,0.04));border:1px solid rgba(255,255,255,0.03)}
    .options{display:grid;grid-template-columns:repeat(2,1fr);gap:12px;margin-top:12px}
    .option{padding:14px;border-radius:10px;border:1px solid rgba(255,255,255,0.05);background:linear-gradient(180deg,#06224a,#04203b);cursor:pointer;font-weight:700;display:flex;align-items:center;min-height:64px;transition:transform .12s,box-shadow .12s}
    .option:hover:not(.disabled){transform:scale(1.02);box-shadow:0 8px 30px rgba(0,0,0,0.4)}
    .option.disabled{opacity:0.45;cursor:not-allowed}
    .option.correct{background:linear-gradient(90deg,#0b6b2a,#0ab14a);box-shadow:0 8px 30px rgba(10,180,80,0.12)}
    .option.wrong{background:linear-gradient(90deg,#7b1720,#e53935);box-shadow:0 8px 30px rgba(229,57,53,0.12)}

    .controls{display:flex;gap:8px;margin-top:14px;align-items:center}
    .control-btn{background:#071f3a;border:1px solid rgba(255,255,255,0.05);padding:8px 12px;border-radius:8px;cursor:pointer;color:var(--accent);font-weight:700}
    .control-btn:disabled{opacity:0.4;cursor:not-allowed}

    .timer{margin-left:10px;font-weight:800;padding:6px 10px;border-radius:8px;background:rgba(255,255,255,0.03);border:1px solid rgba(255,255,255,0.03)}

    .lights{height:8px;width:100%;margin-top:12px;display:flex;gap:6px}
    .light{flex:1;height:8px;border-radius:6px;background:rgba(255,255,255,0.03);transition:background .18s,transform .18s}
    .light.flash-correct{background:linear-gradient(90deg,#0bb14a,#06a83a);transform:scaleY(1.6)}
    .light.flash-wrong{background:linear-gradient(90deg,#e53935,#b71c1c);transform:scaleY(1.6)}

    .side{display:flex;flex-direction:column;gap:12px}
    .prizes,.teams{background:rgba(0,0,0,0.45);padding:12px;border-radius:10px;border:1px solid rgba(255,255,255,0.03)}
    .team{display:flex;justify-content:space-between;align-items:center;padding:8px;border-radius:8px;margin-bottom:8px}
    .team.eliminated{opacity:.35;text-decoration:line-through}

    footer{grid-column:1/3;text-align:center;font-size:.82rem;opacity:.9;margin-top:8px}

    /* opening overlay styles */
    #opening-overlay {
      position:fixed; inset:0; z-index:9999;
      display:flex; align-items:center; justify-content:center; flex-direction:column;
      background: linear-gradient(180deg, rgba(3,4,10,0.95), rgba(3,4,10,0.98));
      color:#fff; text-align:center; padding:24px;
    }
    #opening-overlay h1{font-size:3rem;margin:0 0 8px;color:var(--accent); text-shadow:0 0 18px rgba(255,212,59,0.6)}
    #opening-overlay p{margin:0 0 18px;font-size:1.1rem;opacity:.95}
    .open-buttons{display:flex;gap:12px;justify-content:center;flex-wrap:wrap}
    .open-buttons button{padding:12px 28px;border-radius:10px;border:none;cursor:pointer;font-weight:800}
    #startGame{background:#0bb14a;color:#04203b}
    #skipIntro{background:transparent;border:1px solid rgba(255,255,255,0.06);color:#fff}

    @media(max-width:980px){
      .wrap{grid-template-columns:1fr}
      .side{order:2}
      main{order:1}
    }
  </style>
</head>
<body>
  <div class="wrap" role="application" aria-label="Jogo do Milhão Cerrado">
    <header>
      <div>
        <h1>Jogo do Milhão Cerrado — criado por Carlos, Arthur, Wendel, Marcos, Guilherme</h1>
        <div class="subtitle">Tema: Cerrado — 30 perguntas (médio/difícil) • 4 duplas • +1 vida a cada 3 acertos</div>
      </div>
      <div class="subtitle">Rodada: <span id="roundNumber">1</span> • Pergunta nº <span id="questionNumber">1</span></div>
    </header>

    <main class="main" id="mainPanel">
      <div id="roundInfo">
        <div class="small">Vez da dupla:</div>
        <div id="currentTeam" style="font-weight:900">—</div>
        <div style="flex:1"></div>
        <div class="small">Acertos: <span id="teamCorrects">0</span></div>
        <div class="small">Vidas: <span id="teamLives">0</span></div>
        <div class="timer" id="timer">30s</div>
      </div>

      <div id="question">Carregando pergunta...</div>

      <div class="options" id="options"></div>

      <div class="lights" id="lights">
        <div class="light"></div><div class="light"></div><div class="light"></div><div class="light"></div>
      </div>

      <div class="controls">
        <button id="fifty" class="control-btn">50/50</button>
        <button id="audience" class="control-btn">Plateia</button>
        <button id="skip" class="control-btn">Pular</button>
        <button id="endTurn" class="control-btn">Encerrar Turno</button>
      </div>

      <div id="message" aria-live="polite" style="margin-top:12px;font-weight:800;min-height:28px"></div>
    </main>

    <aside class="side">
      <div class="prizes">
        <div style="font-weight:800;margin-bottom:8px">Escada de prêmios</div>
        <div id="prizeList"></div>
      </div>

      <div class="teams">
        <div style="font-weight:800;margin-bottom:8px">Duplas</div>
        <div id="teamList"></div>
      </div>
    </aside>

    <footer>(Regras) — Cada dupla responde uma pergunta por rodada. Se errar e não tiver vidas, é eliminada. A cada 3 acertos, +1 vida. Lifelines por dupla: 50/50, Plateia, Pular (1x cada).</footer>
  </div>

  <!-- audio element for opening -->
  <audio id="openingAudio" preload="auto" src="assets/opening.mp3"></audio>

  <script>
    /* -----------------------
       WebAudio helpers (kept for effects)
       ----------------------- */
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    function ensureAudio(){ if(audioCtx.state === 'suspended') audioCtx.resume(); }

    function tone(freq, dur = 0.12, when = 0, type = 'sine', gain = 0.25) {
      const now = audioCtx.currentTime + when;
      const o = audioCtx.createOscillator();
      const g = audioCtx.createGain();
      o.type = type; o.frequency.value = freq;
      o.connect(g); g.connect(audioCtx.destination);
      g.gain.setValueAtTime(0.0001, now);
      g.gain.exponentialRampToValueAtTime(gain, now + 0.01);
      o.start(now);
      g.gain.exponentialRampToValueAtTime(0.0001, now + dur);
      o.stop(now + dur + 0.02);
    }

    function soundQuestion(){ tone(440,0.07); tone(660,0.07,0.08); tone(880,0.07,0.16); }
    function soundCorrect(){ tone(880,0.09); tone(1100,0.09,0.1); tone(1400,0.12,0.22); }
    function soundWrong(){ tone(220,0.14); tone(170,0.14,0.12); tone(120,0.2,0.3); }
    function soundWin(){ tone(880,0.09); tone(1100,0.09,0.12); tone(1400,0.12,0.26); }

    /* -----------------------
       Perguntas (30 — Cerrado médio/difícil)
       Cada item: { q, a: [4], c: index, hint: '...' }
       ----------------------- */
    const allQuestions = [
      { q: 'Qual é a principal característica do solo ferruginoso do Cerrado?', a: ['Alta fertilidade natural','Rico em óxidos de ferro e baixo em matéria orgânica','Solo calcário','Solo aluvial muito fértil'], c: 1, hint:'Presença intensa de óxidos de ferro, terreno ácido e com baixa matéria orgânica.' },
      { q: 'Qual processo lêxico (biológico) explica a presença de raízes tuberosas em muitas espécies do Cerrado?', a: ['Adaptação a alagamentos','Armazenamento de água e nutrientes para sobreviver à seca','Resposta à salinidade','Facilitar dispersão por animais'], c: 1, hint:'Armazenam reservas para sobreviver ao longo período seco.' },
      { q: 'Qual é o papel ecológico das veredas no Cerrado?', a: ['Servem apenas como pastagem', 'Funcionam como corredores hídricos e de biodiversidade', 'São áreas de cultivo','São formações urbanas'], c: 1, hint:'Associação com buriti e áreas alagadas que mantêm aquíferos e biodiversidade.' },
      { q: 'A que formação vegetal se refere o termo "cerradão"?', a: ['Uma savana aberta com gramíneas predominantes','Uma fisionomia mais densa e arborizada do Cerrado', 'Área pantanosa', 'Área de restinga'], c: 1, hint:'Mais árvores, copa contínua, aspecto de floresta sazonal.' },
      { q: 'Qual o impacto direto da monocultura extensiva (soja/silvicultura) no Cerrado?', a: ['Aumento da biodiversidade','Fragmentação de habitats e perda de endemismos','Melhora da recarga de aquíferos','Aumento de veredas naturais'], c: 1, hint:'Transforma paisagem e reduz hábitats nativos.' },
      { q: 'O Cerrado é conhecido como "berço d’água". Por quê?', a: ['Porque tem muitos lagos', 'Porque abriga nascentes que alimentam grandes bacias', 'Porque é inundado o ano todo', 'Porque tem rios largos e profundos'], c: 1, hint:'Nascem rios que abastecem várias bacias importantes.' },
      { q: 'Qual aquífero tem sua recarga substancialmente influenciada por áreas do Cerrado?', a: ['Aquífero Guarani','Aquífero Alter do Chão','Aquífero Amazonas','Aquífero Pantanal'], c: 0, hint:'Guarani é um dos maiores aquíferos da América do Sul e recebe recarga do Planalto Central.' },
      { q: 'Qual animal endêmico do Cerrado é adaptado a longas distâncias e comportamento oportunista (frugívoro/onívoro)?', a: ['Onça-pintada','Lobo-guará','Tamanduá-bandeira','Peixe-boi'], c: 1, hint:'Mamífero com longas pernas e dieta variada, símbolo do Cerrado.' },
      { q: 'Qual dos seguintes fatores agronômicos favorece a erosão acelerada no Cerrado convertido para agricultura?', a: ['Plantio direto com cobertura permanente','Queima controlada','Remoção de vegetação nativa sem práticas conservacionistas','Uso de terraceamento'], c: 2, hint:'Perda de cobertura vegetal nativa expõe o solo.' },
      { q: 'Por que muitas espécies do Cerrado têm folhas coriáceas e com estômatos pouco ativos?', a: ['Para aumentar a fotossíntese no frio','Para reduzir perda de água durante períodos secos','Para atrair polinizadores','Para facilitar dispersão'], c: 1, hint:'Adaptação ao déficit hídrico sazonal.' },
      { q: 'O que é "mata ciliar" e por que é crucial no Cerrado?', a: ['Formação costeira','Vegetação que acompanha cursos d’água e protege margens e nascentes','Área urbana ao lado de rios','Solo que retém água'], c: 1, hint:'Protege rios, mantém sombra e evita assoreamento.' },
      { q: 'Qual vegetação típica do Cerrado é usada tradicionalmente na confecção de licor e óleos regionais?', a: ['Jabuticaba','Pequi','Cajueiro','Açaí'], c: 1, hint:'Fruto com polpa oleosa e sabor marcante, muito usado regionalmente.' },
      { q: 'O que torna o Cerrado um hotspot de conservação?', a: ['Baixa diversidade','Grande diversidade, alto endemismo e forte conversão por atividades humanas','Clima frio e estável','Somente cobertura de árvores'], c: 1, hint:'Muitos endemismos e pressão antrópica elevada.' },
      { q: 'Qual prática de manejo agrícola é considerada mais adequada para reduzir impactos no Cerrado?', a: ['Desmatamento e rotação zero','Integração lavoura-pecuária-floresta (ILPF)','Queima de restos a cada safra','Monocultura extensiva sem rotação'], c: 1, hint:'ILPF combina produção com conservação e pode reduzir pressão sobre áreas nativas.' },
      { q: 'O que é "ecótono" no contexto do Cerrado?', a: ['Um tipo de solo','Zona de transição entre dois biomas, com mistura de espécies','Uma espécie de planta','Um rio'], c: 1, hint:'Transição entre Cerrado e Mata Atlântica/Amazônia, por exemplo.' },
      { q: 'As "veredas" estão associadas frequentemente a qual tipo de solo/hidrologia?', a: ['Solos arenosos extremamente secos','Áreas sazonalmente encharcadas com lençol freático aflorante','Rochas expostas','Planícies costeiras'], c: 1, hint:'Áreas úmidas com buritis e presença permanente/temporária de água.' },
      { q: 'Qual o efeito das queimadas frequentes e intensas não controladas no Cerrado?', a: ['Aumento de biodiversidade nativa','Perda de organominerais do solo e mortalidade de fauna e flora','Melhora da estrutura do solo','Criação de mais veredas'], c: 1, hint:'Queimadas fora do ciclo natural prejudicam as espécies e o solo.' },
      { q: 'Qual unidade de conservação é referência no Cerrado e conhecida por chapadas e cachoeiras?', a: ['Parque Nacional da Chapada dos Veadeiros','Parque Nacional do Iguaçu','Parque Nacional da Tijuca','Parque Nacional da Serra dos Órgãos'], c: 0, hint:'Local de grande valor ecológico no Planalto Central.' },
      { q: 'Qual característica define espécies endêmicas do Cerrado?', a: ['Distribuição cosmopolita','Exclusividade de ocorrência em uma região geográfica limitada','Altamente migratórias','Introduzidas por humanos'], c: 1, hint:'Encontradas apenas no Cerrado ou em áreas restritas.' },
      { q: 'Como a fragmentação do habitat afeta populações de vertebrados no Cerrado?', a: ['Aumenta conectividade genética','Reduz tamanhos populacionais e aumenta isolamento genético','Proporciona mais nichos com maior diversidade','Fortalece populações locais'], c: 1, hint:'Fragmentos isolados diminuem fluxo gênico e aumentam extinção local.' },
      { q: 'Qual técnica de restauração é frequentemente empregada para recuperar áreas degradadas no Cerrado?', a: ['Plantio de espécies exóticas em massa','Restauração com espécies nativas e controle de invasoras','Impermeabilização do solo','Uso exclusivo de fertilizantes químicos'], c: 1, hint:'Reintroduzir espécies nativas e controlar plantas invasoras.' },
      { q: 'O que é um “sistema de vereda” do ponto de vista ecológico?', a: ['Área urbana verde','Mosaico de área alagada e mosaico de cerrado que funciona como refúgio para fauna aquática e terrestre','Uma técnica agrícola','Uma estrada de terra'], c: 1, hint:'Ecossistema de transição úmido que abriga espécies específicas.' },
      { q: 'Qual é a relação entre a cobertura vegetal do Cerrado e a recarga de aquíferos?', a: ['Cobertura vegetal diminui recarga','Vegetação nativa protege e favorece recarga de aquíferos','Vegetação não influencia','Recarga depende só de chuva'], c: 1, hint:'Cobertura nativa ajuda infiltração e proteção das nascentes.' },
      { q: 'Por que o manejo do fogo deve ser cuidadoso em programas de conservação do Cerrado?', a: ['Fogo sempre elimina espécies nativas','Queimas controladas, se bem planejadas, podem ser ecológicas; queimas intensas são destrutivas','O fogo não é histórico no Cerrado','Fogo substitui manejo de água'], c: 1, hint:'O fogo tem papel ecológico natural, mas o manejo humano pode torná-lo prejudicial.' },
      { q: 'Qual é a principal razão da rápida conversão do Cerrado nas últimas décadas?', a: ['Expansão urbana sem agricultura', 'Expansão da fronteira agrícola e pecuária intensiva', 'Turismo massivo', 'Exploração mineral apenas'], c: 1, hint:'Conversão para agricultura (soja, pastagem) e grandes projetos de infraestrutura.' },
      { q: 'O que significa que uma espécie é "funcionalmente importante" em um ecossistema como o Cerrado?', a: ['Tem grande valor estético','Tem papel central em processos ecológicos (polinização, dispersão, estruturação do habitat)','É a espécie mais rara','É uma espécie introduzida'], c: 1, hint:'Contribui fortemente para funcionamento do ecossistema.' },
      { q: 'Quais serviços ecossistêmicos o Cerrado fornece de maneira crítica para o Brasil?', a: ['Somente alimento','Regulação hídrica, produção de água, biodiversidade e serviços culturais','Apenas madeira','Nada significativo'], c: 1, hint:'Água, biodiversidade e suporte a atividades econômicas.' },
      { q: 'Por que o monitoramento remoto (satélites) é importante no Cerrado?', a: ['É mais caro que trabalho de campo','Permite detectar desmatamento, queimadas e mudanças de uso do solo em larga escala','Não tem utilidade','Somente útil para mapas turísticos'], c: 1, hint:'Permite vigilância espacial e temporal das transformações da paisagem.' },
      { q: 'Qual é um indicador confiável de degradação do solo no Cerrado após conversão agropecuária?', a: ['Aumento da diversidade de plantas nativas','Perda de matéria orgânica e compactação do solo','Aumento de veredas naturais','Melhora da infiltração'], c: 1, hint:'Perda de MO, compactação e perda de estrutura indicam degradação.' }
    ];

    /* -----------------------
       Estado do jogo (duplas)
       ----------------------- */
    const teams = [
      { id: 1, name: 'Dupla A (Carlos & Arthur)', corrects: 0, lives: 0, active: true, used: { fifty: false, audience: false, skip: false } },
      { id: 2, name: 'Dupla B (Wendel & Marcos)', corrects: 0, lives: 0, active: true, used: { fifty: false, audience: false, skip: false } },
      { id: 3, name: 'Dupla C (Guilherme & Jogador)', corrects: 0, lives: 0, active: true, used: { fifty: false, audience: false, skip: false } },
      { id: 4, name: 'Dupla D (Convidada)', corrects: 0, lives: 0, active: true, used: { fifty: false, audience: false, skip: false } }
    ];

    let questionIndex = 0; // pointer into allQuestions
    let round = 1;
    let turnIdx = 0; // index within active teams for current round

    const prizeList = ['1.000', '2.000', '5.000', '10.000', '25.000', '50.000', '100.000', '250.000', '500.000', '1.000.000'];

    /* DOM refs */
    const currentTeamEl = document.getElementById('currentTeam');
    const teamListEl = document.getElementById('teamList');
    const optionsEl = document.getElementById('options');
    const questionEl = document.getElementById('question');
    const roundNumberEl = document.getElementById('roundNumber');
    const questionNumberEl = document.getElementById('questionNumber');
    const teamCorrectsEl = document.getElementById('teamCorrects');
    const teamLivesEl = document.getElementById('teamLives');
    const messageEl = document.getElementById('message');
    const prizeListEl = document.getElementById('prizeList');
    const timerEl = document.getElementById('timer');
    const lights = Array.from(document.querySelectorAll('.light'));

    /* Timer */
    let timerInterval = null;
    let timeLeft = 30;

    /* Initialize UI */
    function init(){
      renderPrizeList();
      renderTeams();
      nextTurn();
    }

    /* Render prizes */
    function renderPrizeList() {
      prizeListEl.innerHTML = '';
      prizeList.slice().reverse().forEach((p, i) => {
        const div = document.createElement('div'); div.className = 'prize';
        div.innerHTML = `<div>${prizeList.length - i}</div><div>R$ ${p}</div>`;
        prizeListEl.appendChild(div);
      });
    }

    /* Render teams */
    function renderTeams() {
      teamListEl.innerHTML = '';
      teams.forEach(t => {
        const d = document.createElement('div'); d.className = 'team' + (t.active ? '' : ' eliminated');
        d.innerHTML = `<div style="font-weight:800">${t.name}</div><div class="small">Acertos: ${t.corrects} • Vidas: ${t.lives}</div>`;
        teamListEl.appendChild(d);
      });
    }

    /* Active teams */
    function getActiveTeams() { return teams.filter(t => t.active); }

    /* Next turn logic */
    function nextTurn() {
      ensureAudio();
      const active = getActiveTeams();
      if (active.length <= 1) {
        endGame(active[0] || null);
        return;
      }
      if (turnIdx >= active.length) { turnIdx = 0; round++; roundNumberEl.textContent = round; }

      const team = active[turnIdx];
      currentTeamEl.textContent = team.name;
      teamCorrectsEl.textContent = team.corrects;
      teamLivesEl.textContent = team.lives;
      messageEl.textContent = '';

      if (questionIndex >= allQuestions.length) {
        endGame(null);
        return;
      }
      const q = allQuestions[questionIndex];
      questionNumberEl.textContent = questionIndex + 1;
      showQuestionForTeam(q, team);
    }

    /* Show question for a given team */
    function showQuestionForTeam(q, team) {
      // play MP3 opening? no — only play question tone
      soundQuestion();
      startTimer(() => { // on timeout treat as wrong attempt
        handleTimeout(team, q);
      });

      questionEl.textContent = q.q;
      optionsEl.innerHTML = '';
      const shuffled = q.a.map((t, i) => ({ t, i })).sort(() => Math.random() - 0.5);
      shuffled.forEach((opt, idx) => {
        const b = document.createElement('button');
        b.className = 'option';
        b.innerHTML = `<strong>${String.fromCharCode(65 + idx)}.</strong>&nbsp; ${opt.t}`;
        b.onclick = () => answerAttempt(opt.i, b, team, q);
        optionsEl.appendChild(b);
      });

      // lifelines state per team
      document.getElementById('fifty').disabled = team.used.fifty;
      document.getElementById('audience').disabled = team.used.audience;
      document.getElementById('skip').disabled = team.used.skip;

      // bind lifelines (they capture current team & q)
      document.getElementById('fifty').onclick = () => useFifty(team, q);
      document.getElementById('audience').onclick = () => useAudience(team, q);
      document.getElementById('skip').onclick = () => useSkip(team, q);
      document.getElementById('endTurn').onclick = () => { // end turn without answering
        messageEl.textContent = `${team.name} encerrou o turno.`; stopTimer(); proceedAfterTurn(true);
      };
    }

    /* Answer attempt */
    function answerAttempt(chosenOrigIndex, btn, team, q) {
      stopTimer();
      Array.from(document.querySelectorAll('.option')).forEach(b => b.disabled = true);
      const correct = chosenOrigIndex === q.c;
      if (correct) {
        btn.classList.add('correct'); flashLights('correct'); soundCorrect();
        team.corrects += 1;
        if (team.corrects % 3 === 0) {
          team.lives += 1;
          messageEl.textContent = `${team.name} acertou e ganhou +1 vida! (vidas: ${team.lives})`;
        } else {
          messageEl.textContent = `${team.name} acertou!`;
        }
        questionIndex++;
        renderTeams();
        setTimeout(() => { proceedAfterTurn(true); }, 1000);
      } else {
        btn.classList.add('wrong'); flashLights('wrong'); soundWrong();
        if (team.lives > 0) {
          team.lives -= 1;
          messageEl.textContent = `${team.name} errou, usou 1 vida e permanece (vidas: ${team.lives}).`;
          questionIndex++;
          renderTeams();
          setTimeout(() => proceedAfterTurn(true), 1200);
        } else {
          team.active = false;
          messageEl.textContent = `${team.name} errou e foi eliminada!`;
          questionIndex++;
          renderTeams();
          setTimeout(() => proceedAfterTurn(false), 1200);
        }
      }
    }

    /* Handle timeout as wrong answer */
    function handleTimeout(team, q) {
      Array.from(document.querySelectorAll('.option')).forEach(b => b.disabled = true);
      flashLights('wrong'); soundTension(0.5); soundWrong();
      if (team.lives > 0) {
        team.lives -= 1;
        messageEl.textContent = `${team.name} estourou o tempo mas consumiu 1 vida (vidas: ${team.lives}).`;
        questionIndex++;
        renderTeams();
        setTimeout(() => proceedAfterTurn(true), 1200);
      } else {
        team.active = false;
        messageEl.textContent = `${team.name} estourou o tempo e foi eliminada!`;
        questionIndex++;
        renderTeams();
        setTimeout(() => proceedAfterTurn(false), 1200);
      }
    }

    /* After turn: advance index or keep if elimination */
    function proceedAfterTurn(advanceTurnIndex) {
      const active = getActiveTeams();
      if (advanceTurnIndex) turnIdx++;
      // if turnIdx beyond active length, nextTurn will reset and increase round
      renderTeams();
      nextTurn();
    }

    /* Lifelines implementations */
    function useFifty(team, q) {
      if (team.used.fifty) return;
      team.used.fifty = true; document.getElementById('fifty').disabled = true;
      const buttons = Array.from(document.querySelectorAll('.option'));
      const mapping = buttons.map((b, idx) => ({ btn: b, origIndex: getOrigIndexFromButtonText(b.innerText, q) }));
      const wrongs = mapping.filter(m => m.origIndex !== q.c);
      wrongs.sort(() => Math.random() - 0.5);
      wrongs.slice(0, 2).forEach(w => { w.btn.disabled = true; w.btn.classList.add('disabled'); });
      messageEl.textContent = `${team.name} usou 50/50.`;
    }

    function useAudience(team, q) {
      if (team.used.audience) return;
      team.used.audience = true; document.getElementById('audience').disabled = true;
      const perc = [0, 0, 0, 0]; let base = 45 + Math.floor(Math.random() * 26);
      perc[q.c] = base; let rem = 100 - base; const wrongs = [0, 1, 2, 3].filter(i => i !== q.c);
      wrongs.forEach((w, i) => { const v = Math.floor(Math.random() * (rem + 1)); perc[w] = v; rem -= v; });
      perc[q.c] += rem;
      messageEl.textContent = `Plateia: A:${perc[0]}% B:${perc[1]}% C:${perc[2]}% D:${perc[3]}%`;
    }

    function useSkip(team, q) {
      if (team.used.skip) return;
      team.used.skip = true; document.getElementById('skip').disabled = true;
      messageEl.textContent = `${team.name} usou Pular e avança sem ganhar vida.`;
      questionIndex++;
      stopTimer();
      setTimeout(() => proceedAfterTurn(true), 800);
    }

    /* helpers to map displayed option text back to original index */
    function getOrigIndexFromButtonText(text, q) {
      const label = text.replace(/^\s*[A-D]\.\s*/, '').trim();
      return q.a.findIndex(x => x === label);
    }

    /* Timer functions */
    function startTimer(onTimeout) {
      stopTimer();
      timeLeft = 30;
      timerEl.textContent = timeLeft + 's';
      soundTension(0.8);
      timerInterval = setInterval(() => {
        timeLeft--;
        timerEl.textContent = timeLeft + 's';
        if (timeLeft <= 5) { tickBeep(); }
        if (timeLeft <= 0) {
          stopTimer();
          if (typeof onTimeout === 'function') onTimeout();
        }
      }, 1000);
    }
    function stopTimer() { if (timerInterval) { clearInterval(timerInterval); timerInterval = null; } }

    /* Visual lights flash */
    function flashLights(type) {
      lights.forEach((l, i) => {
        l.classList.remove('flash-correct', 'flash-wrong');
        setTimeout(() => l.classList.add(type === 'correct' ? 'flash-correct' : 'flash-wrong'), i * 40);
        setTimeout(() => l.classList.remove('flash-correct', 'flash-wrong'), 700 + i * 40);
      });
    }

    /* End game */
    function endGame(winner) {
      stopTimer();
      if (!winner) {
        messageEl.textContent = 'Fim de jogo — sem vencedor claro (sem perguntas restantes ou empate).';
      } else {
        messageEl.textContent = `🎉 Fim de jogo — vencedor: ${winner.name}`;
      }
      soundWin();
      document.getElementById('fifty').disabled = true;
      document.getElementById('audience').disabled = true;
      document.getElementById('skip').disabled = true;
      document.getElementById('endTurn').disabled = true;
      currentTeamEl.textContent = winner ? winner.name : '—';
    }

    /* Start / UI init */
    // init() will be called after opening overlay finishes
    function init(){ renderPrizeList(); renderTeams(); nextTurn(); }

    /* Opening overlay: plays mp3 from assets/opening.mp3 when user clicks Start */
    function playOpeningOverlay(){
      // create overlay
      const overlay = document.createElement('div');
      overlay.id = 'opening-overlay';
      overlay.innerHTML = `
        <div>
          <h1>💰 Jogo do Milhão Cerrado 💰</h1>
          <p>Teste seus conhecimentos sobre o Cerrado — boa sorte!</p>
          <div class="open-buttons">
            <button id="startGame">Começar ▶</button>
            <button id="skipIntro">Pular Abertura</button>
          </div>
        </div>
      `;
      document.body.appendChild(overlay);

      const openingAudio = document.getElementById('openingAudio');

      function startAndClose(){
        // try to play mp3 (user-initiated since called on click)
        if(openingAudio){
          openingAudio.currentTime = 0;
          openingAudio.play().catch(()=>{ /* se bloqueado, ignora */ });
        }
        // small vinheta com WebAudio também
        try{ tone(440,0.06,0,'sine',0.08); tone(660,0.06,0.07,'sine',0.08); tone(880,0.08,0.14,'sine',0.08); }catch(e){}
        overlay.style.transition = 'opacity 500ms';
        overlay.style.opacity = '0';
        setTimeout(()=>{ overlay.remove(); init(); }, 600);
      }

      document.getElementById('startGame').addEventListener('click', startAndClose);
      document.getElementById('skipIntro').addEventListener('click', () => { overlay.remove(); init(); });
    }

    // Ensure DOM loaded before showing overlay
    window.addEventListener('DOMContentLoaded', () => {
      // minor delay to ensure UI elements exist
      setTimeout(() => playOpeningOverlay(), 150);
      // resume audioCtx on first gesture
      ['click','keydown','touchstart'].forEach(ev => document.addEventListener(ev, ensureAudio, { once:true }));
    });
  </script>
</body>
</html>
