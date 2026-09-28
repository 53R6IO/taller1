# Taller 1 – Supervisión de contratos públicos de bienes (SECOP II)

**MINE-4101: Ciencia de Datos Aplicada — Universidad de los Andes, 2026-20**

## Integrantes

| Nombre | Código |
|---|---|
| Samuel Charry Tobar | 202313509 |
| Sergio Perez | 202314506 |

## Objetivo

Ayudar a la oficina de control interno de una entidad pública a decidir qué contratos de compra de bienes (compraventa y suministros) deberían tener supervisión más cercana desde la firma. Para eso buscamos qué características del contrato (valor, modalidad, destino del gasto, sector) se asocian con desviaciones en la ejecución: adición de plazo, subejecución del presupuesto y contratos terminados sin liquidar.

## Alcance

- **Datos:** 196.391 contratos de bienes firmados entre 2019 y 2025, extraídos de SECOP II (`secop_bienes.parquet`, 36 columnas).
- **Qué se hizo:** entendimiento y limpieza de datos, análisis univariado y multivariado, cuatro pruebas de hipótesis y construcción de criterios de focalización.
- **Periodo:** la adición de plazo se analiza con todos los años (2019-2025), porque el dato queda registrado aunque el contrato siga activo. La subejecución y la liquidación se analizan solo con contratos terminados firmados entre 2021 y 2024, porque los primeros años tienen mucho subregistro de pagos y en 2025 el 63,8% de los contratos sigue en ejecución.
- **Qué no se hizo:** no hay modelos predictivos ni multivariados; los resultados son asociaciones, no causas.

## Organización del repositorio

```
├── README.md               # este archivo (incluye el informe ejecutivo)
├── taller.ipynb            # notebook con toda la solución (puntos 1 a 4)
├── secop_bienes.parquet    # datos de entrada
└── requirements.txt        # dependencias
```

El notebook está organizado según el enunciado:

1. **Entendimiento inicial de los datos:** dimensiones, tipos, top 5 de variables, análisis univariado y problemas de calidad.
2. **Estrategia de análisis.**
3. **Desarrollo de la estrategia:** indicadores de desviación, elección del periodo e hipótesis 1 a 4.
4. **Resultados y recomendaciones:** segmentos de supervisión, recomendación y limitaciones.

## Instrucciones de ejecución

Se necesita Python 3.10 o superior.

```bash
pip install -r requirements.txt
jupyter notebook taller.ipynb
```

El archivo `secop_bienes.parquet` debe estar en la misma carpeta del notebook (también funciona dentro de `data/`). En Google Colab, si el archivo no está en `/content`, el notebook pide subirlo.

Solo hay un notebook, y se ejecuta de arriba a abajo (*Run All*). Tarda unos pocos minutos.

## Dependencias

| Librería | Uso |
|---|---|
| pandas | carga y manipulación de datos |
| numpy | cálculos numéricos |
| matplotlib / seaborn | visualizaciones |
| scipy | pruebas estadísticas |
| pyarrow | lectura del archivo parquet |

Versiones con las que se probó: pandas 2.3.3, numpy 1.26.4, matplotlib 3.10.0, seaborn 0.13.2, scipy 1.16.3, pyarrow 23.0.1. Se necesita matplotlib 3.10 o superior porque el notebook usa `boxplot(orientation=...)`.

---

## Informe ejecutivo: criterios de focalización para la supervisión

### Contexto

El equipo de supervisión es pequeño y no puede seguir de cerca los miles de contratos que se firman cada año. Queríamos saber si, con información disponible **al momento de la firma**, se puede identificar un grupo pequeño de contratos que concentre más riesgo de desviarse y más dinero.

Como medida principal de desviación usamos la **adición de plazo**, porque está registrada en todos los contratos. En el total de la base, **9,51% de los contratos tuvo adición de plazo** (18.662 de 196.199 después de la limpieza). Esa es la tasa base contra la que se compara todo lo demás.

### Hallazgos

**1. El valor del contrato es el factor más relacionado con la adición de plazo.**
La tasa de adición sube casi de forma continua con el valor: 2,3% en el 10% de contratos más baratos y 22,1% en el 10% más caro. La mediana de valor de los contratos con adición es 86,7 millones, frente a 30,9 millones en los que no tienen adición (Mann-Whitney, p < 0,001; correlación biserial = 0,36, efecto moderado).

**2. La modalidad también importa, sobre todo la licitación pública.**

| Modalidad | Contratos | % con adición | Veces la tasa base |
|---|---:|---:|---:|
| Licitación pública | 2.328 | 25,8% | 2,7 |
| Selección abreviada menor cuantía sin manifestación | 235 | 20,0% | 2,1 |
| Régimen especial (con ofertas) | 6.134 | 18,4% | 1,9 |
| Selección abreviada subasta inversa | 32.900 | 15,8% | 1,7 |
| Selección abreviada de menor cuantía | 8.817 | 14,7% | 1,5 |
| Contratación directa (con ofertas) | 4.995 | 9,0% | 0,9 |
| Contratación directa | 3.891 | 7,6% | 0,8 |
| Mínima cuantía | 117.685 | 7,2% | 0,8 |
| Régimen especial | 19.214 | 6,2% | 0,7 |

La relación es significativa (chi-cuadrado, p < 0,001) pero el efecto es pequeño (V de Cramér = 0,14). A las cinco primeras las llamamos **modalidades competitivas**.

**3. Valor y modalidad aportan cada uno por su lado.**
Al cruzarlos, en casi todas las modalidades la tasa sube con el valor, y dentro de un mismo rango de valor sigue habiendo diferencias entre modalidades. En el cuartil más alto (más de 100 millones), la licitación pública llega a 27,0% y el régimen especial con ofertas a 27,7%, frente a 19,0% en mínima cuantía y 13,3% en contratación directa.

**4. El sector y el destino del gasto casi no diferencian la ejecución.**
Las diferencias en ejecución presupuestal por destino del gasto y por sector son estadísticamente significativas, pero muy pequeñas (épsilon² < 0,01). La mediana de ejecución es 100% en todos los grupos. Funcionamiento tiene algo más de subejecución que inversión (10,7% vs 7,5%), pero no alcanza para usarlo como criterio principal.

**5. Hay un problema serio de reporte de pagos.**
El 42,3% de los contratos terminados no tiene ningún pago registrado y el 55,3% no aparece liquidado. Son cifras demasiado altas para ser reales, así que lo más probable es que las entidades no estén reportando pagos en SECOP. Por eso no se puede medir bien la subejecución.

**6. La limpieza de datos cambia conclusiones.**
Hay 4 contratos con valores imposibles (el mayor, 12.858 billones de pesos por tres cortacéspedes). Con ellos, la correlación de Pearson entre valor y días adicionados no es significativa (p = 0,90). Al quitarlos sí lo es. Si no se limpian los datos, se llega a una conclusión equivocada.

### Criterios de focalización recomendados

Proponemos tres niveles, que se aplican según la capacidad del equipo. Todos usan información que se conoce al firmar el contrato.

| Nivel | Criterio | % de contratos | Contratos/año aprox. | % del valor total | % con adición |
|---|---|---:|---:|---:|---:|
| 1 | Toda licitación pública | 1,2% | ~330 | 21,6% | 25,8% |
| 2 | Nivel 1 + contratos de más de 100 millones, por modalidad competitiva y con destino inversión | 9,5% | ~2.700 | 43,6% | 20,6% |
| 3 | Nivel 1 + contratos de más de 100 millones por modalidad competitiva (cualquier destino) | 18,3% | ~5.100 | 65,9% | 18,7% |

- **Nivel 1 (equipo muy pequeño):** revisar todas las licitaciones públicas. Son pocas, concentran una quinta parte del dinero y una de cada cuatro tiene adición de plazo.
- **Nivel 2 (capacidad media):** agregar los contratos de más de 100 millones que se adjudicaron por modalidad competitiva y son de inversión. Con menos del 10% de los contratos se cubre el 43,6% del valor.
- **Nivel 3 (más capacidad):** quitar el filtro de inversión. Con el 18,3% de los contratos se cubre dos tercios del valor contratado y la tasa de adición es casi el doble de la base.

Los contratos que quedan por fuera del nivel 3 (81,7%) tienen una tasa de adición de 7,5% y representan solo el 34,1% del valor.

**Recomendaciones adicionales:**

- **No usar el sector ni el destino del gasto como criterio principal**, porque no diferencian la ejecución. El destino inversión solo sirve para reducir la carga en el nivel 2.
- **Revisar la calidad del reporte en SECOP**: pedir a las áreas que registren los pagos y las liquidaciones. Mientras eso no mejore, no se puede hacer seguimiento a la subejecución con estos datos.
- **Validar el valor al registrar el contrato**, por ejemplo con una alerta cuando el valor sea 0 o absurdamente alto, para evitar errores de digitación como los encontrados.
- **Actualizar los criterios cada año**, porque la tasa de adición cambia entre años (4,2% en 2019 y 9,4% en 2024).

### Limitaciones

- Los datos los reporta cada entidad y no se pudieron contrastar con otra fuente. Corregimos los errores evidentes, pero puede haber otros.
- Una adición de plazo no siempre es una irregularidad; puede estar justificada.
- Los criterios no capturan todas las desviaciones: el nivel 3 incluye el 36% de todos los contratos con adición. El otro 64% está en contratos de menor valor o de modalidades no competitivas, donde la tasa es más baja pero hay muchos más contratos.
- Por el subregistro de pagos, el análisis de subejecución y liquidación es poco confiable, y nos apoyamos sobre todo en la adición de plazo.
- Los resultados son asociaciones. No sabemos si el valor o la modalidad causan las adiciones o si hay otra variable detrás (por ejemplo, la complejidad del bien).
- Los contratos de 2025 siguen en curso (su tasa de adición de 17,3% se sale del patrón), así que las cifras pueden cambiar.
- No hicimos un modelo multivariado; los segmentos salen de cruces simples entre variables.
