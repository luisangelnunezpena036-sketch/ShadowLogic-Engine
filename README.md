<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Juego de Roles Ocultos - Edición Pro</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;600;800&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Plus Jakarta Sans', sans-serif; background-color: #0f172a; color: #f8fafc; overflow-x: hidden; }
        .glass { background: rgba(30, 41, 59, 0.7); backdrop-filter: blur(12px); border: 1px solid rgba(255,255,255,0.1); }
        .role-blur { filter: blur(20px); transition: filter 0.3s ease; }
        .role-visible { filter: blur(0); }
        .suspicion-card { border-left: 4px solid #ef4444; background: rgba(239, 68, 68, 0.1); }
        .custom-scroll::-webkit-scrollbar { width: 6px; }
        .custom-scroll::-webkit-scrollbar-thumb { background: #334155; border-radius: 10px; }
    </style>
</head>
<body class="min-h-screen flex items-center justify-center p-4">

    <!-- Contenedor Principal -->
    <div class="w-full max-w-4xl">
        
        
        <!-- PANTALLA 1: Configuración -->
        <div id="screen-setup" class="glass p-8 rounded-3xl shadow-2xl text-center space-y-6">
            <div class="space-y-2">
                <h1 class="text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-indigo-400 to-cyan-400">JUEGO DE ROLES</h1>
                <p class="text-slate-400">Prototipo de Programación - Debate y Sospechas</p>
            </div>
            <div class="py-6">
                <label class="block text-sm font-bold text-slate-500 uppercase mb-3">Cantidad de Participantes</label>
                <input type="number" id="input-players" min="10" max="15" value="12" 
                       class="bg-slate-800 border border-slate-700 rounded-2xl p-4 text-2xl text-center w-32 focus:ring-2 ring-indigo-500 outline-none">
            </div>
            <button onclick="startGame()" class="w-full py-4 bg-indigo-600 hover:bg-indigo-500 rounded-2xl font-bold text-xl transition-all shadow-lg shadow-indigo-500/20">
                CONFIGURAR PARTIDA
            </button>
        </div>

        <!-- PANTALLA 2: Revelación de Roles -->
        <div id="screen-roles" class="hidden glass p-8 rounded-3xl space-y-6 text-center">
            <h2 class="text-2xl font-bold text-indigo-300">Asignación de Cargos</h2>
            <div class="p-6 bg-slate-900/50 rounded-2xl border border-slate-700">
                <p class="text-slate-400 mb-2">Entregue el dispositivo a:</p>
                <h3 id="current-player-name" class="text-3xl font-black text-white">Jugador_1</h3>
            </div>
            <div class="relative group">
                <div id="role-display-box" class="role-blur bg-indigo-900/30 p-8 rounded-2xl border-2 border-dashed border-indigo-500/30">
                    <span id="role-name" class="text-2xl font-bold text-indigo-400">ROL SECRETO</span>
                </div>
                <button onmousedown="revealRole(true)" onmouseup="revealRole(false)" ontouchstart="revealRole(true)" ontouchend="revealRole(false)"
                        class="absolute inset-0 w-full h-full flex items-center justify-center bg-transparent font-bold text-sm text-indigo-300/50 cursor-pointer">
                    MANTENER PARA VER ROL
                </button>
            </div>
            <p id="juez-info" class="text-xs text-amber-400 italic hidden"></p>
            <button onclick="nextRole()" class="w-full py-4 bg-slate-700 hover:bg-slate-600 rounded-2xl font-bold transition-all">
                SIGUIENTE JUGADOR
            </button>
        </div>

        <!-- PANTALLA 3: Ciclo Principal (Noche/Amanecer) -->
        <div id="screen-main" class="hidden grid grid-cols-1 md:grid-cols-3 gap-6">
            <div class="md:col-span-2 space-y-6">
                <div class="glass p-8 rounded-3xl text-center space-y-4">
                    <h2 id="phase-title" class="text-3xl font-bold text-indigo-400 uppercase tracking-widest">La Noche</h2>
                    <div id="action-area" class="p-6 bg-slate-900/50 rounded-2xl min-h-[100px] flex items-center justify-center italic text-slate-300">
                        Los impostores están eligiendo a su víctima...
                    </div>
                    <div id="night-inputs" class="hidden space-y-4">
                        <!-- Inputs dinámicos para Doctor/Detective -->
                    </div>
                    <button id="btn-next-phase" onclick="advancePhase()" class="w-full py-4 bg-indigo-600 hover:bg-indigo-500 rounded-2xl font-bold">
                        CONTINUAR
                    </button>
                </div>
                
                <div class="glass p-6 rounded-3xl">
                    <h3 class="text-sm font-bold text-slate-500 uppercase mb-4">Registro del Juez</h3>
                    <div id="game-logs" class="h-48 overflow-y-auto custom-scroll space-y-2 text-sm text-slate-400"></div>
                </div>
            </div>
            
            <div class="glass p-6 rounded-3xl">
                <h3 class="text-sm font-bold text-slate-500 uppercase mb-4">Pueblo Vivo</h3>
                <div id="player-list" class="grid grid-cols-2 gap-2"></div>
            </div>
        </div>

        <!-- PANTALLA 4: Fase de Debate (Corregida) -->
        <div id="screen-debate" class="hidden space-y-6">
            <div class="text-center glass p-6 rounded-3xl border-amber-500/30">
                <h2 class="text-3xl font-black text-amber-400 tracking-tighter">FASE DE DEBATE</h2>
                <p class="text-slate-400 text-sm">Registren sus sospechas antes de la votación final</p>
            </div>
            
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div class="glass p-6 rounded-3xl space-y-4">
                    <h3 class="font-bold text-xs uppercase text-indigo-400">Publicar Acusación</h3>
                    <select id="debate-author" class="w-full bg-slate-800 border border-slate-700 rounded-xl p-3 outline-none focus:ring-2 ring-indigo-500 text-sm">
                        <option value="">¿Quién eres tú?</option>
                    </select>
                    <select id="debate-target" class="w-full bg-slate-800 border border-slate-700 rounded-xl p-3 outline-none focus:ring-2 ring-indigo-500 text-sm">
                        <option value="">¿A quién sospechas?</option>
                    </select>
                    <textarea id="debate-reason" placeholder="Escribe tu base de sospecha o prueba..." class="w-full h-24 bg-slate-800 border border-slate-700 rounded-xl p-3 outline-none focus:ring-2 ring-indigo-500 resize-none text-sm"></textarea>
                    <button onclick="postSuspicion()" class="w-full py-3 bg-red-600 hover:bg-red-500 rounded-xl font-bold transition-all text-sm">REGISTRAR EN EL MURO</button>
                </div>

                <div class="glass p-6 rounded-3xl flex flex-col">
                    <h3 class="font-bold text-xs uppercase text-slate-500 mb-4">Muro de Inteligencia</h3>
                    <div id="suspicion-wall" class="flex-grow h-64 overflow-y-auto custom-scroll space-y-3 pr-2">
                        <p class="text-slate-500 text-center italic text-sm mt-10">No hay acusaciones todavía...</p>
                    </div>
                </div>
            </div>
            
            <button onclick="showVotingScreen()" class="w-full py-4 bg-indigo-600 hover:bg-indigo-500 rounded-2xl font-bold text-xl shadow-lg">PROCEDER A VOTACIÓN</button>
        </div>

        <!-- PANTALLA 5: Votación -->
        <div id="screen-voting" class="hidden glass p-8 rounded-3xl text-center space-y-6">
            <h2 class="text-3xl font-bold text-red-400 uppercase">Votación Popular</h2>
            <p class="text-slate-400">Es hora de elegir a alguien para ser ejecutado.</p>
            <div id="voting-area" class="grid grid-cols-2 md:grid-cols-3 gap-3"></div>
            <div id="voting-confirm" class="hidden p-4 bg-slate-900 rounded-2xl border border-indigo-500/30">
                <p id="vote-info" class="text-lg font-bold"></p>
                <button onclick="confirmVote()" class="mt-4 px-8 py-2 bg-indigo-600 rounded-xl font-bold">CONFIRMAR VOTO</button>
            </div>
        </div>

    </div>

    <script>
        let state = {
            players: [],
            roles: {},
            alive: {},
            impostores: [],
            doctor: '',
            detective: '',
            juez: '',
            currentRevealIndex: 0,
            phase: 'setup', // setup, roles, night, sunrise, debate, voting
            logs: [],
            suspicions: [],
            currentVoterIndex: 0,
            votes: {},
            nightData: {}
        };

        const screens = ['setup', 'roles', 'main', 'debate', 'voting'];

        function showScreen(id) {
            screens.forEach(s => {
                const el = document.getElementById(`screen-${s}`);
                if(el) el.classList.add('hidden');
            });
            const target = document.getElementById(`screen-${id}`);
            if(target) target.classList.remove('hidden');
        }

        function startGame() {
            const count = parseInt(document.getElementById('input-players').value);
            const numPlayers = Math.max(10, Math.min(15, count));
            
            state.players = Array.from({length: numPlayers}, (_, i) => `Jugador_${i + 1}`);
            state.alive = {};
            state.players.forEach(p => state.alive[p] = true);

            // Asignación de Roles
            let pool = [...state.players];
            const numImpostores = numPlayers >= 13 ? 3 : 2;
            
            state.impostores = [];
            for(let i=0; i<numImpostores; i++) {
                let p = pool.splice(Math.floor(Math.random()*pool.length), 1)[0];
                state.roles[p] = "Impostor";
                state.impostores.push(p);
            }

            state.doctor = pool.splice(Math.floor(Math.random()*pool.length), 1)[0];
            state.roles[state.doctor] = "Doctor";

            state.detective = pool.splice(Math.floor(Math.random()*pool.length), 1)[0];
            state.roles[state.detective] = "Detective";

            state.juez = pool.splice(Math.floor(Math.random()*pool.length), 1)[0];
            state.roles[state.juez] = "Juez";

            pool.forEach(p => state.roles[p] = "Ciudadano");

            state.currentRevealIndex = 0;
            updateRoleScreen();
            showScreen('roles');
        }

        function updateRoleScreen() {
            const p = state.players[state.currentRevealIndex];
            document.getElementById('current-player-name').innerText = p;
            document.getElementById('role-name').innerText = state.roles[p];
            
            const juezInfo = document.getElementById('juez-info');
            if(state.roles[p] === "Juez") {
                juezInfo.innerText = `👁️ Sabes que los impostores son: ${state.impostores.join(', ')}`;
                juezInfo.classList.remove('hidden');
            } else {
                juezInfo.classList.add('hidden');
            }
        }

        function revealRole(show) {
            const box = document.getElementById('role-display-box');
            if(show) box.classList.add('role-visible');
            else box.classList.remove('role-visible');
        }

        function nextRole() {
            state.currentRevealIndex++;
            if(state.currentRevealIndex < state.players.length) {
                updateRoleScreen();
            } else {
                startNight();
            }
        }

        function startNight() {
            state.phase = 'night';
            addLog("La noche ha caído sobre el pueblo.");
            updateUI();
            showScreen('main');
            
            const vivosNoImpostores = state.players.filter(p => state.alive[p] && state.roles[p] !== 'Impostor');
            state.nightData.victim = vivosNoImpostores[Math.floor(Math.random() * vivosNoImpostores.length)];
            
            document.getElementById('action-area').innerHTML = `<p class="text-red-400 font-bold">¡Los asesinos han actuado! El Juez está procesando el reporte.</p>`;
        }

        function advancePhase() {
            if(state.phase === 'night') {
                showDebateScreen();
            } else if(state.phase === 'sunrise') {
                showDebateScreen();
            }
        }

        function showDebateScreen() {
            state.phase = 'debate';
            state.suspicions = [];
            showScreen('debate');
            
            const authorSel = document.getElementById('debate-author');
            const targetSel = document.getElementById('debate-target');
            
            [authorSel, targetSel].forEach(sel => {
                sel.innerHTML = `<option value="">${sel.id === 'debate-author' ? '¿Quién eres tú?' : '¿A quién sospechas?'}</option>`;
                state.players.forEach(p => {
                    if(state.alive[p]) sel.innerHTML += `<option value="${p}">${p}</option>`;
                });
            });
            renderSuspicions();
        }

        function postSuspicion() {
            const author = document.getElementById('debate-author').value;
            const target = document.getElementById('debate-target').value;
            const reason = document.getElementById('debate-reason').value;

            if(!author || !target || !reason) return;

            state.suspicions.push({ author, target, reason });
            renderSuspicions();
            
            document.getElementById('debate-reason').value = '';
            addLog(`Acusación: ${author} sospecha de ${target}`);
        }

        function renderSuspicions() {
            const wall = document.getElementById('suspicion-wall');
            wall.innerHTML = '';
            
            if(state.suspicions.length === 0) {
                wall.innerHTML = '<p class="text-slate-500 text-center italic text-sm mt-10">No hay acusaciones todavía...</p>';
                return;
            }

            [...state.suspicions].reverse().forEach(s => {
                wall.innerHTML += `
                    <div class="suspicion-card p-3 rounded-xl border border-red-500/20">
                        <div class="flex justify-between text-xs mb-1">
                            <span class="font-bold text-indigo-300">${s.author} acusa a:</span>
                            <span class="font-black text-red-400 uppercase">${s.target}</span>
                        </div>
                        <p class="text-sm text-slate-200 italic">"${s.reason}"</p>
                    </div>
                `;
            });
        }

        function showVotingScreen() {
            state.phase = 'voting';
            showScreen('voting');
            state.currentVoterIndex = 0;
            state.votes = {};
            prepareNextVote();
        }

        function prepareNextVote() {
            const voterList = state.players.filter(p => state.alive[p]);
            if(state.currentVoterIndex >= voterList.length) {
                processVotingResults();
                return;
            }

            const currentVoter = voterList[state.currentVoterIndex];
            const area = document.getElementById('voting-area');
            area.innerHTML = '';
            
            document.querySelector('#screen-voting h2').innerText = `Voto de: ${currentVoter}`;

            voterList.forEach(p => {
                area.innerHTML += `
                    <button onclick="selectVote('${p}')" class="p-3 bg-slate-800 hover:bg-indigo-600 rounded-xl text-sm transition-all">
                        ${p}
                    </button>
                `;
            });
        }

        let selectedCandidate = '';
        function selectVote(p) {
            selectedCandidate = p;
            document.getElementById('vote-info').innerText = `Vas a votar por: ${p}`;
            document.getElementById('voting-confirm').classList.remove('hidden');
        }

        function confirmVote() {
            const voterList = state.players.filter(p => state.alive[p]);
            const voter = voterList[state.currentVoterIndex];
            state.votes[selectedCandidate] = (state.votes[selectedCandidate] || 0) + 1;
            
            state.currentVoterIndex++;
            document.getElementById('voting-confirm').classList.add('hidden');
            prepareNextVote();
        }

        function processVotingResults() {
            const victim = state.nightData.victim;
            state.alive[victim] = false;
            addLog(`⚖️ Amanecer: ${victim} fue eliminado anoche.`);

            let maxVotes = -1;
            let expelled = '';
            for(let p in state.votes) {
                if(state.votes[p] > maxVotes) {
                    maxVotes = state.votes[p];
                    expelled = p;
                }
            }

            state.alive[expelled] = false;
            addLog(`🗳️ El pueblo ha ejecutado a ${expelled} (${state.roles[expelled]})`);
            
            checkWinConditions();
        }

        function checkWinConditions() {
            const impostoresVivos = state.impostores.filter(p => state.alive[p]).length;
            const buenosVivos = state.players.filter(p => state.alive[p] && state.roles[p] !== 'Impostor').length;

            if(impostoresVivos === 0) {
                gameOver("¡VICTORIA CIUDADANA! Los impostores han sido erradicados.");
            } else if(impostoresVivos >= buenosVivos) {
                gameOver("¡VICTORIA DE IMPOSTORES! El pueblo ha caído bajo su control.");
            } else {
                state.phase = 'night';
                showScreen('main');
                updateUI();
                startNight();
            }
        }

        function gameOver(msg) {
            document.body.innerHTML = `
                <div class="glass p-12 rounded-3xl text-center space-y-6 max-w-md">
                    <h1 class="text-4xl font-black text-white">FIN DEL JUEGO</h1>
                    <p class="text-xl text-indigo-300">${msg}</p>
                    <button onclick="location.reload()" class="w-full py-4 bg-indigo-600 rounded-2xl font-bold">REINTENTAR</button>
                </div>
            `;
        }

        function addLog(msg) {
            state.logs.push(msg);
            const logBox = document.getElementById('game-logs');
            if(logBox) {
                logBox.innerHTML = state.logs.map(l => `<p>• ${l}</p>`).reverse().join('');
            }
        }

        function updateUI() {
            const list = document.getElementById('player-list');
            if(!list) return;
            list.innerHTML = '';
            state.players.forEach(p => {
                const status = state.alive[p] ? 'bg-emerald-500/20 text-emerald-400' : 'bg-red-500/20 text-red-400 line-through opacity-50';
                list.innerHTML += `<div class="p-2 rounded-lg text-xs font-bold text-center ${status}">${p}</div>`;
            });
        }
    </script>
</body>
</html>
