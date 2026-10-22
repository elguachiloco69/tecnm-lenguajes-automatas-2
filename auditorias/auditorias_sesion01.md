# Bitácora de Requerimientos y Auditoría IA — Sesión 01
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Encuadre, Diagnóstico de Punteros y Metodología IA  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Diego Reyes Martínez  
**Matrícula:** No.24531376  
**Track:** C


---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Inversión de buffer in-place (char*)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al programarlo a mano lo más complicado fue manejar bien los índices de los dos punteros. Con un buffer de longitud `len`, el último elemento válido está en `buf + len - 1` y no en `buf + len`; si se me pasa ese desfase leo o escribo un byte fuera del arreglo (error off-by-one) y en C no marca error de compilación, solo da comportamiento indefinido. También tuve que decidir la condición del ciclo: usar `izq < der` y no `izq != der`, porque en longitud par los punteros se cruzan sin encontrarse y el ciclo no terminaría. Otra restricción fue que, al ser in-place, no puedo usar un arreglo auxiliar, así que el intercambio se hace con una sola variable temporal para mantener O(1) de memoria. Por último, tuve que considerar que `len` es `size_t` (sin signo): si `len` es 0, la resta `len - 1` da la vuelta y se convierte en un número enorme, por lo que hay que validar antes de calcular el puntero final.


---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Punteros, O(n) tiempo, O(1) memoria |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Estoy trabajando en C99 compilado con GCC, sin usar
bibliotecas que no sean de la biblioteca estándar.

Rol: Actúa como un ingeniero de software de sistemas y compiladores.
Especificación: Implementa una función que invierta un buffer de
caracteres in-place usando aritmética de punteros (dos punteros, uno al
inicio y otro al final). Debe correr en tiempo O(n) y usar memoria O(1),
sin arreglos auxiliares. Debe manejar los casos NULL, longitud 0,
longitud 1 y longitudes pares e impares sin acceder fuera de límites.

Output: Entrégame únicamente código modular en C y un breve análisis de
complejidad temporal y espacial. No incluyas explicaciones adicionales.
```

---


## 3. Código Devuelto por el Modelo
```c
#include <stddef.h>

void invertir_buffer(char *buf, size_t len)
{
    if (buf == NULL || len < 2) {
        return;
    }
    char *izq = buf;
    char *der = buf + (len - 1);
    while (izq < der) {
        char tmp = *izq;
        *izq++ = *der;
        *der-- = tmp;
    }
}

/* Complejidad: tiempo O(n) (n/2 intercambios), espacio O(1). */

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Valores NULL, buffers vacios y longitudes impares

Compilación: `gcc -std=c99 -Wall -Wextra -fsanitize=address,undefined -g -o harness harness.c`

Salida del harness:
```text
[PASS] longitud par                 resultado -> "dcba" (esperado "dcba")
[PASS] longitud impar               resultado -> "edcba" (esperado "edcba")
[PASS] longitud 1                   resultado -> "x" (esperado "x")
[PASS] buffer vacio (len 0)         resultado -> "" (esperado "")
[PASS] puntero NULL                 sin crash
[PASS] doble inversion              resultado -> "hola" (esperado "hola")
[PASS] con byte nulo intermedio     bytes: 64 63 00 62 61 

Resultado: TODAS LAS PRUEBAS PASARON (0 fallos)

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

```
### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Revisando la respuesta del modelo noté que el análisis de complejidad lo entregó sin mencionar el riesgo del tipo `size_t`: si la guarda inicial no existiera, con `len == 0` la expresión `len - 1` se desborda hacia abajo y `der` apuntaría muy lejos del buffer. Por eso verifiqué que la condición `len < 2` cubriera tanto el buffer vacío como el de un solo elemento antes de calcular `der`. Para comprobarlo armé un test harness con `assert` que prueba `NULL`, longitud 0, longitud 1, longitud par ("abcd"), longitud impar ("abcde") y la propiedad de que invertir dos veces regresa el buffer original. Lo compilé con `gcc -std=c99 -Wall -Wextra -fsanitize=address,undefined` y todas las pruebas pasaron sin reportes de AddressSanitizer ni UBSan.

---
**Firma digital:** elguachiloco69
