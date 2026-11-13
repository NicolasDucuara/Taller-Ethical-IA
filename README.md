# Taller: Auditoría de Sesgo y Documentación de Modelos de IA

[![Fairness Audit CI](https://github.com/NicolasDucuara/Taller-Ethical-IA/actions/workflows/fairness-audit.yml/badge.svg)](https://github.com/NicolasDucuara/Taller-Ethical-IA/actions/workflows/fairness-audit.yml)

> ✏️ **Reemplaza `<USUARIO>/<REPO>` del badge por los de tu repositorio.**

Taller práctico de 30 minutos. **No se entrena ningún modelo**: se audita uno ya
existente a partir de sus predicciones, se integra la auditoría en CI y se
documentan los resultados en una Model Card.

## 🎯 Objetivo

Al finalizar el taller serás capaz de:

1. Calcular métricas de equidad (**Disparate Impact**, **Statistical Parity Difference**,
   **Equal Opportunity Difference**) a partir de un CSV de predicciones.
2. Aplicar la **regla de los cuatro quintos** (DI ≥ 0.80) como criterio de aprobación.
3. Usar la auditoría como **quality gate en CI/CD** con GitHub Actions.
4. Documentar los hallazgos en una **Model Card**.

## 📁 Estructura

```
├── .github/workflows/fairness-audit.yml   # CI (no modificar)
├── data/model_predictions.csv             # Dataset a auditar
├── scripts/calculate_audit.py             # Script con 2 TODO por implementar
├── tests/                                 # Tests del script + datasets de referencia
├── MODEL_CARD_TEMPLATE.md                 # Plantilla de la Model Card
└── docs/img/                              # Aquí van tus capturas
```

## 🚀 Cómo empezar

Cada estudiante trabaja en **su propio repositorio público**, porque el CI de GitHub Actions
se ejecuta ahí. Clonar el repositorio del taller no basta: tu `git push` debe ir a tu repo, no al del docente.

### 1. Crea tu repositorio vacío en GitHub

1. Entra a <https://github.com/new>.
2. Nómbralo, por ejemplo, `bias-audit-workshop`.
3. Selecciona visibilidad **Public**.
4. **No marques** "Add a README", ".gitignore" ni "license". El repo debe quedar vacío.
5. Pulsa **Create repository**.

### 2. Clona el taller y apunta al repo que creaste

```bash
git clone https://github.com/<USUARIO_DOCENTE>/<REPO_DEL_TALLER>.git bias-audit-workshop
cd bias-audit-workshop

git remote set-url origin https://github.com/<TU_USUARIO>/<TU_REPO>.git
git branch -M main
git push -u origin main
```

Verifica que apuntas a tu repo y no al del docente:

```bash
git remote -v      # debe mostrar TU usuario, no el del docente
```

### 3. Comprueba que el CI está activo

1. Abre la pestaña **Actions** de tu repositorio.
2. Debe aparecer un run de **Fairness Audit CI** disparado por tu primer push.
3. Si ves un botón verde _"I understand my workflows, go ahead and enable them"_, púlsalo y relanza con **Run workflow**.

> Este primer run es **esperado**: `unit-tests` ✅ (con 2 tests _skipped_) y
> `fairness-gate` ❌. El modelo es sesgado a propósito; no es un error tuyo.

### 4. Actualiza el badge

Reemplaza `<USUARIO>/<REPO>` en la primera línea de este README por los tuyos.

### 5. Instala las dependencias (Python 3.9+)

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements-dev.txt
```

## ▶️ Ejecución

```bash
python scripts/calculate_audit.py                    # audita data/model_predictions.csv
python scripts/calculate_audit.py --data <otro.csv>  # audita otro archivo
pytest -v                                            # tests del script
echo $?                                              # 0 = aprueba, 1 = viola la regla 4/5
```

---

## 📝 Pasos del taller

### Paso 1. Ejecuta el script tal como está

Corre el script sin modificar nada. Las métricas SPD y EOD deben aparecer como `PENDIENTE`.

**Evidencia 1: salida en consola (TODOs pendientes):**

```
Archivo auditado: /home/nicolas/fastAPI/Taller-Ethical-IA/data/model_predictions.csv
Registros cargados: 24

Accuracy global: 62.5%
Tasa de aceptación - Grupo mayoritario (0): 75.0%
Tasa de aceptación - Grupo minoritario (1): 25.0%

Matriz de confusión - Grupo Mayoritario (0):
   TN=2  FP=3  FN=1  TP=6
   TPR (Recall) = 85.7%   FPR = 60.0%

Matriz de confusión - Grupo Minoritario (1):
   TN=4  FP=0  FN=5  TP=3
   TPR (Recall) = 37.5%   FPR = 0.0%

--- Métricas de equidad ---
Disparate Impact Ratio (DI):          0.333
Statistical Parity Difference (SPD):  PENDIENTE (TODO 2)
Equal Opportunity Difference (EOD):   PENDIENTE (TODO 1)

============================================================
🚨 ALERTA: VIOLACIÓN DE LA REGLA DE LOS CUATRO QUINTOS 🚨
   DI = 0.333  <  umbral = 0.80
   El modelo presenta IMPACTO DISPAR contra el grupo minoritario.
   Acción: devolver a desarrollo para mitigación de sesgos.
============================================================
```

![Evidencia 1: TODOs pendientes](docs/img/evidencia-1-pendientes.png)

### Paso 2. Implementa los TODO

En `scripts/calculate_audit.py` implementa:

- **TODO 1**: `equal_opportunity_difference` (TPR_min − TPR_maj)
- **TODO 2**: `statistical_parity_difference` (rate_min − rate_maj)

Verifica con `pytest -v`: los tests de SPD y EOD deben pasar de _skipped_ a _passed_.

**Evidencia 2: salida de `pytest -v` con TODOs implementados:**

```
============================= test session starts ==============================
platform linux -- Python 3.12.3, pytest-9.2.0, pluggy-1.7.0
cachedir: .pytest_cache
rootdir: /home/nicolas/fastAPI/Taller-Ethical-IA
collecting ... collected 6 items

tests/test_audit.py::test_selection_rates_biased PASSED                  [ 16%]
tests/test_audit.py::test_disparate_impact_biased PASSED                 [ 33%]
tests/test_audit.py::test_spd_and_eod_biased PASSED                      [ 50%]
tests/test_audit.py::test_zero_majority_rate_raises PASSED               [ 66%]
tests/test_audit.py::test_exit_code_is_1_for_biased_model PASSED         [ 83%]
tests/test_audit.py::test_exit_code_is_0_for_fair_model PASSED           [100%]

============================== 6 passed in 1.09s ===============================
```

![Evidencia 2: pytest -v](docs/img/evidencia-2-pytest.png)

**Evidencia 3: salida en consola sobre el dataset sesgado (TODOs implementados):**

```
Archivo auditado: /home/nicolas/fastAPI/Taller-Ethical-IA/data/model_predictions.csv
Registros cargados: 24

Accuracy global: 62.5%
Tasa de aceptación - Grupo mayoritario (0): 75.0%
Tasa de aceptación - Grupo minoritario (1): 25.0%

Matriz de confusión - Grupo Mayoritario (0):
   TN=2  FP=3  FN=1  TP=6
   TPR (Recall) = 85.7%   FPR = 60.0%

Matriz de confusión - Grupo Minoritario (1):
   TN=4  FP=0  FN=5  TP=3
   TPR (Recall) = 37.5%   FPR = 0.0%

--- Métricas de equidad ---
Disparate Impact Ratio (DI):          0.333
Statistical Parity Difference (SPD):  -0.500
Equal Opportunity Difference (EOD):   -0.482
(!) Brecha mayor a 0.10 en SPD y/o EOD: revisar. Estas dos métricas son informativas; la compuerta usa solo el DI.

============================================================
🚨 ALERTA: VIOLACIÓN DE LA REGLA DE LOS CUATRO QUINTOS 🚨
   DI = 0.333  <  umbral = 0.80
   El modelo presenta IMPACTO DISPAR contra el grupo minoritario.
   Acción: devolver a desarrollo para mitigación de sesgos.
============================================================
```

![Evidencia 3: dataset sesgado con TODOs implementados](docs/img/evidencia-3-sesgado.png)

### Paso 3. Primer push: la compuerta debe fallar

```bash
git add . && git commit -m "Implementa SPD y EOD" && git push
```

Ve a la pestaña **Actions** de tu repositorio.

> **¿Por qué hay un check en rojo?** El script no está roto. Detecta que el _modelo_
> es sesgado y devuelve `exit 1`. Eso es exactamente lo que debe hacer un quality gate.

| #   | Captura                   | Guárdala como                    | Qué debe verse                        |
| --- | ------------------------- | -------------------------------- | ------------------------------------- |
| 1   | Resumen del workflow      | `docs/img/ci-1-gate-rojo.png`    | `unit-tests` ✅ y `fairness-gate` ❌  |
| 2   | Log del job fallido       | `docs/img/ci-2-log-alerta.png`   | Mensaje de violación y el valor de DI |
| 3   | Pestaña _Summary_ del run | `docs/img/ci-3-summary-rojo.png` | Tabla con DI, SPD y EOD               |

![CI en rojo](docs/img/ci-1-gate-rojo.png)
![Log de alerta](docs/img/ci-2-log-alerta.png)
![Summary en rojo](docs/img/ci-3-summary-rojo.png)

### Paso 4. Mitigación simulada

Simula que el equipo de desarrollo corrigió el modelo (por ejemplo, ajustando el umbral de decisión del grupo minoritario):

```bash
cp tests/fixtures/fair_predictions.csv data/model_predictions.csv
```

**Evidencia 4: salida en consola tras la mitigación:**

```
Archivo auditado: /home/nicolas/fastAPI/Taller-Ethical-IA/data/model_predictions.csv
Registros cargados: 24

Accuracy global: 75.0%
Tasa de aceptación - Grupo mayoritario (0): 75.0%
Tasa de aceptación - Grupo minoritario (1): 66.7%

Matriz de confusión - Grupo Mayoritario (0):
   TN=2  FP=3  FN=1  TP=6
   TPR (Recall) = 85.7%   FPR = 60.0%

Matriz de confusión - Grupo Minoritario (1):
   TN=3  FP=1  FN=1  TP=7
   TPR (Recall) = 87.5%   FPR = 25.0%

--- Métricas de equidad ---
Disparate Impact Ratio (DI):          0.889
Statistical Parity Difference (SPD):  -0.083
Equal Opportunity Difference (EOD):   +0.018

============================================================
✅ APROBADO: el modelo cumple la regla de los cuatro quintos.
   DI = 0.889  >=  umbral = 0.80
============================================================
```

![Evidencia 4: tras la mitigación](docs/img/evidencia-4-mitigacion.png)

### Paso 5. Segundo push: la compuerta debe pasar

```bash
git add . && git commit -m "Aplica mitigación simulada" && git push
```

| #   | Captura                   | Guárdala como                     | Qué debe verse      |
| --- | ------------------------- | --------------------------------- | ------------------- |
| 4   | Resumen del workflow      | `docs/img/ci-4-gate-verde.png`    | Ambos jobs ✅       |
| 5   | Pestaña _Summary_ del run | `docs/img/ci-5-summary-verde.png` | Tabla con DI ≥ 0.80 |

![CI en verde](docs/img/ci-4-gate-verde.png)
![Summary en verde](docs/img/ci-5-summary-verde.png)

---

## 🧠 Reflexión

### A. Resultados

Completa con los valores de tu ejecución.

| Métrica                     | Dataset sesgado | Tras mitigación |
| --------------------------- | --------------- | --------------- |
| Accuracy global             | 62.5%           | 75.0%           |
| Tasa de aceptación, grupo 0 | 75.0%           | 75.0%           |
| Tasa de aceptación, grupo 1 | 25.0%           | 66.7%           |
| TPR, grupo 0                | 85.7%           | 85.7%           |
| TPR, grupo 1                | 37.5%           | 87.5%           |
| FPR, grupo 0                | 60.0%           | 60.0%           |
| FPR, grupo 1                | 0.0%            | 25.0%           |
| **Disparate Impact (DI)**   | 0.333           | 0.889           |
| **SPD**                     | −0.500          | −0.083          |
| **EOD**                     | −0.482          | +0.018          |
| ¿Pasa la regla 4/5?         | ☐ Sí ☑ No       | ☑ Sí ☐ No       |

Muestra el cálculo manual de las tres métricas del dataset sesgado:

```
DI  = 0.25 / 0.75 = 0.333
SPD = 0.25 − 0.75 = −0.500
EOD = 0.375 − 0.857 = −0.482
```

### B. Preguntas

**1. ¿La EOD coincide con la conclusión del DI? ¿Miden lo mismo? Justifica.**

> Sí, ambas apuntan a la misma conclusión (el modelo perjudica al grupo minoritario), pero **no miden lo mismo**. El DI (0.333) compara las **tasas de aceptación** de ambos grupos sin importar si las personas estaban realmente calificadas: mide paridad demográfica en el resultado. La EOD (−0.482) compara únicamente el **TPR** —qué porcentaje de las personas *verdaderamente calificadas* (`true_label = 1`) recibió un resultado positivo—: mide igualdad de oportunidad. Que ambas sean fuertemente desfavorables al grupo 1 refuerza el diagnóstico desde dos ángulos distintos: no solo se acepta menos al grupo minoritario (DI), sino que, incluso entre quienes sí merecían aprobación, el modelo acierta mucho menos con ese grupo (EOD). Podrían discrepar en otros escenarios (p. ej., si las tasas base de calificación fueran muy distintas), por eso se reportan por separado.

**2. Calcula qué proporción de cada grupo está realmente calificada (`true_label = 1`). ¿La brecha de aceptación se explica por mérito?**

> - Grupo 0 (mayoritario): 7 de 12 calificados = **58.3%**.
> - Grupo 1 (minoritario): 8 de 12 calificados = **66.7%**.
>
> **No, la brecha no se explica por mérito; de hecho ocurre lo contrario.** El grupo minoritario está *más* calificado (66.7% frente a 58.3%), pero recibe una tasa de aceptación mucho *menor* (25.0% frente a 75.0%). Si la decisión siguiera el mérito, el grupo 1 debería aprobarse al menos tanto como el grupo 0. La disparidad es puro sesgo del modelo, no diferencia de calificación real.

**3. ¿Cuántas personas calificadas del grupo 1 fueron rechazadas (FN)? ¿Qué implica en un caso real?**

> **FN del grupo 1 = 5.** De las 8 personas realmente calificadas del grupo minoritario, 5 fueron rechazadas (solo 3 TP). En un caso real (crédito, empleo, admisión) esto significa que la mayoría de los candidatos aptos del grupo desfavorecido quedan fuera: pérdida de oportunidades legítimas, daño reputacional y legal para la organización, y refuerzo del círculo histórico de exclusión. Los falsos negativos aquí no son un error neutro: recaen sistemáticamente sobre el grupo que menos puede permitírselo.

**4. Compara el FPR de ambos grupos. ¿Qué dice sobre los estándares que aplica el modelo a cada uno?**

> - FPR grupo 0: **60.0%** (3 de 5 no calificados fueron aceptados).
> - FPR grupo 1: **0.0%** (ningún no calificado fue aceptado).
>
> El modelo aplica un **doble estándar**: es extremadamente permisivo con el grupo mayoritario (le aprueba incluso a más de la mitad de quienes no cumplen) y extremadamente estricto con el minoritario (no le perdona ni un solo error). Combinado con el TPR bajo del grupo 1 (37.5%), el patrón es inequívoco: al grupo 0 se le da el beneficio de la duda y al grupo 1 se le exige perfección. Ese desbalance es la firma clásica de un modelo sesgado.

**5. El accuracy global puede verse aceptable aunque el modelo sea inequitativo. ¿Por qué ocurre?**

> Porque el accuracy global (62.5%) es un **promedio agregado** que diluye el comportamiento por grupo y no distingue *a quién* acierta o falla. Un modelo puede mantener un número global "decente" acertando mucho en un grupo y fallando sistemáticamente en otro; los errores se compensan en el total. Además, el accuracy trata por igual todos los errores (FP y FN) e ignora cómo se distribuyen entre grupos y qué consecuencias tiene cada tipo. Por eso una auditoría de equidad **desagrega** las métricas: el agregado oculta justamente la discriminación que buscamos detectar.

**6. El dataset tiene solo 12 registros por grupo. ¿Qué limitaciones tiene esta auditoría?**

> Con 12 registros por grupo (24 en total) la auditoría es **ilustrativa, no concluyente**. Cada individuo pesa ~8% del grupo, así que una sola predicción cambia radicalmente las tasas; los intervalos de confianza serían enormes y las métricas, muy inestables. No permite análisis interseccional (combinaciones de atributos), no captura subgrupos ni casos límite, y es muy sensible al azar del muestreo. En producción se necesitarían cientos o miles de casos por grupo, estratificados y representativos de la población real, para que DI, SPD y EOD tengan validez estadística.

**7. Propón dos mitigaciones técnicas reales (no la simulada) y un riesgo de cada una.**

> **a) Re-ponderación / re-muestreo en el entrenamiento (pre-procesamiento):** reajustar los pesos o rebalancear los ejemplos para corregir el sesgo histórico del dataset antes de entrenar. *Riesgo:* puede degradar la exactitud global o introducir sobreajuste en el grupo minoritario si los datos son escasos, y no corrige sesgos que estén en las propias etiquetas.
>
> **b) Restricciones de equidad en el entrenamiento (in-processing), p. ej. _adversarial debiasing_ o _exponentiated gradient_ con una restricción de equal opportunity:** optimizar el modelo sujeto a que el TPR sea similar entre grupos. *Riesgo:* trade-off directo entre equidad y rendimiento, mayor complejidad de entrenamiento y mantenimiento, y la restricción debe elegirse con cuidado porque optimizar una métrica de equidad puede empeorar otra (imposibilidad de satisfacerlas todas a la vez).

### C. Conclusión: auditoría vs. Model Card

Compara los valores que escribiste en tu `MODEL_CARD.md` antes de ejecutar el script
(tus hipótesis) con los obtenidos en la auditoría.

| Métrica                     | Hipótesis (antes) | Obtenido (auditoría) | ¿Coincide? |
| --------------------------- | ----------------- | -------------------- | ---------- |
| Accuracy global             | ~85%              | 62.5%                | No         |
| Tasa de aprobación, grupo 0 | ~70%              | 75.0%                | Aprox.     |
| Tasa de aprobación, grupo 1 | ~65%              | 25.0%                | No         |
| Disparate Impact            | ~0.90             | 0.333                | No         |

> _Hipótesis previa (antes de ejecutar el script):_ partiendo solo de la descripción del taller —un modelo "ya entrenado y en uso"— la expectativa ingenua era un accuracy razonablemente alto (~85%) y tasas de aprobación parecidas entre grupos (DI cercano a 0.90, dentro de la regla 4/5), asumiendo que un modelo en producción sería aproximadamente justo.

**Conclusión (máx. 5 líneas):** ¿Qué te enseña esta comparación sobre llenar una Model Card con estimaciones frente a datos reales? ¿Es apto el modelo original para despliegue?

> La comparación muestra que las hipótesis "de intuición" fallaron en casi todo: el accuracy real (62.5%) fue mucho menor y, sobre todo, la brecha de aceptación (DI 0.333) es abismal frente al ~0.90 que supuse. Llenar una Model Card con estimaciones da una falsa sensación de equidad y puede legitimar un modelo discriminatorio; solo los datos reales de la auditoría revelan el sesgo. **El modelo original NO es apto para despliegue:** viola la regla de los 4/5 (DI 0.333 < 0.80), penaliza al grupo más calificado y aplica un doble estándar (FPR 60% vs 0%). Debe volver a desarrollo para mitigación.

---

## 📋 Model Card

1. Copia la plantilla: `cp MODEL_CARD_TEMPLATE.md MODEL_CARD.md`
2. Complétala con los resultados **reales** de tu auditoría sobre el dataset sesgado.
3. Verifica que la "Conclusión de la Auditoría" sea coherente con DI, SPD y EOD.

## ✅ Checklist de entrega

- [ ] TODO 1 y TODO 2 implementados; `pytest -v` sin _skipped_ ni fallos
- [ ] Evidencias 1 a 4 (salidas de consola) pegadas arriba
- [ ] 5 capturas de CI en `docs/img/` (3 en rojo, 2 en verde) y visibles en este README
- [ ] Sección Reflexión completa (A, B y C)
- [ ] `MODEL_CARD.md` completada
- [ ] Repositorio compartido con el docente

## 🧮 Rúbrica sugerida

| Criterio                                               | Puntos |
| ------------------------------------------------------ | ------ |
| Implementación correcta de SPD y EOD (tests en verde)  | 25     |
| Evidencias de CI (rojo y verde, con capturas legibles) | 25     |
| Reflexión: interpretación correcta de las métricas     | 25     |
| Model Card completa y coherente con la auditoría       | 25     |
