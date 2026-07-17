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
// @name         Throw - Poke Idle World (Con Interfaz)
// @namespace    http://tampermonkey.net/
// @version      3.2
// @description  Interfaz gráfica abajo a la derecha. Clics corregidos usando LocalStorage nativo.
// @author       PollitoScripts
// @match        *://poke.idleworld.online/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    // 1. SISTEMA DE GUARDADO NATIVO (Sin Sandbox de Tampermonkey)
    function cargarConfig(clave, valorPorDefecto) {
        try {
            const guardado = localStorage.getItem(clave);
            if (guardado) return JSON.parse(guardado);
        } catch(e) {}
        return valorPorDefecto;
    }

    function guardarConfig(clave, valor) {
        localStorage.setItem(clave, JSON.stringify(valor));
    }

    let prioridadesNormal = cargarConfig("pollito_normal", ["Ultra Ball", "Super Ball", "Great Ball", "Poke Ball"]);
    let prioridadesShiny = cargarConfig("pollito_shiny", ["Idle Ball", "Ultra Ball", "Super Ball", "Great Ball", "Poke Ball"]);

    // 2. FUNCIONES BASE
    function obtenerTiempoAleatorio(min, max) {
        return Math.floor(Math.random() * (max - min + 1) + min);
    }

    async function simularClicReal(elemento) {
        if (!elemento) return;
        // Al usar @grant none, 'window' vuelve a ser la ventana real del juego, validando el clic
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

    // 3. LÓGICA DE CAPTURA
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

    // 4. INTERFAZ GRÁFICA (GUI) ABAJO A LA DERECHA
    function crearInterfaz() {
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
            <div style="font-weight:bold; text-align:center; margin-bottom:10px; font-size:15px; color:#f1c40f;">PollitoScripts v3.2</div>
            <div class="pollito-seccion">
                <div class="pollito-titulo">Prioridad Normal</div>
                <div id="lista-normal"></div>
            </div>
            <div class="pollito-seccion">
                <div class="pollito-titulo">Prioridad Shiny ✨</div>
                <div id="lista-shiny"></div>
            </div>
            <button id="pollito-guardar">Guardar Preferencias</button>
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
            alert("¡Preferencias de PollitoScripts guardadas correctamente!");
            panel.style.display = 'none';
        };

        renderizarListas();
    }

    crearInterfaz();
    buclePrincipal(obtenerTiempoAleatorio(1000, 2500));
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
