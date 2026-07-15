# Poke-Idle-World-Auto-Catch

---

## ⚡ Auto-clicker de "Throw" para Poke Idle World
Este script automatiza el lanzamiento de pokéballs haciendo clic en el botón "Throw" de manera interna.

¿La ventaja? Se ejecuta en segundo plano. Puedes minimizar el navegador, estar jugando a otra cosa o trabajando en tu PC, y el script seguirá capturando sin mover tu ratón físico ni quitarte el foco de la pantalla.
---

## 🛠️ Instrucciones de Instalación:
Instala un gestor de scripts en tu navegador (Opera, Chrome, Brave o Edge):

Descarga e instala la extensión [![Descargar Tampermonkey](https://img.shields.io/badge/Descargar-Tampermonkey-black?style=for-the-badge&logo=tampermonkey&logoColor=white)](https://www.tampermonkey.net/)

Crea el script:

Haz clic en el icono de Tampermonkey (arriba a la derecha en tu navegador) y selecciona "Crear un nuevo script..." (o "Agregar nuevo script").

Pega el código:

Borra todo el texto que aparezca por defecto en el editor.

Copia y pega el código que tienes abajo.

---
```javascript

// ==UserScript==
// @name         Auto-clicker "Throw" - Poke Idle World
// @namespace    http://tampermonkey.net/
// @version      1.1
// @description  Pulsa automáticamente el botón "Throw" en Poke Idle World simulando clics reales.
// @author       PollitoScripts
// @match        *://poke.idleworld.online/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    // Intervalo de comprobación en milisegundos (1000ms = 1 segundo)
    // Puedes bajarlo a 500 (medio segundo) si quieres que sea más rápido.
    const INTERVALO = 1000;

    // Función para simular un clic físico real (MouseDown -> MouseUp -> Click)
    // Esto engaña al motor del juego para que crea que lo ha pulsado un humano.
    function simularClicReal(elemento) {
        const opciones = { bubbles: true, cancelable: true, view: window };
        elemento.dispatchEvent(new MouseEvent('mousedown', opciones));
        elemento.dispatchEvent(new MouseEvent('mouseup', opciones));
        elemento.dispatchEvent(new MouseEvent('click', opciones));
    }

    setInterval(() => {
        // Buscamos cualquier elemento en la página
        const elementos = document.querySelectorAll('*');

        for (let elemento of elementos) {
            // Buscamos si el elemento contiene el texto "Throw"
            // Usamos .includes() en lugar de coincidencia exacta por si tiene espacios ocultos
            if (elemento.textContent && elemento.textContent.trim() === "Throw") {

                // Nos aseguramos de que sea un elemento interactivo visible y no un contenedor grande
                if (elemento.offsetWidth > 0 && elemento.offsetHeight > 0 && elemento.tagName !== 'BODY' && elemento.tagName !== 'HTML') {

                    simularClicReal(elemento);
                    console.log("¡Lanzamiento automatizado ejecutado!");
                    break; // Detiene el bucle actual para no repetir clics en el mismo segundo
                }
            }
        }
    }, INTERVALO);
})();

```
---

Guarda el script:

Ve al menú Archivo > Guardar (o presiona Ctrl + S).

¡Listo! Entra en Poke Idle World, recarga la página y verás cómo el juego lanza automáticamente cada vez que aparece el botón.
