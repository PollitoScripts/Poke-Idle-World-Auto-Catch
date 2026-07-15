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
// @version      1.3
// @description  Pulsa el botón "Throw" imitando a un jugador promedio, sin dejar rastro en la consola.
// @author       PollitoScripts
// @match        *://poke.idleworld.online/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    function simularClicReal(elemento) {
        const opciones = { bubbles: true, cancelable: true, view: window };
        elemento.dispatchEvent(new MouseEvent('mousedown', opciones));
        elemento.dispatchEvent(new MouseEvent('mouseup', opciones));
        elemento.dispatchEvent(new MouseEvent('click', opciones));
    }

    // Genera un número aleatorio entre un mínimo y un máximo de milisegundos
    function obtenerTiempoAleatorio(min, max) {
        return Math.floor(Math.random() * (max - min + 1) + min);
    }

    function buscarYFuego() {
        const elementos = document.querySelectorAll('*');
        let botonEncontrado = null;

        for (let elemento of elementos) {
            if (elemento.textContent && elemento.textContent.trim() === "Throw") {
                if (elemento.offsetWidth > 0 && elemento.offsetHeight > 0 && elemento.tagName !== 'BODY' && elemento.tagName !== 'HTML') {
                    botonEncontrado = elemento;
                    break;
                }
            }
        }

        if (botonEncontrado) {
            // Tarda entre 600ms (poco más de medio segundo) y 1600ms (segundo y medio) en reaccionar y hacer clic.
            const tiempoReaccion = obtenerTiempoAleatorio(600, 1600);

            setTimeout(() => {
                simularClicReal(botonEncontrado);

                // Tras lanzar, el jugador promedio se relaja. Espera entre 2 y 2.5 segundos antes de buscar el siguiente Pokémon.
                buclePrincipal(obtenerTiempoAleatorio(2000, 2500));
            }, tiempoReaccion);

        } else {
            // Si no hay botón en pantalla, el jugador promedio "mira" de vez en cuando (cada 1.5 a 3 segundos)
            buclePrincipal(obtenerTiempoAleatorio(1500, 3000));
        }
    }

    function buclePrincipal(retraso) {
        setTimeout(buscarYFuego, retraso);
    }

    // El script arranca tras un pequeño retraso inicial aleatorio
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
