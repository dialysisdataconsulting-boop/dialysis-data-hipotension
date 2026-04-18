# 🩺 Predictor de Hipotensión Intradialítica
**DIALYSIS-DATA Consulting**

> *"No es falta de atención… es falta de análisis."*

---

## ¿Qué es esto?

Modelo de regresión logística para predecir el riesgo de **hipotensión intradialítica** antes y durante la sesión de hemodiálisis, usando variables clínicas y de máquina capturables en cualquier unidad.

La hipotensión intradialítica es la complicación aguda más frecuente en hemodiálisis (20–30% de sesiones). Es prevenible. El problema es que nadie conecta los datos que ya existen.

---

## Resultados

| Métrica | Valor |
|---|---|
| Algoritmo | Regresión Logística |
| **AUC** | **0.714** |
| Recall (threshold 0.14) | 0.810 |
| Recall (threshold 0.20) | 0.635 |
| N sesiones | 12,404 |
| Eventos hipotensión | 2,481 (20%) |
| Dataset | MIMIC (acceso libre) |

---

## Variables predictoras

Organizadas por dominio clínico:

### Demográficas
| Variable | Descripción |
|---|---|
| `edad` | Edad del paciente |
| `peso_pre` | Peso predialítico (kg) |

### Hemodinámicas del paciente
| Variable | OR | Interpretación |
|---|---|---|
| `D_TAS` | 1.79 | TAS basal alta → mayor riesgo de caída relativa |
| `D_TAD` | 0.97 | Efecto marginal |

### Parámetros de máquina
| Variable | OR | Interpretación |
|---|---|---|
| `D_PresionVenosa` | 1.18 | Presión venosa elevada como marcador de estrés |
| `D_FlujoSangre` | 0.72 | Flujo alto → sesión eficiente, menor estrés |
| `D_PTM` | 0.83 | Presión transmembrana |

### Variables clínicas derivadas
| Variable | Fórmula | OR | Referencia |
|---|---|---|---|
| `uf_ml_kg_h` | (UF×1000)/(peso_pre×3h) | 0.70 | KDOQI: riesgo si >13 ml/kg/h |
| `delta_presion` | TAS - Presión arterial máquina | 1.35 | Diferencial hemodinámico |
| `stress_hemodinamico` | uf_ml_kg_h / TAS | 1.49 | Índice compuesto de carga |

---

## Semáforo clínico

```
🔴 ALTO   (probabilidad > 0.20) → Intervención preventiva recomendada
🟡 MEDIO  (probabilidad > 0.14) → Monitoreo frecuente
🟢 BAJO   (probabilidad ≤ 0.14) → Seguimiento estándar
```

El threshold se ajustó para maximizar **recall (sensibilidad)** porque en hemodiálisis un falso negativo tiene mayor costo clínico que un falso positivo.

---

## Uso rápido

```python
import pickle
import pandas as pd

# Cargar modelo
with open('modelo_hipotension.pkl', 'rb') as f:
    artefactos = pickle.load(f)

modelo  = artefactos['modelo']
scaler  = artefactos['scaler']
features = artefactos['features']

# Predecir para un paciente
paciente = {
    'edad': 68, 'peso_pre': 75.0, 'peso_post': 72.5,
    'D_TAS': 145, 'D_TAD': 85,
    'D_PresionArterial': 120, 'D_PresionVenosa': 180,
    'D_PTM': 25, 'D_FlujoSangre': 350, 'D_UF': 2.5
}
# Ver notebook para función predecir_riesgo() completa
```

---

## Estructura del repositorio

```
📁 dialysis-data-hipotension/
├── predictor_hipotension_dialisis.ipynb  # Notebook principal
├── modelo_hipotension.pkl                # Modelo entrenado
├── README.md                             # Este archivo
└── curva_roc.png                         # Curva ROC generada
```

---

## Estándares de referencia

- **KDIGO** — Kidney Disease: Improving Global Outcomes
- **AAMI** — Association for the Advancement of Medical Instrumentation
- **NOM-003** — Norma Oficial Mexicana de Hemodiálisis
- **KDOQI** — tasa de ultrafiltración >13 ml/kg/h como umbral de riesgo

---

## Limitaciones

- Tiempo de sesión fijo en 3h (aplicable a unidades ambulatorias de CDMX)
- Entrenado en población MIMIC (EE.UU.) — **requiere validación en población mexicana**
- Colinealidad parcial entre `D_UF` y `uf_ml_kg_h`

---

## Próximos pasos — Hacia un dataset nacional

Este modelo es el **prototipo base**. El objetivo de DIALYSIS-DATA es:

1. Recolectar datos por unidad para recalibración local
2. Personalizar el modelo a la población de cada clínica
3. Construir el **primer dataset nacional de hemodiálisis en México**
4. Validación prospectiva multicéntrica

> Si eres director/a de una unidad de hemodiálisis y quieres participar:  
> 📩 dialysis.data.consulting@gmail.com

---

## Sobre DIALYSIS-DATA Consulting

Servicio de análisis de datos aplicado a unidades de hemodiálisis.  
Enfermera especialista en cuidados críticos y nefrología.  
Experiencia en Instituto Nacional de Nutrición Salvador Zubirán.

**Ciudad de México** | dialysis.data.consulting@gmail.com

---

*Dataset: MIMIC — uso libre para investigación (PhysioNet)*  
*© 2026 DIALYSIS-DATA Consulting*
