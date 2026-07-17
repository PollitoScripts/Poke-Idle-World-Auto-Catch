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
// @name         Throw - Poke Idle World
// @namespace    http://tampermonkey.net/
// @version      2.0
// @description  Detector de Shinies ultra-preciso limitado solo a la zona de captura activa para evitar falsos positivos.
// @author       PollitoScripts
// @match        *://poke.idleworld.online/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

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

    // NUEVO DETECTOR QUIRÚRGICO: Solo busca el emoji ✨ dentro de la caja de captura activa
    function esShinyActivo(botonThrow) {
        if (!botonThrow) return false;
        
        // Buscamos la caja de captura activa donde vive el botón "Throw"
        const cajaCaptura = botonThrow.closest('.capture-box, #capture, [class*="capture" i]') 
                           || botonThrow.parentElement.parentElement.parentElement;
        
        if (cajaCaptura) {
            const textoCaja = cajaCaptura.textContent || '';
            // Si el emoji ✨ está específicamente dentro de esta caja, es un Shiny activo
            if (textoCaja.includes('✨')) {
                return true;
            }
        }
        return false;
    }

    function buscarYFuego() {
        const botonThrowGenerico = buscarBotonThrow();

        if (botonThrowGenerico) {
            let bolaAEquipar = null;
            const esShiny = esShinyActivo(botonThrowGenerico); // Pasamos el botón para localizar la zona exacta

            // Jerarquía estricta de recursos
            const listaPrioridades = esShiny 
                ? ["Idle Ball", "Ultra Ball", "Super Ball", "Great Ball", "Poke Ball"]
                : ["Ultra Ball", "Super Ball", "Great Ball", "Poke Ball"];

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

            // Tiempos rápidos de reacción para que no se acumulen
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
                
                // Pausa post-captura rápida para limpiar la cola
                buclePrincipal(obtenerTiempoAleatorio(900, 1400));
            }, tiempoReaccion);

        } else {
            buclePrincipal(obtenerTiempoAleatorio(800, 1500));
        }
    }

    function buclePrincipal(retraso) {
        setTimeout(buscarYFuego, retraso);
    }

    buclePrincipal(obtenerTiempoAleatorio(500, 1200));
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

Abre la web del juego en Kiwi Browser, pon la vista en modo escritorio (si el juego se ve raro) y empezará a lanzar solo.

--- 

## 🍏 Si usas iPhone / iPad (iOS)

---

En el ecosistema de Apple tienes dos opciones muy buenas:

## Opción A: Orion Browser (La más recomendada)
Descarga Orion Browser desde la App Store (es un navegador ultrarrápido diseñado para iOS).

Abre Orion, ve a la Chrome Web Store e instala Tampermonkey.

Abre la extensión, añade el script v2.0 y guarda.

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
