# Poke-Idle-World-Auto-Catch

---

## ⚡ Auto-clicker de "Throw" para Poke Idle World
Este script automatiza el lanzamiento de pokéballs haciendo clic en el botón "Throw" de manera interna.

¿La ventaja? Se ejecuta en segundo plano. Puedes minimizar el navegador, estar jugando a otra cosa o trabajando en tu PC, y el script seguirá capturando sin mover tu ratón físico ni quitarte el foco de la pantalla.
---

## 🛠️ Instrucciones de Instalación para PC:
Instala un gestor de scripts en tu navegador (Opera, Chrome, Brave o Edge):

Descarga e instala la extensión  [![Descargar Tampermonkey](https://img.shields.io/badge/Descargar-Tampermonkey-black?style=for-the-badge&logo=tampermonkey&logoColor=white)](https://www.tampermonkey.net/)

Crea el script:

Haz clic en el icono de Tampermonkey (arriba a la derecha en tu navegador) y selecciona "Crear un nuevo script..." (o "Agregar nuevo script").

Pega el código:

Borra todo el texto que aparezca por defecto en el editor.

Copia y pega el código que tienes abajo.

---
```javascript

// ==UserScript==
// @name         Throw + JustPokedex (v4.5 ENGLISH UI) - Poke Idle World
// @namespace    poke-idle-world-tools
// @version      4.9
// @description  Auto-thrower + JustPokedex with English UI, multi-language tooltip parsing. Now includes the official Quality/Power formulas (rarity tiers, capture-odds bands, exact Power calc).
// @author       PollitoScripts
// @match        *://poke.idleworld.online/*
// @grant        none
// @run-at       document-idle
// ==/UserScript==

(function () {
    'use strict';

    // =========================================================================
    // 1. MAIN SECTION: AUTO-THROW (PollitoScripts v3.2) - AUTOCATCH
    // =========================================================================

    function cargarConfig(clave, valorPorDefecto) {
        try {
            const guardado = localStorage.getItem(clave);
            if (guardado) return JSON.parse(guardado);
        } catch (e) { }
        return valorPorDefecto;
    }

    function guardarConfig(clave, valor) {
        localStorage.setItem(clave, JSON.stringify(valor));
    }

    let prioridadesNormal = cargarConfig("pollito_normal", ["Ultra Ball", "Super Ball", "Great Ball", "Poke Ball"]);
    let prioridadesShiny = cargarConfig("pollito_shiny", ["Idle Ball", "Ultra Ball", "Super Ball", "Great Ball", "Poke Ball"]);

    function obtenerTiempoAleatorio(min, max) {
        return Math.floor(Math.random() * (max - min + 1) + min);
    }

    async function simularClicReal(elemento) {
        if (!elemento) return;
        const opciones = { bubbles: true, cancelable: true, view: window };
        elemento.dispatchEvent(new MouseEvent('mousedown', opciones));
        await new Promise(resolve => setTimeout(resolve, obtenerTiempoAleatorio(10, 30)));
        elemento.dispatchEvent(new MouseEvent('mouseup', opciones));
        elemento.dispatchEvent(new MouseEvent('click', opciones));
    }

    function buscarBotonThrow() {
        const elementos = document.querySelectorAll('button, div');
        for (let elemento of elementos) {
            if ((elemento.textContent || '').trim().toLowerCase() === "throw" && elemento.offsetWidth > 0) {
                return elemento;
            }
        }
        return null;
    }

    function esShinyActivo(botonThrow) {
        if (!botonThrow) return false;
        const cajaCaptura = botonThrow.closest('.capture-box, #capture, [class*="capture" i]')
            || botonThrow.parentElement.parentElement.parentElement;
        if (cajaCaptura) {
            if ((cajaCaptura.textContent || '').includes('✨')) return true;
        }
        return false;
    }

    function buscarYFuego() {
        const botonThrowGenerico = buscarBotonThrow();

        if (botonThrowGenerico) {
            let bolaAEquipar = null;
            const esShiny = esShinyActivo(botonThrowGenerico);
            const listaPrioridades = esShiny ? prioridadesShiny : prioridadesNormal;

            for (let nombreBall of listaPrioridades) {
                const selector = `button[title="${nombreBall}" i], button[title="${nombreBall.replace('Poke', 'Poké')}" i]`;
                const botonBall = document.querySelector(selector);

                if (botonBall && !botonBall.disabled) {
                    if (!botonBall.classList.contains('on')) {
                        bolaAEquipar = botonBall;
                    }
                    break;
                }
            }

            const tiempoReaccion = obtenerTiempoAleatorio(250, 600);

            setTimeout(async () => {
                if (bolaAEquipar) {
                    await simularClicReal(bolaAEquipar);
                    setTimeout(async () => {
                        const botonThrowConfirmar = buscarBotonThrow();
                        if (botonThrowConfirmar) await simularClicReal(botonThrowConfirmar);
                    }, obtenerTiempoAleatorio(100, 200));
                } else {
                    await simularClicReal(botonThrowGenerico);
                }
                buclePrincipal(obtenerTiempoAleatorio(900, 1400));
            }, tiempoReaccion);

        } else {
            buclePrincipal(obtenerTiempoAleatorio(800, 1500));
        }
    }

    function buclePrincipal(retraso) {
        setTimeout(buscarYFuego, retraso);
    }

    function crearInterfazThrow() {
        const estilos = document.createElement('style');
        estilos.innerHTML = `
            #pollito-btn-config { position: fixed; bottom: 20px; right: 20px; z-index: 10000; background: #34495e; color: white; border: 2px solid #f1c40f; padding: 10px; border-radius: 50%; cursor: pointer; font-size: 20px; box-shadow: 0px 4px 6px rgba(0,0,0,0.3); transition: transform 0.2s; }
            #pollito-btn-config:hover { transform: scale(1.1); }
            #pollito-panel { position: fixed; bottom: 80px; right: 20px; z-index: 10000; background: #2c3e50; color: white; border: 2px solid #f1c40f; padding: 15px; border-radius: 8px; width: 280px; font-family: Arial, sans-serif; box-shadow: 0px 4px 15px rgba(0,0,0,0.5); display: none; }
            .pollito-seccion { margin-bottom: 15px; }
            .pollito-titulo { font-size: 14px; font-weight: bold; color: #f1c40f; margin-bottom: 8px; border-bottom: 1px solid #7f8c8d; padding-bottom: 3px; }
            .pollito-item { display: flex; justify-content: space-between; align-items: center; background: #34495e; padding: 5px 8px; margin-bottom: 4px; border-radius: 4px; font-size: 12px; }
            .pollito-botones button { background: #e67e22; color: white; border: none; padding: 2px 6px; margin-left: 2px; border-radius: 3px; cursor: pointer; font-weight: bold; }
            .pollito-botones button:hover { background: #d35400; }
            #pollito-guardar { width: 100%; background: #27ae60; color: white; border: none; padding: 8px; border-radius: 4px; cursor: pointer; font-weight: bold; font-size: 13px; margin-top: 5px; }
            #pollito-guardar:hover { background: #219653; }
        `;
        document.head.appendChild(estilos);

        const botonConfig = document.createElement('div');
        botonConfig.id = 'pollito-btn-config';
        botonConfig.innerHTML = '⚙️';
        document.body.appendChild(botonConfig);

        const panel = document.createElement('div');
        panel.id = 'pollito-panel';
        panel.innerHTML = `
            <div style="font-weight:bold; text-align:center; margin-bottom:10px; font-size:15px; color:#f1c40f;">PollitoScripts v4.5</div>
            <div class="pollito-seccion">
                <div class="pollito-titulo">Normal Priority</div>
                <div id="lista-normal"></div>
            </div>
            <div class="pollito-seccion">
                <div class="pollito-titulo">Shiny Priority ✨</div>
                <div id="lista-shiny"></div>
            </div>
            <button id="pollito-guardar">Save Preferences</button>
        `;
        document.body.appendChild(panel);

        botonConfig.onclick = () => {
            panel.style.display = panel.style.display === 'none' || panel.style.display === '' ? 'block' : 'none';
        };

        function renderizarListas() {
            const contenedorNormal = document.getElementById('lista-normal');
            const contenedorShiny = document.getElementById('lista-shiny');
            contenedorNormal.innerHTML = '';
            contenedorShiny.innerHTML = '';

            prioridadesNormal.forEach((ball, index) => {
                const item = document.createElement('div');
                item.className = 'pollito-item';
                item.innerHTML = `
                    <span>${index + 1}. ${ball}</span>
                    <div class="pollito-botones">
                        ${index > 0 ? `<button class="btn-subir-n" data-idx="${index}">🔼</button>` : ''}
                        ${index < prioridadesNormal.length - 1 ? `<button class="btn-bajar-n" data-idx="${index}">🔽</button>` : ''}
                    </div>
                `;
                contenedorNormal.appendChild(item);
            });

            prioridadesShiny.forEach((ball, index) => {
                const item = document.createElement('div');
                item.className = 'pollito-item';
                item.innerHTML = `
                    <span>${index + 1}. ${ball}</span>
                    <div class="pollito-botones">
                        ${index > 0 ? `<button class="btn-subir-s" data-idx="${index}">🔼</button>` : ''}
                        ${index < prioridadesShiny.length - 1 ? `<button class="btn-bajar-s" data-idx="${index}">🔽</button>` : ''}
                    </div>
                `;
                contenedorShiny.appendChild(item);
            });

            document.querySelectorAll('.btn-subir-n').forEach(btn => btn.onclick = () => moverItem(prioridadesNormal, parseInt(btn.dataset.idx), -1));
            document.querySelectorAll('.btn-bajar-n').forEach(btn => btn.onclick = () => moverItem(prioridadesNormal, parseInt(btn.dataset.idx), 1));
            document.querySelectorAll('.btn-subir-s').forEach(btn => btn.onclick = () => moverItem(prioridadesShiny, parseInt(btn.dataset.idx), -1));
            document.querySelectorAll('.btn-bajar-s').forEach(btn => btn.onclick = () => moverItem(prioridadesShiny, parseInt(btn.dataset.idx), 1));
        }

        function moverItem(arr, index, direccion) {
            const temp = arr[index];
            arr[index] = arr[index + direccion];
            arr[index + direccion] = temp;
            renderizarListas();
        }

        document.getElementById('pollito-guardar').onclick = () => {
            guardarConfig("pollito_normal", prioridadesNormal);
            guardarConfig("pollito_shiny", prioridadesShiny);
            alert("PollitoScripts preferences saved successfully!");
            panel.style.display = 'none';
        };

        renderizarListas();
    }


    // =========================================================================
    // 2. JUSTPOKEDEX SECTION (EXACT IV FORMULA + MULTI-LANGUAGE READER)
    // =========================================================================

    const CONFIG_POKEDEX = {
        tooltipSelector: ".inv-tip",
        panelId: "pokemon-reader-panel",
        storageKey: "pokemon-reader-panel-state",
        maxIVTotal: 192,
        calidadMaxima: 1.80,
        // Por debajo de este nivel, el redondeo de stats visibles se amplifica
        // al invertir la fórmula de IVs y el resultado deja de ser fiable.
        nivelMinimoIvConfiable: 5
    };

    let ultimoPokemon = null;
    let mostrarPanelMoves = false;
    const danoPorGolpe = new Map();
    let ultimoGolpeUsado = null;
    let creaturesData = [];
    let creaturesMapByName = new Map();

    // --- EXACT IV FORMULA ---
    function calcularIvIndividual(statKey, statVal, baseStat, nivel, qualidade) {
        if (!statVal || !baseStat || !nivel || !qualidade) return 0;
        const exp = (statKey === 'hp' || statKey === 'vel' || statKey === 'speed') ? 0.95 : 0.80;
        const mult = (nivel / 100) * Math.pow(qualidade, exp);
        if (mult === 0) return 0;
        const x = Math.round(((statVal / mult) - baseStat) / 2);
        // Confirmado: el "Growth" (lo que este script llama IV) va de 1 a 32 por stat, nunca 0.
        return Math.max(1, Math.min(32, x));
    }

    function calcularIvsExactos(pokemon) {
        if (!pokemon || !pokemon.nome) return null;
        const c = creaturesMapByName.get(pokemon.nome.toLowerCase().trim());
        if (!c) return null;

        const baseHp = c.baseHp ?? c.hp ?? c.stats?.hp;
        const baseAtk = c.baseAtk ?? c.atk ?? c.stats?.atk;
        const baseDef = c.baseDef ?? c.def ?? c.stats?.def;
        const baseSpA = c.baseSpAtk ?? c.spAtk ?? c.spa ?? c.stats?.spAtk;
        const baseSpD = c.baseSpDef ?? c.spDef ?? c.spd ?? c.stats?.spDef;
        const baseVel = c.baseSpeed ?? c.speed ?? c.vel ?? c.stats?.speed;

        const q = pokemon.multiplicadorQualidade || 1;
        const lvl = pokemon.nivel || 1;

        const ivHp = calcularIvIndividual('hp', pokemon.hpNum, baseHp, lvl, q);
        const ivAtk = calcularIvIndividual('atk', pokemon.atkNum, baseAtk, lvl, q);
        const ivDef = calcularIvIndividual('def', pokemon.defNum, baseDef, lvl, q);
        const ivSpA = calcularIvIndividual('spa', pokemon.spaNum, baseSpA, lvl, q);
        const ivSpD = calcularIvIndividual('spd', pokemon.spdNum, baseSpD, lvl, q);
        const ivVel = calcularIvIndividual('vel', pokemon.velNum, baseVel, lvl, q);

        const ivTotal = ivHp + ivAtk + ivDef + ivSpA + ivSpD + ivVel;

        return {
            hp: ivHp, atk: ivAtk, def: ivDef, spa: ivSpA, spd: ivSpD, vel: ivVel,
            total: ivTotal
        };
    }

    // =========================================================================
    // 2.1 OFFICIAL QUALITY & POWER FORMULAS (community datamine, confirmed)
    //
    //     Power = (HP + Atk + Def + SpAtk + SpDef + Spd) × Quality
    //     stat  = round( (base + 2×growth) × level/100 × Quality^exp )
    //
    //     "Growth" (called "IV" throughout this script) ranges 1..32 per stat.
    //     Growth and Quality are both fixed at capture — evolving does NOT
    //     re-roll either of them.
    //     Quality appears twice in the maths (once inside every stat, and again
    //     in Power), which is why it swings Power far more than any single IV.
    // =========================================================================

    const CONFIG_CALIDAD = {
        // Etiqueta de rareza según el multiplicador de Quality del Pokémon
        ETIQUETAS: [
            { min: 4.0, max: Infinity, label: "Divine" },
            { min: 3.0, max: 4.0, label: "Ancient" },
            { min: 2.0, max: 3.0, label: "Mythic" },
            { min: 1.7, max: 2.0, label: "Legendary" },
            { min: 1.5, max: 1.7, label: "Epic" },
            { min: 1.3, max: 1.5, label: "Rare" },
            { min: 1.1, max: 1.3, label: "Uncommon" },
            { min: 1.0, max: 1.1, label: "Common" },
            { min: -Infinity, max: 1.0, label: "Weak" }
        ],
        // Probabilidad de obtener cada banda de Quality en una captura salvaje normal
        // (tope 1.8; el hueco 1.6-1.7 es intencional: <1% de los Pokémon llegan a 1.7+,
        // y solo un 0.28846% nace con Quality perfecta, exactamente 1.800).
        BANDAS_CAPTURA: [
            { min: 0.8, max: 0.9, chance: 5 },
            { min: 0.9, max: 1.0, chance: 5 },
            { min: 1.0, max: 1.1, chance: 34.03846 },
            { min: 1.1, max: 1.2, chance: 20 },
            { min: 1.2, max: 1.3, chance: 10 },
            { min: 1.3, max: 1.4, chance: 10 },
            { min: 1.4, max: 1.5, chance: 10 },
            { min: 1.5, max: 1.6, chance: 5 },
            { min: 1.6, max: 1.7, chance: 0 },
            { min: 1.7, max: 1.799, chance: 0.67308 },
            { min: 1.799, max: 1.800001, chance: 0.28846 } // Quality perfecta (1.800)
        ],
        // Los tramos Mythic/Ancient/Divine (Quality 2.0+) NO salen de capturas
        // salvajes normales (tope 1.8) — solo de shinies o Pokémon criados.
        QUALITY_MAXIMA_SALVAJE: 1.8
    };

    function obtenerEtiquetaCalidad(qualidade) {
        if (!Number.isFinite(qualidade)) return null;
        const fila = CONFIG_CALIDAD.ETIQUETAS.find(r => qualidade >= r.min && qualidade < r.max);
        return fila ? fila.label : null;
    }

    function obtenerBandaCaptura(qualidade) {
        if (!Number.isFinite(qualidade) || qualidade > CONFIG_CALIDAD.QUALITY_MAXIMA_SALVAJE) return null;
        const fila = CONFIG_CALIDAD.BANDAS_CAPTURA.find(b => qualidade >= b.min && qualidade < b.max);
        return fila ? fila.chance : null;
    }

    // Power = (HP + Atk + Def + SpAtk + SpDef + Spd) × Quality
    // Se usa como respaldo cuando el tooltip del juego no muestra "Power" directamente,
    // y también sirve para validar el valor que sí muestra el juego.
    function calcularPoderExacto(pokemon) {
        if (!pokemon) return null;
        const stats = [pokemon.hpNum, pokemon.atkNum, pokemon.defNum, pokemon.spaNum, pokemon.spdNum, pokemon.velNum]
            .map(Number);
        if (stats.some(s => !Number.isFinite(s) || s <= 0)) return null;
        const q = pokemon.multiplicadorQualidade;
        if (!q) return null;
        return Math.round(stats.reduce((a, b) => a + b, 0) * q);
    }

    function guardarEstadoPokedex(panel) {
        try {
            const rect = panel.getBoundingClientRect();
            const estado = {
                left: rect.left,
                top: rect.top,
                minimized: panel.classList.contains("minimized"),
                mostrarMoves: mostrarPanelMoves
            };
            localStorage.setItem(CONFIG_POKEDEX.storageKey, JSON.stringify(estado));
        } catch (e) { }
    }

    function restaurarEstadoPokedex(panel) {
        try {
            const salvo = localStorage.getItem(CONFIG_POKEDEX.storageKey);
            if (!salvo) return;
            const estado = JSON.parse(salvo);
            if (Number.isFinite(estado.left) && Number.isFinite(estado.top)) {
                panel.style.left = `${Math.max(0, Math.min(window.innerWidth - 290, estado.left))}px`;
                panel.style.top = `${Math.max(0, Math.min(window.innerHeight - 100, estado.top))}px`;
            }
            if (estado.minimized) {
                panel.classList.add("minimized");
                const btnMin = document.getElementById('minimize-pokedex');
                if (btnMin) btnMin.textContent = '+';
            }
            if (estado.mostrarMoves) {
                mostrarPanelMoves = true;
                const movesPanel = document.getElementById('moves-panel');
                if (movesPanel) movesPanel.style.display = 'block';
            }
        } catch (e) { }
    }

    function actualizarPosicionMoves() {
        const mainPanel = document.getElementById(CONFIG_POKEDEX.panelId);
        const movesPanel = document.getElementById('moves-panel');
        if (!mainPanel || !movesPanel || movesPanel.style.display === 'none') return;
        const rect = mainPanel.getBoundingClientRect();
        let left = rect.right + 10;
        if (left + 260 > window.innerWidth) {
            left = rect.left - 270;
        }
        movesPanel.style.left = `${Math.max(10, left)}px`;
        movesPanel.style.top = `${rect.top}px`;
    }

    function extraerDanoDeObjeto(obj, depth = 0, resultados = []) {
        if (!obj || typeof obj !== "object" || depth > 6) return resultados;
        const nombreGolpe = obj.moveName || obj.attackName || obj.spellName ||
            (typeof obj.move === "string" ? obj.move : obj.move?.name) ||
            (typeof obj.attack === "string" ? obj.attack : null) ||
            (typeof obj.skill === "string" ? obj.skill : obj.skill?.name);
        const dano = Number(obj.damage ?? obj.dmg ?? obj.dano ?? obj.amount);
        if (typeof nombreGolpe === "string" && nombreGolpe.trim() && Number.isFinite(dano)) {
            resultados.push({
                name: nombreGolpe.trim(),
                dmg: dano,
                type: typeof obj.type === "string" ? obj.type : null,
                eff: Number.isFinite(Number(obj.eff)) ? Number(obj.eff) : null
            });
            return resultados;
        }
        for (const val of Object.values(obj)) {
            extraerDanoDeObjeto(val, depth + 1, resultados);
        }
        return resultados;
    }

    function registrarDanos(datos) {
        const golpes = extraerDanoDeObjeto(datos);
        if (golpes.length === 0) return;
        for (const g of golpes) {
            const clave = g.name.toLowerCase();
            ultimoGolpeUsado = clave;
            const actual = danoPorGolpe.get(clave) || { count: 0, total: 0 };
            danoPorGolpe.set(clave, {
                name: g.name,
                lastDmg: g.dmg,
                total: actual.total + g.dmg,
                count: actual.count + 1,
                type: g.type || actual.type,
                eff: g.eff
            });
        }
        actualizarPanelMoves();
    }

    try {
        const originalWS = window.WebSocket;
        window.WebSocket = new Proxy(originalWS, {
            construct(target, args) {
                const ws = new target(...args);
                ws.addEventListener("message", (event) => {
                    try {
                        if (typeof event.data === "string") {
                            const parsed = JSON.parse(event.data);
                            registrarDanos(parsed);
                        }
                    } catch (e) { }
                });
                return ws;
            }
        });
        window.WebSocket.prototype = originalWS.prototype;
    } catch (e) { }

    const TYPE_SYSTEM = {
        CHART: {
            normal: { rock: 0.5, ghost: 0, steel: 0.5 },
            fire: { fire: 0.5, water: 0.5, grass: 2, ice: 2, bug: 2, rock: 0.5, dragon: 0.5, steel: 2 },
            water: { fire: 2, water: 0.5, grass: 0.5, ground: 2, rock: 2, dragon: 0.5 },
            electric: { water: 2, electric: 0.5, grass: 0.5, ground: 0, flying: 2, dragon: 0.5 },
            grass: { fire: 0.5, water: 2, grass: 0.5, poison: 0.5, ground: 2, flying: 0.5, bug: 0.5, rock: 2, dragon: 0.5, steel: 0.5 },
            ice: { fire: 0.5, water: 0.5, grass: 2, ice: 0.5, ground: 2, flying: 2, dragon: 2, steel: 0.5 },
            fighting: { normal: 2, ice: 2, poison: 0.5, flying: 0.5, psychic: 0.5, bug: 0.5, rock: 2, ghost: 0, dark: 2, steel: 2, fairy: 0.5 },
            poison: { grass: 2, poison: 0.5, ground: 0.5, rock: 0.5, ghost: 0.5, steel: 0, fairy: 2 },
            ground: { fire: 2, electric: 2, grass: 0.5, poison: 2, flying: 0, bug: 0.5, rock: 2, steel: 2 },
            flying: { electric: 0.5, grass: 2, fighting: 2, bug: 2, rock: 0.5, steel: 0.5 },
            psychic: { fighting: 2, poison: 2, psychic: 0.5, dark: 0, steel: 0.5 },
            bug: { fire: 0.5, grass: 2, fighting: 0.5, poison: 0.5, flying: 0.5, psychic: 2, ghost: 0.5, dark: 2, steel: 0.5, fairy: 0.5 },
            rock: { fire: 2, ice: 2, fighting: 0.5, ground: 0.5, flying: 2, bug: 2, steel: 0.5 },
            ghost: { normal: 0, psychic: 2, ghost: 2, dark: 0.5 },
            dragon: { dragon: 2, steel: 0.5, fairy: 0 },
            dark: { fighting: 0.5, psychic: 2, ghost: 2, dark: 0.5, fairy: 0.5 },
            steel: { fire: 0.5, water: 0.5, electric: 0.5, ice: 2, rock: 2, steel: 0.5, fairy: 2 },
            fairy: { fire: 0.5, fighting: 2, poison: 0.5, dragon: 2, dark: 2, steel: 0.5 }
        },
        COLORS: {
            normal: "#a8a878", fire: "#f08030", water: "#6890f0", electric: "#f8d030", grass: "#78c850",
            ice: "#98d8d8", fighting: "#c03028", poison: "#a040a0", ground: "#e0c068", flying: "#a890f0",
            psychic: "#f85888", bug: "#a8b820", rock: "#b8a038", ghost: "#705898", dragon: "#7038f8",
            dark: "#705848", steel: "#b8b8d0", fairy: "#ee99ac"
        },
        // Display labels (now English)
        TRADUCOES: {
            normal: "Normal", fire: "Fire", water: "Water", electric: "Electric", grass: "Grass",
            ice: "Ice", fighting: "Fighting", poison: "Poison", ground: "Ground", flying: "Flying",
            psychic: "Psychic", bug: "Bug", rock: "Rock", ghost: "Ghost", dragon: "Dragon",
            dark: "Dark", steel: "Steel", fairy: "Fairy"
        },
        // Reverse lookup for parsing tooltip text in multiple languages (kept as-is, input parsing only)
        TRADUCOES_INVERSAS: {
            "normal": "normal", "fogo": "fire", "fuego": "fire", "fire": "fire", "agua": "water", "water": "water", "eletrico": "electric", "electrico": "electric", "electric": "electric",
            "planta": "grass", "grass": "grass", "gelo": "ice", "hielo": "ice", "ice": "ice", "lutador": "fighting", "lucha": "fighting", "fighting": "fighting", "veneno": "poison", "poison": "poison",
            "terra": "ground", "tierra": "ground", "voador": "flying", "volador": "flying", "flying": "flying", "psiquico": "psychic", "psychic": "psychic",
            "inseto": "bug", "bicho": "bug", "bug": "bug", "pedra": "rock", "roca": "rock", "fantasma": "ghost", "ghost": "ghost", "dragao": "dragon",
            "dragon": "dragon", "sombrio": "dark", "siniestro": "dark", "dark": "dark", "aco": "steel", "acero": "steel", "steel": "steel", "fada": "fairy", "hada": "fairy", "fairy": "fairy",
            "rock": "rock", "ground": "ground"
        }
    };

    function obtenerClaveTipo(tipoPt) {
        if (!tipoPt) return null;
        const pt = tipoPt.toLowerCase().trim().normalize("NFD").replace(/[\u0300-\u036f]/g, "");
        if (TYPE_SYSTEM.COLORS[pt]) return pt;
        return TYPE_SYSTEM.TRADUCOES_INVERSAS[pt] || null;
    }

    function typeBadgeHtml(tipoEng) {
        const cor = TYPE_SYSTEM.COLORS[tipoEng] || "#888";
        const nomeEn = TYPE_SYSTEM.TRADUCOES[tipoEng] || tipoEng;
        return `<span style="background:${cor}; color:#fff; padding:2px 6px; border-radius:4px; font-size:10px; font-weight:bold; display:inline-block; margin:1px;">${escapeHtml(nomeEn)}</span>`;
    }

    function obtenerEfectividadHtml(pokemon) {
        if (!pokemon || !pokemon.tipos) return "";
        const tiposEng = pokemon.tipos.map(t => obtenerClaveTipo(t)).filter(Boolean);
        if (tiposEng.length === 0) return "";

        const todosTipos = Object.keys(TYPE_SYSTEM.CHART);
        const toma4x = [], toma2x = [], imune = [];

        for (const atk of todosTipos) {
            const multiplicador = tiposEng.reduce((acc, def) => acc * (TYPE_SYSTEM.CHART[atk][def] ?? 1), 1);
            if (multiplicador === 2) toma2x.push(atk);
            else if (multiplicador === 4) toma4x.push(atk);
            else if (multiplicador === 0) imune.push(atk);
        }

        const crearLinha = (icono, label, lista, color) => {
            if (lista.length === 0) return "";
            return `
                <div style="display:flex; align-items:center; gap:6px; margin:4px 0;">
                    <span style="color:${color}; font-weight:bold; width:70px; font-size:10px; flex-shrink:0;">${icono} ${label}</span>
                    <div style="display:flex; flex-wrap:wrap; gap:2px;">${lista.map(t => typeBadgeHtml(t)).join("")}</div>
                </div>
            `;
        };

        return `
            <div style="margin-top:8px; padding:8px; background:rgba(0,0,0,0.25); border-radius:6px; font-size:11px;">
                <div style="font-weight:bold; color:#f1c40f; margin-bottom:4px; font-size:10px; text-transform:uppercase;">📊 Effectiveness</div>
                ${crearLinha("🛡️", "Takes 4x", toma4x, "#ff6b6b")}
                ${crearLinha("🛡️", "Takes 2x", toma2x, "#ffb04a")}
                ${crearLinha("🛡️", "Immune", imune, "#85c5ff")}
            </div>
        `;
    }

    async function cargarCreatures() {
        try {
            const respuesta = await fetch("/game/creatures.json");
            if (respuesta.ok) {
                const datos = await respuesta.json();
                creaturesData = Array.isArray(datos?.creatures) ? datos.creatures : (Array.isArray(datos) ? datos : []);
                for (const c of creaturesData) {
                    if (c && c.name) creaturesMapByName.set(c.name.toLowerCase().trim(), c);
                }
            }
        } catch (e) { }
    }

    function obtenerMovesDelPokemon(pokemon) {
        if (!pokemon || !pokemon.nome) return [];
        const c = creaturesMapByName.get(pokemon.nome.toLowerCase().trim());
        if (!c) return [];

        const extraer = s => {
            if (typeof s === "string") return { name: s };
            if (!s || typeof s !== "object") return null;
            const nome = s.name || s.moveName || s.move || s.id;
            return nome ? {
                name: String(nome),
                power: s.power ?? s.basePower ?? s.damage ?? s.dmg ?? null,
                type: s.type ? String(s.type) : (s.element ? String(s.element) : null)
            } : null;
        };

        const listaOriginales = [c.moves, c.attacks, c.skills, c.spells];
        for (const arr of listaOriginales) {
            if (Array.isArray(arr) && arr.length > 0) return arr.map(extraer).filter(Boolean);
        }
        return [];
    }

    // --- TOOLTIP PARSER: SUPPORTS ENGLISH / PORTUGUESE / SPANISH GAME TEXT (input parsing, unchanged) ---
    function parsePokemon(texto) {
        if (!texto) return null;
        const lineas = texto.split("\n").map(l => l.trim()).filter(Boolean);
        if (!lineas.length) return null;

        const nome = lineas[0] || "Unknown";
        const tipos = [];

        // Extract level (supports Nv 100, Lv 100, Lv. 100, etc.)
        const nivelMatch = texto.match(/(?:Nv|Lv)\.?\s*(\d+)/i);
        const nivel = Number(nivelMatch?.[1] || 0);

        for (const linea of lineas.slice(1)) {
            // Stop if it's a known stat/attribute line
            if (/^(Ativo|Active|Nv|Lv|Qualidade|Calidad|Quality|IV|HP|Atk|Def|SpA|SpD|Spd|Vel|Poder|Team|Grupo|Power)/i.test(linea)) break;
            if (/(Quality|Qualidade|Calidad|Lv\.|Nv\.|Team|Active|Ativo)/i.test(linea)) continue;
            tipos.push(linea);
        }

        const ivMatch = texto.match(/IV\s*(\d+)\s*(?:\/\s*(\d+))?/i);
        const calidadMatch = texto.match(/(?:Qualidade|Calidad|Quality)\s+([^\n]+)/i)?.[1]?.trim() || null;
        const multMatch = Number(calidadMatch?.match(/(?:×|x)\s*([\d.,]+)/i)?.[1]?.replace(',', '.'));

        const cleanNum = (str) => str ? Number(str.replace(/[,.]/g, '')) : 0;

        // Stat lookup, case-sensitive distinction for SpD and Spd preserved
        const hpStr = texto.match(/(?:HP|PS)\s+([\d.,]+)/i)?.[1];
        const atkStr = texto.match(/(?:Atk|Ataque)\s+([\d.,]+)/i)?.[1];
        const defStr = texto.match(/(?:Def|Defensa)\s+([\d.,]+)/i)?.[1];
        const spaStr = texto.match(/(?:SpA|Sp\.?\s*Atk)\s+([\d.,]+)/i)?.[1];

        // SpD -> Special Defense (case sensitive)
        let spdStr = texto.match(/\bSpD\b\s+([\d.,]+)/)?.[1];
        if (!spdStr) spdStr = texto.match(/\b(?:Sp\.?\s*Def|Df\.?\s*Sp)\b\s+([\d.,]+)/i)?.[1];

        // Spd / Vel -> Speed (case sensitive lowercase 'd' or full words)
        let velStr = texto.match(/\bSpd\b\s+([\d.,]+)/)?.[1];
        if (!velStr) velStr = texto.match(/\b(?:Speed|Vel|Velocidad)\b\s+([\d.,]+)/i)?.[1];

        return {
            nome,
            tipos,
            nivel,
            qualidade: calidadMatch,
            multiplicadorQualidade: Number.isFinite(multMatch) ? multMatch : 1,
            ivActual: Number(ivMatch?.[1] || 0),
            ivMaximo: Number(ivMatch?.[2] || CONFIG_POKEDEX.maxIVTotal),
            hp: hpStr, hpNum: cleanNum(hpStr),
            atk: atkStr, atkNum: cleanNum(atkStr),
            def: defStr, defNum: cleanNum(defStr),
            spa: spaStr, spaNum: cleanNum(spaStr),
            spd: spdStr, spdNum: cleanNum(spdStr),
            vel: velStr, velNum: cleanNum(velStr),
            poder: texto.match(/(?:Poder|Power)\s+([\d.,]+)/i)?.[1]
        };
    }

    function escapeHtml(str) {
        return String(str || "").replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
    }

    function formatearNumero(num) {
        if (!Number.isFinite(Number(num))) return "-";
        return new Intl.NumberFormat().format(num);
    }

    function crearPanelesPokedex() {
        const estilos = document.createElement('style');
        estilos.innerHTML = `
            #pokemon-reader-panel { position: fixed; top: 20px; left: 20px; z-index: 9999; background: #2c3e50; color: white; border: 2px solid #3498db; padding: 12px; border-radius: 8px; width: 290px; font-family: Arial, sans-serif; box-shadow: 0px 4px 15px rgba(0,0,0,0.5); }
            #pokemon-reader-panel.minimized .pokedex-body { display: none; }
            .pokedex-header { display: flex; justify-content: space-between; align-items: center; cursor: move; background: #34495e; padding: 6px 10px; margin: -12px -12px 10px -12px; border-radius: 6px 6px 0 0; font-weight: bold; color: #3498db; font-size: 13px; }
            .pokedex-row { display: flex; justify-content: space-between; font-size: 12px; margin-bottom: 3px; border-bottom: 1px solid rgba(255,255,255,0.05); padding-bottom: 2px; }
            #moves-panel { position: fixed; top: 20px; left: 320px; z-index: 9998; background: #1a252f; color: white; border: 2px solid #e74c3c; padding: 10px; border-radius: 8px; width: 260px; font-family: Arial, sans-serif; box-shadow: 0px 4px 15px rgba(0,0,0,0.5); display: none; }
        `;
        document.head.appendChild(estilos);

        const panel = document.createElement('div');
        panel.id = CONFIG_POKEDEX.panelId;
        panel.innerHTML = `
            <div class="pokedex-header" id="drag-handle">
                <span>🔍 Poke-Reader / IVs Formula</span>
                <div>
                    <button id="btn-toggle-moves" style="background:#e74c3c; border:none; color:white; border-radius:3px; cursor:pointer; font-size:10px; padding:2px 5px; margin-right:4px;">⚔ Moves</button>
                    <button id="minimize-pokedex" style="background:#7f8c8d; border:none; color:white; border-radius:3px; cursor:pointer; font-size:10px; padding:2px 5px;">−</button>
                </div>
            </div>
            <div class="pokedex-body" id="pokedex-content">
                <div style="text-align:center; color:#95a5a6; font-size:12px; padding:10px 0;">Hover over a Pokémon...</div>
            </div>
        `;
        document.body.appendChild(panel);

        const movesPanel = document.createElement('div');
        movesPanel.id = 'moves-panel';
        document.body.appendChild(movesPanel);

        restaurarEstadoPokedex(panel);
        actualizarPosicionMoves();

        document.getElementById('minimize-pokedex').onclick = () => {
            panel.classList.toggle('minimized');
            document.getElementById('minimize-pokedex').textContent = panel.classList.contains('minimized') ? '+' : '−';
            guardarEstadoPokedex(panel);
        };

        document.getElementById('btn-toggle-moves').onclick = () => {
            mostrarPanelMoves = !mostrarPanelMoves;
            movesPanel.style.display = mostrarPanelMoves ? 'block' : 'none';
            if (mostrarPanelMoves) {
                actualizarPanelMoves();
                actualizarPosicionMoves();
            }
            guardarEstadoPokedex(panel);
        };

        const handle = document.getElementById("drag-handle");
        let arrastando = false, offsetX = 0, offsetY = 0;

        handle.addEventListener("mousedown", e => {
            if (e.target.tagName === 'BUTTON') return;
            arrastando = true;
            const rect = panel.getBoundingClientRect();
            offsetX = e.clientX - rect.left;
            offsetY = e.clientY - rect.top;
            document.body.style.userSelect = "none";
        });

        document.addEventListener("mousemove", e => {
            if (!arrastando) return;
            let left = e.clientX - offsetX;
            let top = e.clientY - offsetY;
            panel.style.left = `${Math.max(0, Math.min(window.innerWidth - panel.offsetWidth, left))}px`;
            panel.style.top = `${Math.max(0, Math.min(window.innerHeight - panel.offsetHeight, top))}px`;
            actualizarPosicionMoves();
        });

        document.addEventListener("mouseup", () => {
            if (arrastando) {
                arrastando = false;
                document.body.style.userSelect = "";
                guardarEstadoPokedex(panel);
            }
        });
    }

    function renderizarPokedex(pokemon) {
        const container = document.getElementById('pokedex-content');
        if (!container || !pokemon) return;

        const efectividad = obtenerEfectividadHtml(pokemon);
        const maxIVTotal = CONFIG_POKEDEX.maxIVTotal;

        const ivsExactos = calcularIvsExactos(pokemon);
        const ivFinal = ivsExactos ? ivsExactos.total : pokemon.ivActual;

        const ivPorcentaje = ((ivFinal / maxIVTotal) * 100).toFixed(1);
        const maxCalidad = CONFIG_POKEDEX.calidadMaxima;
        const calidadPorcentaje = ((pokemon.multiplicadorQualidade / maxCalidad) * 100).toFixed(1);

        // --- Rareza y probabilidad según la fórmula oficial de Quality ---
        const etiquetaCalculada = obtenerEtiquetaCalidad(pokemon.multiplicadorQualidade);
        const bandaCaptura = obtenerBandaCaptura(pokemon.multiplicadorQualidade);
        const superaTopeSalvaje = pokemon.multiplicadorQualidade > CONFIG_CALIDAD.QUALITY_MAXIMA_SALVAJE;

        let infoRarezaHtml = '';
        if (etiquetaCalculada) {
            infoRarezaHtml += `<div style="font-size:10px; color:#bb8fce; margin-top:3px;">🏷️ Rarity tier: <strong>${etiquetaCalculada}</strong></div>`;
        }
        if (bandaCaptura !== null) {
            infoRarezaHtml += `<div style="font-size:10px; color:#8c98aa;">🎲 ~${bandaCaptura}% of wild captures land in this Quality range</div>`;
        } else if (superaTopeSalvaje) {
            infoRarezaHtml += `<div style="font-size:10px; color:#e67e22;">✨ Above the 1.8 wild-capture cap — only shinies/bred Pokémon reach this Quality.</div>`;
        }

        // --- Power exacto: Power = (HP+Atk+Def+SpAtk+SpDef+Spd) × Quality ---
        const poderCalculado = calcularPoderExacto(pokemon);
        const poderMostrado = pokemon.poder || (poderCalculado != null ? formatearNumero(poderCalculado) : null);

        let desgloseIvsHtml = '';
        if (ivsExactos) {
            desgloseIvsHtml = `
                <div style="font-size:10px; color:#2ecc71; background:rgba(0,0,0,0.2); padding:4px; border-radius:4px; margin-top:4px;">
                    HP:${ivsExactos.hp} | Atk:${ivsExactos.atk} | Def:${ivsExactos.def} | SpA:${ivsExactos.spa} | SpD:${ivsExactos.spd} | Vel:${ivsExactos.vel}
                </div>
            `;
        }

        const nivelBajoConfiabilidad = pokemon.nivel > 0 && pokemon.nivel < CONFIG_POKEDEX.nivelMinimoIvConfiable;
        const avisoIvsHtml = nivelBajoConfiabilidad ? `
            <div style="margin-top:4px; padding:6px 8px; background:rgba(231,76,60,0.15); border-left:3px solid #e74c3c; border-radius:0 4px 4px 0; font-size:10px; color:#f5b7b1; line-height:1.4;">
                ⚠️ <strong>Low-level estimate:</strong> at Lv. ${pokemon.nivel} the visible stats are rounded so heavily that reversing the IV formula amplifies that rounding into a large error. This number may not match the in-game IV until the Pokémon levels up (roughly Lv. ${CONFIG_POKEDEX.nivelMinimoIvConfiable}+ for a reliable read).
            </div>
        ` : '';

        container.innerHTML = `
            <div style="font-size:14px; font-weight:bold; color:#f1c40f; margin-bottom:6px; display:flex; justify-content:space-between; align-items:center;">
                <span>${escapeHtml(pokemon.nome)}</span>
                <span style="font-size:11px; color:#3498db;">Lv. ${pokemon.nivel}</span>
            </div>
            <div class="pokedex-row"><span>Types:</span> <span>${pokemon.tipos.map(t => typeBadgeHtml(obtenerClaveTipo(t) || t)).join(" ")}</span></div>
            <div class="pokedex-row" title="When comparing two Pokémon of the same species, Quality outweighs the IV total — avoid multiplying tier × IV, it's misleading.">
                <span>Quality:</span>
                <strong>${escapeHtml(pokemon.qualidade || etiquetaCalculada || 'Normal')} (${calidadPorcentaje}%)</strong>
            </div>
            ${infoRarezaHtml}
            <div class="pokedex-row" style="margin-top:4px;">
                <span>Exact IVs${nivelBajoConfiabilidad ? ' ⚠️' : ''}:</span>
                <strong style="color:${nivelBajoConfiabilidad ? '#e74c3c' : (ivPorcentaje > 80 ? '#2ecc71' : '#f39c12')}">${ivFinal} / ${maxIVTotal} (${ivPorcentaje}%)</strong>
            </div>
            ${desgloseIvsHtml}
            ${avisoIvsHtml}
            ${pokemon.hp ? `<div class="pokedex-row" style="margin-top:4px;"><span>HP / Power:</span> <span>${pokemon.hp} / ⚡${poderMostrado || '-'}</span></div>` : ''}
            ${pokemon.atk ? `<div class="pokedex-row"><span>Stats:</span> <small>Atk:${pokemon.atk} | Def:${pokemon.def} | SpA:${pokemon.spa} | SpD:${pokemon.spd} | Vel:${pokemon.vel}</small></div>` : ''}
            ${efectividad}
        `;
    }

    function actualizarPanelMoves() {
        const movesPanel = document.getElementById("moves-panel");
        if (!movesPanel || movesPanel.style.display === "none") return;

        if (!ultimoPokemon) {
            movesPanel.innerHTML = `<div style="font-size:12px; color:#aaa; text-align:center;">Waiting for active Pokémon...</div>`;
            return;
        }

        const moves = obtenerMovesDelPokemon(ultimoPokemon);
        let html = `<div style="font-weight:bold; color:#e74c3c; border-bottom:1px solid #7f8c8d; margin-bottom:8px; font-size:12px;">⚔ Movimientos de ${escapeHtml(ultimoPokemon.nome)}</div>`;

        if (moves.length === 0) {
            html += `<div style="font-size:11px; color:#8c98aa;">No move data in creatures.json</div>`;
        } else {
            for (const m of moves) {
                const clave = m.name.toLowerCase().trim();
                const danoInfo = danoPorGolpe.get(clave);
                const isActivo = ultimoGolpeUsado === clave;
                const dmgText = danoInfo ? `💥 ${formatearNumero(danoInfo.lastDmg)}` : '';

                html += `
                    <div style="background:${isActivo ? 'rgba(241,198,68,0.15)' : 'rgba(255,255,255,0.03)'}; border-left: 3px solid ${isActivo ? '#f1c40f' : '#7f8c8d'}; padding: 4px 6px; margin-bottom: 4px; border-radius: 0 4px 4px 0; font-size: 11px; display: flex; justify-content: space-between; align-items: center;">
                        <div>
                            <div style="font-weight:bold; color:${isActivo ? '#f1c40f' : '#ecf0f1'};">${escapeHtml(m.name)}</div>
                            <small style="color:#bdc3c7;">${m.type ? typeBadgeHtml(obtenerClaveTipo(m.type) || m.type) : ''} ${m.power ? 'Pwr: ' + m.power : ''}</small>
                        </div>
                        <div style="font-weight:bold; color:#e74c3c; font-size:11px;">${dmgText}</div>
                    </div>
                `;
            }
        }
        movesPanel.innerHTML = html;
    }

    // Último texto crudo procesado, para detectar cambios de contenido
    // aunque el DOM siga siendo el mismo elemento (tooltip reutilizado por el juego).
    let ultimoTextoTooltipCrudo = null;

    function procesarTextoTooltip(texto) {
        if (!texto) return;
        texto = texto.trim();
        if (!texto || texto === ultimoTextoTooltipCrudo) return;

        const pokemonParsed = parsePokemon(texto);
        if (pokemonParsed && pokemonParsed.nome) {
            ultimoTextoTooltipCrudo = texto;
            ultimoPokemon = pokemonParsed;
            renderizarPokedex(pokemonParsed);
            if (mostrarPanelMoves) actualizarPanelMoves();
        }
    }

    function manejarPosibleTooltip(e) {
        const target = e.target.closest(CONFIG_POKEDEX.tooltipSelector) || e.target.closest('[class*="tooltip"]');
        if (target) {
            procesarTextoTooltip(target.innerText || target.textContent);
        }
    }

    function observarTooltips() {
        // 1) Detecta al entrar en un tooltip nuevo (caso normal).
        //    Se usa fase de CAPTURA (tercer parámetro `true`) para que nos
        //    lleguemos a enterar del evento incluso si el propio juego llama
        //    a event.stopPropagation() en su manejador (muy común en juegos),
        //    lo cual bloquearía un listener normal en fase de burbuja.
        document.addEventListener('mouseover', manejarPosibleTooltip, true);

        // 2) Detecta cambios de contenido MIENTRAS el cursor se mueve dentro
        //    del mismo elemento/contenedor (p. ej. al pasar de Elekid a Golem
        //    sin salir de la zona interactuable, donde mouseover no vuelve a
        //    disparar por tratarse del mismo nodo). También en fase de captura.
        //    Throttle a ~10 comprobaciones/seg para no sobrecargar el hilo.
        let ultimoChequeoMs = 0;
        document.addEventListener('mousemove', (e) => {
            const ahora = performance.now();
            if (ahora - ultimoChequeoMs < 100) return;
            ultimoChequeoMs = ahora;
            manejarPosibleTooltip(e);
        }, true);

        // 3) Mecanismo original que de verdad hacía el trabajo pesado:
        //    detecta cuando el juego INSERTA un nuevo elemento de tooltip
        //    en el DOM (esto no depende de eventos de ratón ni puede ser
        //    bloqueado por stopPropagation, porque observa el DOM directamente).
        const observer = new MutationObserver((mutations) => {
            for (const m of mutations) {
                for (const node of m.addedNodes) {
                    if (node.nodeType === 1) {
                        const target = node.matches && (node.matches(CONFIG_POKEDEX.tooltipSelector) || node.matches('[class*="tooltip"]'))
                            ? node
                            : node.querySelector && (node.querySelector(CONFIG_POKEDEX.tooltipSelector) || node.querySelector('[class*="tooltip"]'));
                        if (target) {
                            procesarTextoTooltip(target.innerText || target.textContent);
                        }
                    }
                }
            }
        });
        observer.observe(document.body, { childList: true, subtree: true });
    }

    // =========================================================================
    // GENERAL INITIALIZATION
    // =========================================================================
    function iniciarScript() {
        crearInterfazThrow();
        crearPanelesPokedex();
        cargarCreatures();
        observarTooltips();
        buclePrincipal(1000);
    }

    if (document.readyState === 'complete' || document.readyState === 'interactive') {
        iniciarScript();
    } else {
        window.addEventListener('DOMContentLoaded', iniciarScript);
    }

})();
```
---

Guarda el script:

Ve al menú Archivo > Guardar (o presiona Ctrl + S).

Por ultimo Click derecho en la extenesion, --> Administrar Extension --> activas la opcion de (Permitir secuencia de comandos de usuario).

Recargas la pagina de Poke Idle Word y a Disfrutar

---

## 🛠️ Instrucciones de Instalación para Dispositivo Moviles:

---

## 🤖 Si usas Android (La opción más fácil)

---

En Android la mejor opción es Kiwi Browser. Es un navegador basado en Chromium (es decir, es idéntico a Chrome), pero tiene la ventaja de que te deja instalar cualquier extensión de la Chrome Web Store de PC.

Pasos para configurarlo:
Entra en la Play Store y descarga Kiwi Browser.

Abre Kiwi Browser y entra en la Chrome Web Store.

Busca Tampermonkey e instálalo (se instalará exactamente igual que en el ordenador).

Toca los tres puntos de arriba a la derecha en Kiwi, baja del todo y verás el icono de Tampermonkey. Entra en él.

Crea un nuevo script, borra el texto por defecto, pega el código de la v2.0 y dale a Guardar (Ctrl+S o Archivo > Guardar).

Abre de nuevo o recarga la web del juego en Kiwi Browser, pon la vista en modo escritorio (si el juego se ve raro) y empezará a lanzar solo.

--- 

## 🍏 Si usas iPhone / iPad (iOS)

---

En el ecosistema de Apple tienes dos opciones muy buenas:

## Opción A: Orion Browser (La más recomendada)
Descarga Orion Browser desde la App Store (es un navegador ultrarrápido diseñado para iOS).

Abre Orion, ve a la Chrome Web Store e instala Tampermonkey.

Abre la extensión, añade el script y guarda.

## Opción B: Safari + la app "Userscripts"
Descarga la aplicación llamada Userscripts (es gratuita y tiene el icono de una </>) desde la App Store.

Ve a los Ajustes de tu iPhone > Safari > Extensiones y activa "Userscripts".

Abre Safari, toca el botón de las extensiones (el icono de las piezas de puzle o las letras aA), abre Userscripts y crea un nuevo script pegando el código de la v2.0.

¡Listo! Entra en Poke Idle World, recarga la página y verás cómo el juego lanza automáticamente cada vez que aparece el botón.

---

## ⚠️ Un aviso muy importante sobre el "Segundo Plano" en móviles

A diferencia del ordenador, los sistemas operativos de los móviles (tanto Android como iOS) son ultra agresivos con el ahorro de batería.

¿Qué significa esto? Si minimizas el navegador en el móvil, abres WhatsApp o bloqueas la pantalla, el móvil "congelará" la pestaña del juego casi de inmediato para no gastar batería, por lo que el script se pausará.

La solución para farmear en el móvil: Si quieres dejarlo farmeando un rato mientras haces otras cosas (como comer o ver la tele), debes dejar el móvil con la pantalla encendida y el juego abierto en primer plano.

Tip extra: Puedes bajar el brillo del teléfono al mínimo para no gastar batería ni quemar la pantalla mientras el script trabaja solo.


---
