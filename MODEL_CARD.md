# Model Card: Clasificador de Elegibilidad (modelo auditado)

> Plantilla basada en _Model Cards for Model Reporting_ (Mitchell et al., 2019).
> Documenta el **modelo original** evaluado con `scripts/calculate_audit.py` sobre
> `data/model_predictions.csv` (dataset sesgado).

## Model Details

- **Developed by:** Nicolás Ducuara
- **Model Date:** 2026-11-13
- **Model Version:** v1.0.0
- **Model Type:** Clasificador binario de elegibilidad (auditoría basada solo en predicciones; no se reentrena)
- **License:** MIT
- **Contact:** https://github.com/NicolasDucuara/Taller-Ethical-IA

## Intended Use

- **Primary Intended Uses:** Automatización inicial del filtrado de solicitudes (aprobar/rechazar) como apoyo a la decisión humana.
- **Primary Intended Users:** Analistas de riesgo / evaluadores que revisan solicitudes.
- **Out-of-Scope Uses:** Decisiones finales automáticas sin supervisión humana, decisiones judiciales, diagnósticos médicos definitivos, o cualquier exclusión total de personas sin revisión. **No apto para despliegue en su estado actual** (ver auditoría).

## Training & Evaluation Data

- **Dataset Overview:** Conjunto de 24 predicciones (`id`, `true_label`, `predicted_label`, `protected_group`), 12 registros por grupo. Muestra pequeña, con fines de auditoría ilustrativa; no representa volumen de producción.
- **Protected Attributes:** `protected_group` (atributo sensible binario).
- **Group Definition:** `0` = grupo mayoritario; `1` = grupo minoritario.

## Quantitative Performance & Fairness Audit

_Resultados obtenidos tras ejecutar `scripts/calculate_audit.py` sobre el dataset sesgado:_

| Métrica                                    | Valor Obtenido | Umbral Objetivo |
| :----------------------------------------- | :------------- | :-------------- |
| **Accuracy Global**                        | 62.5%          | $> 85\%$        |
| **Tasa de Aprobación (Grupo Mayoritario)** | 75.0%          | -               |
| **Tasa de Aprobación (Grupo Minoritario)** | 25.0%          | -               |
| **TPR / Recall (Grupo Mayoritario)**       | 85.7%          | -               |
| **TPR / Recall (Grupo Minoritario)**       | 37.5%          | -               |
| **FPR (Grupo Mayoritario)**                | 60.0%          | -               |
| **FPR (Grupo Minoritario)**                | 0.0%           | -               |
| **Disparate Impact Ratio (DI)**            | 0.333          | **$\ge 0.80$**  |
| **Statistical Parity Difference (SPD)**    | −0.500         | ~0              |
| **Equal Opportunity Difference (EOD)**     | −0.482         | ~0              |

- **Resultado de la regla de los 4/5:** [ ] Aprobado [x] Violación detectada
- **Conclusión de la Auditoría:** El modelo **viola** la regla de los cuatro quintos (DI = 0.333 < 0.80). Acepta al grupo minoritario a una tasa tres veces menor pese a que ese grupo está *más* calificado (66.7% vs 58.3% de `true_label = 1`). Debe volver a desarrollo para mitigación de sesgos.

## Ethical Considerations & Limitations

- **Known Biases:** Impacto dispar contra el grupo minoritario. El modelo aplica un doble estándar: FPR 60.0% para el grupo 0 (muy permisivo) frente a 0.0% para el grupo 1 (exige perfección), y rechaza a 5 de 8 personas calificadas del grupo minoritario (FN = 5). La brecha no se explica por mérito.
- **Limitations of this Audit:** Muestra muy pequeña (12 por grupo, 24 en total): métricas inestables y sin validez estadística concluyente. Un solo atributo protegido, sin análisis interseccional ni subgrupos. Resultados sensibles al azar del muestreo.
- **Mitigations Applied / Recommended:** _Simulada en el taller:_ ajuste del umbral de decisión del grupo minoritario (lleva DI a 0.889). _Recomendadas reales:_ (a) re-ponderación/re-muestreo en entrenamiento para corregir el sesgo histórico; (b) restricciones de equidad in-processing (p. ej. _exponentiated gradient_ con equal opportunity); en ambos casos, mantener supervisión humana y re-auditar antes de desplegar.

## Sign-off

- **Auditor:** Nicolás Ducuara
- **Audit Date:** 2026-11-13
- **Decision:** [ ] Aprobado para despliegue [x] Devuelto a desarrollo
