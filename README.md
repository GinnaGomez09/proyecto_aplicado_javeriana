# Predicción Nutricional de Recetas Colombianas mediante NLP y ML

## Objetivo
Construir un pipeline NLP híbrido para extraer ingredientes, cantidades y unidades de recetas colombianas, vincularlos con la TCAC y calcular el perfil nutricional de cada receta.

## Estado del proyecto
Fase 1: Comprensión y preparación de datos

## Flujo metodológico
1. EDA de recetas
2. Limpieza y normalización
3. Tokenización y lematización
4. NER culinario
5. Estandarización de ingredientes
6. Conversión de unidades
7. Integración con TCAC
8. Cálculo nutricional
9. Validación de calidad

## Tecnologías
- Python
- pandas
- spaCy
- NLTK
- RapidFuzz
- regex
- Jupyter Notebook

## Datos

Los datos utilizados en este proyecto se encuentran en:

- data/raw/recetas/recetas_ingredientes.csv
- data/raw/tcac/tcac.csv

Estos corresponden a:
- Corpus de recetas colombianas
- Tabla de Composición de Alimentos Colombianos (TCAC)
