# Coordinación de metaheurísticas colaborativas guiada por series temporales mediante Dynamic Time Warping

**Taller CIIS** · Práctica guiada en Google Colab

**Lucas Erazo** · Estudiante de Ingeniería Civil Informática · Pontificia Universidad Católica de Valparaíso

Una metaheurística guarda el mejor valor que encontró hasta el momento. Esa curva sube al principio y después se aplana. En este taller armas un monitor que detecta cuándo la curva se aplanó, usando DTW (*Dynamic Time Warping*), y ves cómo reacciona la búsqueda GWO-DE: cambia sus parámetros entre explotación y exploración.

Trabajamos con dos problemas: la mochila multidimensional (MKP) y HRES2, el diseño de un sistema de viento, sol, batería y electrolizador para producir hidrógeno.

El taller está pensado para quienes no conocen metaheurísticas ni DTW. Se programa una sola función; el resto es ejecutar, mover perillas y comparar.

## Empieza aquí

1. Abre [Google Colab](https://colab.research.google.com) y usa *Archivo → Subir notebook* con `Taller_Ciis.ipynb`. Si está en tu Drive, ábrelo desde ahí con Colab.
2. Ejecuta *Entorno de ejecución → Ejecutar todas*. El notebook queda listo aunque no hayas completado el TODO.
3. Vuelve a la sección 2 y avanza en orden.

No hace falta instalar nada ni subir otros archivos. Los logos y el modelo de HRES2 van dentro del notebook, y la instancia del MKP se descarga sola.

## Archivos

| Archivo | Para quién | Estado |
|---|---|---|
| `Taller_Ciis.ipynb` | Participantes | Sin salidas, con un TODO y la solución colapsada. |
| `Taller_Ciis_Resuelto.ipynb` | Quien guía el taller | Con la función escrita y todas las celdas ejecutadas. |

Las celdas con mucho código aparecen en Colab solo con su título. Se abren con doble clic.

## Qué se hace en cada sección

| # | Sección | Qué haces |
|---|---|---|
| 1 | La curva y la meseta | Miras una corrida de GWO en el MKP y anotas desde qué iteración se estancó. |
| 2 | Rampa o constante | Escribes `calcular_D1_D2_delta`, la única función del taller: compara la ventana con una rampa y con una constante. |
| 3 | Cuándo dispara | Comparas las detecciones del monitor con tu estimación. |
| 4 | Jugar con las perillas | Cambias `W`, `band`, `patience` y `min_slope`, y ves cómo se mueven las detecciones. |
| 5 | Reaccionar con parámetros | Comparas GWO-DE con y sin reacción, y lees la figura de diagnóstico temporal. |
| 6 | HRES2 | Pruebas tu propio sistema de energía, corres GWO-DE con el monitor y usas las mismas perillas sobre la curva nueva. |
| 7 | Cierre | Comentan tres preguntas en grupo. |

## Los dos problemas

| | MKP | HRES2 |
|---|---|---|
| Objetivo | Maximizar la ganancia sin pasar 5 límites de capacidad. | Minimizar el costo de la energía, LCOE (CNY/kWh). |
| Una solución | 100 bits: cada ítem entra o no entra. | 4 números: viento (MW), unidades de electrolizador, batería (MW) y duración de la batería. |
| Datos | `mknapcb1`, instancia 9, de OR-Library. Mejor conocido: 24411. | Un año de clima sintético, hora por hora (8760 horas). |
| Búsqueda en el taller | Población 20, 1000 iteraciones, semilla 0. | Población 30, 1000 generaciones, semilla 1. |

El modelo de HRES2 es el mismo del paper: capacidad de generación de 200 MW, electrolizador de 50 a 100 MW, batería de 0 a 50 MW con 1, 2 o 4 horas, y un límite a la energía que se vende a la red.

## El método

| Pieza | Qué hace | Valores |
|---|---|---|
| Ventana | Los últimos `W` puntos del mejor valor acumulado. | `W = 40` |
| $D_1$, $D_2$, $\Delta$ | Distancia DTW de la ventana a una rampa y a una constante, y su diferencia. | `band = 4` |
| Regla de disparo | Sin mejora durante `plateau_max` pasos, la ventana se parece a la constante y no a la rampa, durante `patience` observaciones seguidas. | `plateau_max = 15`, `patience = 8` |
| Umbrales | Percentil 40 de $D_2$ y percentil 60 de $D_1$ y $\Delta$, con la historia del propio monitor. | 40 / 60 |
| Reacción | Explotación pasa a exploración cuando el monitor dispara. Vuelve a explotación si hubo una mejora reciente. | Explotación: $a=0.5$, $F=0.5$, $CR=0.9$. Exploración: $a=2.0$, $F=0.9$, $CR=0.3$. |

## Requisitos

- Una cuenta de Google para usar Colab gratis, con CPU. No hace falta GPU.
- Internet para descargar la instancia del MKP. Si la descarga falla, el notebook genera una instancia del mismo tamaño, sin mejor conocido.
- `numba` para que HRES2 corra rápido. Colab normalmente lo trae; sin `numba`, la búsqueda de HRES2 tarda bastante más.

En un computador de escritorio el notebook completo corrió en unos dos minutos y medio, y la búsqueda de HRES2 en unos 20 segundos. En Colab puede tardar más.

## Para quien guía el taller

- [ ] Abre `Taller_Ciis.ipynb` en Colab antes del taller y ejecútalo completo.
- [ ] Comprueba que aparezcan los sliders de las secciones 4 y 6c. Si no aparecen, la celda que viene debajo hace lo mismo con números.
- [ ] Ten a mano `Taller_Ciis_Resuelto.ipynb` para mostrar las respuestas.

Si la sección 2 se alarga, carga la Solución y sigue adelante. Las secciones 4 y 5 son el centro del taller: no las recortes.

## Referencias

- Trabajo en el que se basa el taller: *Time-Series-Driven Parameter Control of a Hybrid Grey Wolf Optimizer and Differential Evolution via Dynamic Time Warping*.
- Sakoe, H., Chiba, S. (1978). Dynamic programming algorithm optimization for spoken word recognition. *IEEE Transactions on Acoustics, Speech, and Signal Processing*, 26(1), 43–49. https://doi.org/10.1109/TASSP.1978.1163055
- Mirjalili, S., Mirjalili, S. M., Lewis, A. (2014). Grey Wolf Optimizer. *Advances in Engineering Software*, 69, 46–61. https://doi.org/10.1016/j.advengsoft.2013.12.007

## Contacto

**Lucas Erazo**
Estudiante de Ingeniería Civil Informática, Escuela de Ingeniería Informática, Pontificia Universidad Católica de Valparaíso, Valparaíso, Chile

- Correo: [lucas.erazo@pucv.cl](mailto:lucas.erazo@pucv.cl)
- ORCID: [0009-0001-3259-454X](https://orcid.org/0009-0001-3259-454X)

Si tienes preguntas sobre el taller o encuentras un error en los notebooks, escríbeme al correo.
