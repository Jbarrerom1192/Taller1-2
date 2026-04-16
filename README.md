# 🐶 Sistema Experto para Identificación de Razas de Perros

Un sistema experto basado en **Factores de Certeza (CF)** que identifica la raza más probable de un perro a partir de rasgos observables, manejando incertidumbre y descripciones parciales o imprecisas.

---

## 📋 Descripción

El sistema recibe un conjunto de **hechos con grado de certeza** (por ejemplo: `"orejas_paradas" = 0.8`) y mediante encadenamiento hacia adelante infiere una o varias **razas candidatas**, ordenadas por su Factor de Certeza final.

No se trata de un clasificador de imágenes ni de un modelo de ML: es un sistema basado en reglas explícitas del tipo `SI (rasgos) ENTONCES (raza)`, donde cada regla tiene un CF que indica qué tan representativa es esa combinación de rasgos para la raza.

---

## 🧠 Arquitectura del sistema

El sistema se compone de cuatro módulos principales:

**Base de conocimiento** — más de 60 reglas para 20 razas distintas. Cada regla tiene un CF positivo (evidencia a favor) o negativo (anti-evidencia que descarta la raza).

**Memoria de trabajo** — diccionario `{rasgo: certeza}` con los hechos del caso actual, donde la certeza va de 0.0 (muy incierto) a 1.0 (seguro).

**Motor de inferencia** — implementado en dos variantes:
- `inferir()`: encadenamiento hacia adelante iterativo hasta convergencia.
- `inferir_1pasada()`: una sola pasada con modo de **respaldo parcial** cuando no se disparan reglas completas. Ajusta el CF por cobertura de antecedentes.

**Módulo de explicación** — registra qué reglas se activaron, cuánto aportó cada una y cómo evolucionó el CF de cada raza (trazabilidad completa).

---

## 🎮 Modos de entrada

La interfaz interactiva (`run_interactivo()`) ofrece cuatro formas de describir al perro:

| Opción | Modo | Descripción |
|--------|------|-------------|
| 1️⃣ | Micrófono | Descripción verbal en español (Google Speech API) |
| 2️⃣ | Texto manual | Palabras clave escritas por el usuario |
| 3️⃣ | Ejemplo predefinido | Husky Siberiano, Bulldog Francés o Dálmata |
| 4️⃣ | Rasgos personalizados | Selección manual de rasgos con certeza numérica |

---

## 🐕 Razas incluidas

Pastor Alemán · Husky Siberiano · Beagle · Golden Retriever · Labrador Retriever · Rottweiler · Dachshund · Bulldog Francés · Poodle · Chihuahua · Bulldog Inglés · Pastor Inglés Antiguo · Boxer · Schnauzer · Shar Pei · Cocker Spaniel · Dálmata · Pastor Ganadero Australiano · Corgi · Pastor Belga · Collie · Shiba Inu · Akita

---

## ⚙️ Instalación

```bash
# Dependencias principales
pip install SpeechRecognition

# Para reconocimiento de voz en Windows (requiere PyAudio)
pip install pipwin
pipwin install pyaudio

# En Linux/Mac
pip install pyaudio
```

> El sistema funciona sin micrófono usando las opciones 2, 3 y 4.

---

## 🚀 Uso rápido

```python
# Abrir el notebook y ejecutar todas las celdas en orden.
# Luego lanzar la interfaz interactiva:
run_interactivo()
```

O usar el motor directamente desde código:

```python
from dataclasses import dataclass

hechos = {
    "tamano_grande": 0.9,
    "pelo_largo": 0.85,
    "ojos_azules": 0.8,
    "orejas_paradas": 0.9,
    "cola_poblada": 0.8,
}

ranking, logs = inferir(hechos, REGLAS)

for raza, cf in ranking[:5]:
    print(f"{raza}: {cf:.3f}")
```

También se puede usar inferencia con respaldo para entradas parciales:

```python
ranking, logs = inferir_1pasada(
    hechos,
    REGLAS,
    usar_respaldo=True,
    min_coincidencias=2,
)
```

---

## 📐 Fórmula de combinación de CF

Cuando múltiples reglas apuntan a la misma raza, los factores se combinan con:

```
CF(A,B) = CF(A) + CF(B) × (1 - CF(A))        si ambos ≥ 0
CF(A,B) = CF(A) + CF(B) × (1 + CF(A))        si ambos ≤ 0
CF(A,B) = (CF(A) + CF(B)) / (1 - min(|CF(A)|, |CF(B)|))  en conflicto
```

> Los CF **no son probabilidades**; son medidas de confianza basadas en evidencia acumulada.

---

## 🧪 Tests

El notebook incluye una suite de pruebas dividida en dos bloques:

```python
run_tests_basicos()          # 14 casos funcionales con raza esperada
run_tests_unitarios_ranking_y_respaldo()  # 6 tests sobre ordenamiento y modo respaldo
```

---

## 📁 Estructura del proyecto

```
WhoLetTheDogsOut.ipynb   # Notebook principal (único archivo)
README.md
```

Todo el código está autocontenido en el notebook, organizado en secciones:

1. Imports y configuración
2. Estructuras de datos (`Regla`, `Config`)
3. Funciones auxiliares (`combinar_cf`, `_calcular_aporte_regla`)
4. Motor de inferencia (`inferir`, `inferir_1pasada`)
5. Base de conocimiento (`REGLAS`, `RASGOS_DISPONIBLES`)
6. Módulo de reconocimiento de voz
7. Tests
8. Interfaz interactiva

---

## 📌 Notas técnicas

- El umbral de convergencia por defecto es `ε = 1e-3`.
- El modo respaldo requiere al menos `min_coincidencias=2` antecedentes presentes para activar una regla parcialmente.
- Las reglas negativas (CF < 0) solo se aplican a razas que ya tienen CF > 0, evitando falsos negativos.
- El reconocimiento de voz usa la API de Google Speech en español (`es-ES`) y requiere conexión a internet.
