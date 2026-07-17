# Poke-Idle-World-Auto-Catch

---

## ⚡ Auto-clicker de "Throw" para Poke Idle World
Este script automatiza el lanzamiento de pokéballs haciendo clic en el botón "Throw" de manera interna.

¿La ventaja? Se ejecuta en segundo plano. Puedes minimizar el navegador, estar jugando a otra cosa o trabajando en tu PC, y el script seguirá capturando sin mover tu ratón físico ni quitarte el foco de la pantalla.
---

## 🛠️ Instrucciones de Instalación:
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
// @version      1.8
// @description  Selección de Pokéballs 100% precisa leyendo el atributo "title" y la clase "on" del código nativo.
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
        await new Promise(resolve => setTimeout(resolve, obtenerTiempoAleatorio(15, 45)));
        elemento.dispatchEvent(new MouseEvent('mouseup', opciones));
        elemento.dispatchEvent(new MouseEvent('click', opciones));
    }

    // Busca el botón genérico "Throw"
    function buscarBotonThrow() {
        const elementos = document.querySelectorAll('button, div');
        for (let elemento of elementos) {
            if ((elemento.textContent || '').trim().toLowerCase() === "throw" && elemento.offsetWidth > 0) {
                return elemento;
            }
        }
        return null;
    }

    // Detector de Shinies mediante el emoji
    function esShinyActivo() {
        const cajaCaptura = document.querySelector('.capture-box, #capture, [class*="capture"]');
        if (cajaCaptura) {
            if ((cajaCaptura.textContent || '').includes('✨')) return true;
        }
        const elementos = document.querySelectorAll('span, p, h1, h2, div');
        for (let elemento of elementos) {
            if (elemento.offsetWidth > 0 && (elemento.textContent || '').includes('✨')) return true;
        }
        return false;
    }

    function buscarYFuego() {
        const botonThrowGenerico = buscarBotonThrow();

        if (botonThrowGenerico) {
            let bolaAEquipar = null;
            const esShiny = esShinyActivo();

            // Jerarquía de recursos
            const listaPrioridades = esShiny 
                ? ["Idle Ball", "Ultra Ball", "Super Ball", "Great Ball", "Poke Ball"]
                : ["Ultra Ball", "Super Ball", "Great Ball", "Poke Ball"];

            // --- LA MAGIA NUEVA BASADA EN TU CAPTURA DE PANTALLA ---
            for (let nombreBall of listaPrioridades) {
                // Buscamos exactamente por el atributo "title" (y cubrimos la tilde de Poké Ball)
                const selector = `button[title="${nombreBall}" i], button[title="${nombreBall.replace('Poke', 'Poké')}" i]`;
                const botonBall = document.querySelector(selector);
                
                if (botonBall && !botonBall.disabled) {
                    // Si el botón NO tiene la clase "on", significa que no está seleccionada. 
                    if (!botonBall.classList.contains('on')) {
                        bolaAEquipar = botonBall;
                    }
                    // Si ya tiene la clase "on" (bolaAEquipar se queda null), ya está equipada y no hace falta clickarla.
                    break; 
                }
            }

            const tiempoReaccion = obtenerTiempoAleatorio(600, 1600);

            setTimeout(async () => {
                if (bolaAEquipar) {
                    // Si hay que cambiar de bola, hacemos clic en ella primero
                    await simularClicReal(bolaAEquipar);
                    
                    // Minipausa y clic en Throw para confirmar el lanzamiento
                    setTimeout(async () => {
                        const botonThrowConfirmar = buscarBotonThrow();
                        if (botonThrowConfirmar) await simularClicReal(botonThrowConfirmar);
                    }, obtenerTiempoAleatorio(150, 300));
                } else {
                    // Si la mejor bola ya estaba equipada (tenía la clase "on"), lanzamos directamente
                    await simularClicReal(botonThrowGenerico);
                }
                
                // Pausa post-captura
                buclePrincipal(obtenerTiempoAleatorio(2000, 2500));
            }, tiempoReaccion);

        } else {
            buclePrincipal(obtenerTiempoAleatorio(1500, 3000));
        }
    }

    function buclePrincipal(retraso) {
        setTimeout(buscarYFuego, retraso);
    }

    buclePrincipal(obtenerTiempoAleatorio(1000, 2500));
})();
```
---

Guarda el script:

Ve al menú Archivo > Guardar (o presiona Ctrl + S).

---

Por ultimo Click derecho en la extenesion, --> Administrar Extension --> activas la opcion de (Permitir secuencia de comandos de usuario).

---

¡Listo! Entra en Poke Idle World, recarga la página y verás cómo el juego lanza automáticamente cada vez que aparece el botón.
