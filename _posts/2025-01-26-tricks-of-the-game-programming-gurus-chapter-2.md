---
layout: post
title: "Tricks of The Game Programming Gurus - Capítulo 2"
description: "Un recorrido por los ejemplos prácticos presentados en el capítulo 2 del libro Tricks of The Game Programming Gurus."
image: "https://bgm.cl/assets/img/posts/tricks-of-the-game-programming-gurus-chapter-2/cover.png"
discussion: "https://github.com/BGMP/bgm.cl/discussions/11"
---
¡Hola a todos, ha pasado mucho tiempo desde la última vez que publiqué algo por aquí! ¡Espero que estén bien y estén disfrutando de un buen 2025!

Esta semana retomé la lectura de un libro que comencé a leer el verano pasado, pero no había tenido tiempo de continuar hasta ahora. El título de este libro es Tricks of The Game Programming Gurus, de André LaMothe, uno de mis favoritos. Trata sobre el diseño y desarrollo de videojuegos de la era de DOOM y Wolfenstein. Aquí hay una imagen de la portada del libro:

<div class="popup-post" align="center">
    <a href="{{ site.url }}/assets/img/posts/tricks-of-the-game-programming-gurus-chapter-2/cover.png">
        <img src="{{ site.url }}/assets/img/posts/tricks-of-the-game-programming-gurus-chapter-2/cover.png" alt="" width="40%">
    </a>
</div>
<br/>

La última vez que me puse a leer este libro, logré hacer funcionar algunos de los ejemplos del Capítulo 2 en un entorno moderno, ¡y ahora que lo retomo me doy cuenta de que nunca compartí mi progreso en mi propio blog! Aquí hay una republicación de un [comentario](https://github.com/myfoundation/Game-Programming-Gurus-Reloaded/issues/2#issuecomment-1934658181) que hice en un repositorio de GitHub dedicado a este libro. ¡Disfruten!

...

Para futuros lectores,

Después de varios intentos y errores, ¡logré configurar un entorno de desarrollo funcional para el capítulo 2! Al final, opté por usar DOSBox, más un par de entornos para hacer que todo se construya y enlace correctamente.

Aquí hay una guía paso a paso que elaboré durante el proceso:

#### DOSBox
---
Simplemente descarga DOSBox para tu sistema operativo. Dejé un enlace a su página de descargas en las referencias.

Una vez instalado, monta tu disco C como algún directorio local en tu máquina host. Yo solo creé un directorio llamado `DOSPROGS/` en mi Escritorio y funcionó perfectamente.

#### Ensamblador (MASM611)
---
Descarga Microsoft Macro Assembler (enlace en las referencias). Luego, extrae el contenido de los discos a una carpeta en tu unidad C de MS DOS. Yo opté por `C:\DOSPROGS\MASM611\`.

Desde la consola de comandos de DOS, ejecuta el archivo `SETUP.EXE` que viene con MASM611. El instalador te mostrará una pantalla de instalación. Sigue las instrucciones y selecciona todos los valores predeterminados que consideres apropiados.

<div class="popup-post" align="center">
    <a href="{{ site.url }}/assets/img/posts/tricks-of-the-game-programming-gurus-chapter-2/masm.png">
        <img src="{{ site.url }}/assets/img/posts/tricks-of-the-game-programming-gurus-chapter-2/masm.png" alt="" width="80%">
    </a>
</div>
<br/>

#### C
---
Descarga Borland Turbo C++ (enlace en las referencias). Al igual que con MASM, extrae el contenido de los discos a una carpeta en tu unidad C de MS DOS. Yo usé `C:\DOSPROGS\TCC\`.

Desde la línea de comandos de DOS, ejecuta el archivo `INSTALL.EXE` que viene con TCC. El instalador te mostrará una pantalla de instalación. Sigue las instrucciones y se instalará correctamente. Además, en un momento del proceso de instalación, TCC te preguntará en qué carpeta debería instalarse. En mi caso, seleccioné `C:\TC\`.

<div class="popup-post" align="center">
    <a href="{{ site.url }}/assets/img/posts/tricks-of-the-game-programming-gurus-chapter-2/turbocpp.png">
        <img src="{{ site.url }}/assets/img/posts/tricks-of-the-game-programming-gurus-chapter-2/turbocpp.png" alt="" width="80%">
    </a>
</div>
<br/>

#### Ensamblando, Compilando, Enlazando y Ejecutando
---
##### Paso 1: Ensamblando
Primero, tendrás que escribir un archivo de ensamblador y usar MASM para ensamblarlo. Aquí hay un listado tomado del libro:

Listado 2.8 - Un procedimiento en ensamblador para establecer el modo de video (SETMODEA.ASM)
```asm
.MODEL MEDIUM, C

.CODE

PUBLIC Set_Mode

Set_Mode PROC FAR C vmode:WORD

mov AH,0
mov AL, BYTE PTR vmode

int 10h

ret

Set_Mode ENDP

END
```

Para ensamblar con MASM:
```bash
CD C:\MASM611\BIN
MASM SETMODEA.ASM;
```

Esto creará un archivo `SETMODEA.OBJ`. ¡Guarda este archivo, tendrás que enlazarlo más tarde!

##### Paso 2: Compilando
Ahora, necesitarás la función C que llama a lo que has definido en tu archivo ASM. Guarda el siguiente listado en un archivo:

Listado 2.9 - Una función C para probar el modo de video (SETMODEC.C)
```c
#include <stdio.h>

#define VGA256 0x13
#define TEXT_MODE 0x03

extern Set_Mode(int mode);

void main(void)
{
  // Esto establece el modo de video a 320x200, modo de 256 colores.
  Set_Mode(VGA256);

  // Esperar a que se presione una tecla.
  while (!kbhit()) {}

  // Volver a poner la computadora en modo texto.
  Set_Mode(TEXT_MODE);
} // fin main
```

Para compilar con TCC:
```bash
CD C:\TC\BIN
TCC -IC:\TC\INCLUDE -mm -c SETMODEC.C;
```

Esto creará un archivo `SETMODEC.OBJ`. ¡Guarda este archivo, tendrás que enlazarlo más tarde!

##### Paso 3: Enlazando
Todavía desde el directorio TC, trae `SETMODEA.OBJ` y enlázalo junto con `SETMODEC.OBJ`. Puedes hacerlo de la siguiente manera:
```bash
TLINK c0m SETMODEC.OBJ SETMODEA.OBJ,SETMODE.EXE,,cm -LcC:\TC\LIB
```

##### Paso 4: ¡Ejecutar!
Simplemente ejecuta el archivo ejecutable para probar los resultados:
```bash
SETMODE.EXE
```

##### Notas
---
La flag `-mm` pasada a TCC, y los argumentos `c0m` y `cm` pasados a TLINK representan el tipo de modelo de memoria que se está usando para el programa. En este caso, son todas `m`s porque estamos usando el modelo MEDIUM.

Así, debes reemplazar estos argumentos para trabajar con otros modelos según corresponda:
* TINY: `-mt`; `c0t` y `ct`.
* SMALL: `-ms`; `c0s` y `cs`.
* COMPACT: `-mc`; `c0c` y `cc`.
* MEDIUM: `-mm`; `c0m` y `cm`.
* LARGE: `-ml`; `c0l` y `cl`.
* HUGE: `-mh`; `c0h` y `ch`.

¡Bueno, eso es todo! Continuaré leyendo ahora, espero que esta guía te ayude a comenzar.

##### Referencias y Descargas
---
- DOSBox - [https://www.dosbox.com/download.php?main=1](https://www.dosbox.com/download.php?main=1)
- Microsoft Macro Assembler 6.11 (3.5) (MASM611) - [https://winworldpc.com/product/macro-assembler/6x](https://winworldpc.com/product/macro-assembler/6x)
- Borland Turbo C++ 3.0 (3.5-720k) (TC) - [https://winworldpc.com/product/turbo-c/3x](https://winworldpc.com/product/turbo-c/3x)
