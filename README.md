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

---

Por ultimo Click derecho en la extenesion, --> Administrar Extension --> activas la opcion de (Permitir secuencia de comandos de usuario).

---

¡Listo! Entra en Poke Idle World, recarga la página y verás cómo el juego lanza automáticamente cada vez que aparece el botón.
