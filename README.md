
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Desafío Sumas y Restas - MetaVidaDigital</title>
    <style>
        body { background: linear-gradient(135deg, #11998e, #38ef7d); color: white; font-family: 'Arial', sans-serif; text-align: center; margin: 0; padding: 20px; min-height: 100vh; }
        #display-score { font-size: 2.5rem; color: #fff; margin: 10px; font-weight: bold; text-shadow: 2px 2px 4px rgba(0,0,0,0.3); }
        #tablero { width: 95%; min-height: 220px; border: 6px dashed #fff; margin: 20px auto; display: flex; flex-wrap: wrap; gap: 15px; padding: 20px; background: rgba(0, 0, 0, 0.2); border-radius: 20px; align-items: center; justify-content: center; backdrop-filter: blur(5px); }
        #reserva { display: flex; flex-wrap: wrap; gap: 15px; justify-content: center; padding: 20px; }
        .ficha { width: 135px; height: 90px; display: flex; flex-direction: column; align-items: center; justify-content: center; font-weight: bold; border-radius: 14px; cursor: grab; font-size: 1.2rem; box-shadow: 0 6px rgba(0,0,0,0.3); color: white; user-select: none; }
        .ficha-linea { width: 85%; height: 2px; background-color: rgba(255, 255, 255, 0.5); margin: 5px 0; }
        .controles { margin: 20px; display: flex; justify-content: center; gap: 15px; }
        button { padding: 15px 30px; font-size: 1.5rem; cursor: pointer; border: none; border-radius: 12px; color: white; font-weight: bold; box-shadow: 0 4px rgba(0,0,0,0.3); }
        button:active { transform: translateY(4px); box-shadow: none; }
        #btn-iniciar { background: #ff4757; }
        #btn-reiniciar { background: #2f3542; }
        #mensaje-final { font-size: 2.2rem; color: #fff; margin: 15px auto; font-weight: bold; display: none; text-shadow: 2px 2px 8px rgba(0,0,0,0.5); }
        .btn-flotante { position: fixed; bottom: 25px; right: 25px; background: #ffffff; color: #11998e; padding: 15px 22px; font-size: 1.1rem; font-weight: bold; text-decoration: none; border-radius: 50px; box-shadow: 0 6px 15px rgba(0,0,0,0.3); display: flex; align-items: center; gap: 10px; z-index: 9999; }
    </style>
</head>
<body>

    <h1>🧮 ¡Desafío de Sumas y Restas! ⚡</h1>
    <div id="display-score">Puntaje: 0</div>
    
    <div class="controles">
        <button id="btn-iniciar" onclick="iniciarJuego()">Empezar Juego</button>
        <button id="btn-reiniciar" onclick="reiniciarJuego()" style="display: none;">Reiniciar</button>
    </div>

    <div id="mensaje-final"></div>
    <div id="tablero" ondrop="drop(event)" ondragover="allowDrop(event)"></div>
    <div id="reserva"></div>

    <a href="https://netlify.app" target="_blank" class="btn-flotante" onclick="visitarMateKen(event)">🤖 Ir a MateKen2</a>

    <script>
        let audioCtx = null;
        function playBeep(tipo) {
            if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            const osc = audioCtx.createOscillator(), gain = audioCtx.createGain(); osc.connect(gain); gain.connect(audioCtx.destination); const t = audioCtx.currentTime;
            if (tipo === 'click') { osc.frequency.setValueAtTime(550, t); gain.gain.setValueAtTime(0.08, t); gain.gain.exponentialRampToValueAtTime(0.01, t + 0.08); osc.start(t); osc.stop(t + 0.08); } 
            else if (tipo === 'acierto') { osc.frequency.setValueAtTime(440, t); osc.frequency.setValueAtTime(554, t + 0.08); osc.frequency.setValueAtTime(660, t + 0.16); gain.gain.setValueAtTime(0.12, t); gain.gain.exponentialRampToValueAtTime(0.01, t + 0.3); osc.start(t); osc.stop(t + 0.3); } 
            else if (tipo === 'error') { osc.type = 'sawtooth'; osc.frequency.setValueAtTime(160, t); gain.gain.setValueAtTime(0.15, t); gain.gain.exponentialRampToValueAtTime(0.01, t + 0.25); osc.start(t); osc.stop(t + 0.25); } 
            else if (tipo === 'start') { osc.frequency.setValueAtTime(523, t); osc.frequency.setValueAtTime(659, t + 0.1); gain.gain.setValueAtTime(0.1, t); gain.gain.exponentialRampToValueAtTime(0.01, t + 0.3); osc.start(t); osc.stop(t + 0.3); } 
            else if (tipo === 'teleport') { osc.type = 'triangle'; osc.frequency.setValueAtTime(250, t); osc.frequency.exponentialRampToValueAtTime(1300, t + 0.35); gain.gain.setValueAtTime(0.1, t); gain.gain.exponentialRampToValueAtTime(0.01, t + 0.35); osc.start(t); osc.stop(t + 0.35); }
        }

        const coloresFichas = ["#ff6b6b", "#4bc0c0", "#ff9f43", "#9b59b6", "#10ac84", "#2e86de", "#f53b57", "#5758bb", "#0be881", "#ef5777"];
        let score = 0, buscandoValor = 0, fichasPartida = [], juegoActivo = false;

        function iniciarJuego() { playBeep('start'); document.getElementById('btn-iniciar').style.display = 'none'; document.getElementById('btn-reiniciar').style.display = 'inline-block'; juegoActivo = true; generarEjercicioAleatorio(); }

        function generarEjercicioAleatorio() {
            document.getElementById('tablero').innerHTML = ""; document.getElementById('reserva').innerHTML = ""; document.getElementById('mensaje-final').style.display = 'none';
            score = 0; document.getElementById('display-score').innerText = "Puntaje: " + score; fichasPartida = [];
            
            // GARANTÍA DE NÚMEROS ÚNICOS: Creamos un conjunto de 11 números que jamás se repiten en la misma partida
            let cadenaValores = [];
            let semilla = Math.floor(Math.random() * 10) + 10; 
            for(let i = 0; i < 11; i++) {
                semilla += Math.floor(Math.random() * 5) + 3; // Siempre crece, garantizando que cada número sea único
                cadenaValores.push(semilla);
            }
            // Mezclamos un poco la secuencia interna para alternar sumas y restas aleatorias
            let eslabones = [...cadenaValores];

            for(let i = 0; i < 10; i++) {
                let valDeArriba = eslabones[i];
                let valDeAbajoDeseado = eslabones[i+1];
                let textoOperacion = "", icono = "";

                // Decisión aleatoria entre suma y resta usando valores matemáticos controlados e infalibles
                if (Math.random() > 0.5) {
                    textoOperacion = `${valDeArriba} + ${valDeAbajoDeseado - valDeArriba}`; icono = "➕";
                } else {
                    let desvio = Math.floor(Math.random() * 10) + 5;
                    textoOperacion = `${valDeAbajoDeseado + desvio} - ${desvio}`; icono = "➖";
                }

                fichasPartida.push({ id: "ficha_" + i, valBuscar: valDeArriba, numVisual: valDeArriba, opVisual: textoOperacion, proximoValor: valDeAbajoDeseado, col: coloresFichas[i], ico: icono });
            }

            buscandoValor = fichasPartida[0].valBuscar; // Corregida la referencia del índice inicial
            const fInicio = document.createElement('div'); fInicio.className = 'ficha'; fInicio.style.backgroundColor = '#57606f'; fInicio.style.cursor = 'default';
            fInicio.innerHTML = `<div>START 🏁</div><div class="ficha-linea"></div><div>Busca: ${buscandoValor}</div>`;
            document.getElementById('tablero').appendChild(fInicio);

            let desordenadas = [...fichasPartida].sort(() => Math.random() - 0.5);
            desordenadas.forEach((f) => {
                const div = document.createElement('div'); div.className = 'ficha'; div.id = f.id; div.draggable = true; div.style.backgroundColor = f.col; 
                div.innerHTML = `<div>${f.numVisual}</div><div class="ficha-linea"></div><div>${f.opVisual} <span style="font-size:0.8rem;opacity:0.8;">${f.ico}</span></div>`;
                div.ondragstart = (ev) => { if(!juegoActivo) return; ev.dataTransfer.setData("text", ev.target.id); playBeep('click'); };
                document.getElementById('reserva').appendChild(div);
            });
        }

        function drop(ev) {
            ev.preventDefault(); if (!juegoActivo) return; const idData = ev.dataTransfer.getData("text"); const el = document.getElementById(idData); if (!el) return;
            const ficha = fichasPartida.find(f => f.id === idData);
            if (ficha && ficha.valBuscar === buscandoValor) {
                document.getElementById('tablero').appendChild(el); el.draggable = false; el.style.cursor = 'default'; buscandoValor = ficha.proximoValor; 
                score += 10; document.getElementById('display-score').innerText = "Puntaje: " + score; playBeep('acierto');
                if (document.getElementById('reserva').children.length === 0) finalizarJuego();
            } else { playBeep('error'); }
        }

        function allowDrop(ev) { ev.preventDefault(); }
        function finalizarJuego() { juegoActivo = false; const msg = document.getElementById('mensaje-final'); msg.innerText = `🏆 ¡Excelente! Cadena de 10 completada con éxito: ${score} pts ✨`; msg.style.display = 'block'; playBeep('acierto'); }
        function reiniciarJuego() { playBeep('start'); juegoActivo = true; generarEjercicioAleatorio(); }
        function visitarMateKen(ev) { ev.preventDefault(); playBeep('teleport'); setTimeout(() => { window.open("https://netlify.app", "_blank"); }, 350); }
    </script>
</body>
</html>

