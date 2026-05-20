{
  "nbformat": 4,
  "nbformat_minor": 0,
  "metadata": {
    "colab": {
      "provenance": [],
      "authorship_tag": "ABX9TyOQkU76saAWvoWsZCUaZ3zK",
      "include_colab_link": true
    },
    "kernelspec": {
      "name": "python3",
      "display_name": "Python 3"
    },
    "language_info": {
      "name": "python"
    }
  },
  "cells": [
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "view-in-github",
        "colab_type": "text"
      },
      "source": [
        "<a href=\"https://colab.research.google.com/github/GinnaGomez09/proyecto_aplicado_javeriana/blob/main/notebooks/02_reduccion_dataset_recetas.md\" target=\"_parent\"><img src=\"https://colab.research.google.com/assets/colab-badge.svg\" alt=\"Open In Colab\"/></a>"
      ]
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "uB8LrNNdx5eW"
      },
      "outputs": [],
      "source": [
        "# ============================================================\n",
        "# 02B_REDUCCION_DATASET.ipynb\n",
        "# Reducción y depuración del dataset antes del pipeline NLP\n",
        "# ============================================================\n",
        "\n",
        "import pandas as pd\n",
        "from pathlib import Path\n",
        "\n",
        "# ------------------------------------------------------------\n",
        "# 1. Cargar dataset original\n",
        "# ------------------------------------------------------------\n",
        "\n",
        "input_path = \"data/raw/recetas/recetas_ingredientes.csv\"\n",
        "\n",
        "recetas = pd.read_csv(input_path)\n",
        "\n",
        "print(\"Dimensiones originales:\", recetas.shape)\n",
        "print(\"Número original de recetas:\", recetas[\"receta_uuid\"].nunique())"
      ]
    },
    {
      "cell_type": "code",
      "source": [
        "# ------------------------------------------------------------\n",
        "# 2. Seleccionar únicamente columnas útiles\n",
        "# ------------------------------------------------------------\n",
        "\n",
        "columnas_utiles = [\n",
        "    \"receta_uuid\",\n",
        "    \"receta_titulo\",\n",
        "    \"receta_url\",          # Se conserva temporalmente para filtrar la fuente\n",
        "    \"ingrediente_id\",\n",
        "    \"ingrediente_linea\",\n",
        "    \"cantidad_original\",\n",
        "    \"cantidad_conv\",\n",
        "    \"unidad\",\n",
        "    \"ingrediente_nombre\"\n",
        "]\n",
        "\n",
        "recetas_reducidas = recetas[columnas_utiles].copy()\n",
        "\n",
        "print(\"Columnas seleccionadas:\")\n",
        "print(recetas_reducidas.columns.tolist())"
      ],
      "metadata": {
        "id": "m_rdjRxz5Nh1"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "# ------------------------------------------------------------\n",
        "# 3. Identificar dominio/fuente de las recetas\n",
        "# ------------------------------------------------------------\n",
        "\n",
        "recetas_reducidas[\"fuente\"] = (\n",
        "    recetas_reducidas[\"receta_url\"]\n",
        "    .astype(str)\n",
        "    .str.extract(r\"https?://([^/]+)\")\n",
        ")\n",
        "\n",
        "print(\"Distribución de fuentes:\")\n",
        "display(\n",
        "    recetas_reducidas[\"fuente\"]\n",
        "    .value_counts()\n",
        ")"
      ],
      "metadata": {
        "id": "RRP9pGtj5lRj"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "# ------------------------------------------------------------\n",
        "# 4. Conservar únicamente recetas mejor estructuradas\n",
        "# ------------------------------------------------------------\n",
        "# Se conservan recetas de:\n",
        "# www.mycolombianrecipes.com\n",
        "#\n",
        "# Razones:\n",
        "# - Mejor estructura ingrediente-por-fila\n",
        "# - Menor cantidad de ruido textual\n",
        "# - Recetas colombianas relevantes\n",
        "# - Más fáciles de procesar con recursos limitados\n",
        "# ------------------------------------------------------------\n",
        "\n",
        "recetas_reducidas = recetas_reducidas[\n",
        "    recetas_reducidas[\"fuente\"] == \"www.mycolombianrecipes.com\"\n",
        "].copy()\n",
        "\n",
        "print(\"Dimensiones después del filtrado por fuente:\", recetas_reducidas.shape)\n",
        "print(\"Número de recetas:\", recetas_reducidas[\"receta_uuid\"].nunique())"
      ],
      "metadata": {
        "id": "MKqs-8_i5pWM"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "# ------------------------------------------------------------\n",
        "# 5. Eliminar registros problemáticos\n",
        "# ------------------------------------------------------------\n",
        "\n",
        "# Eliminar nulos importantes\n",
        "recetas_reducidas = recetas_reducidas.dropna(\n",
        "    subset=[\n",
        "        \"receta_uuid\",\n",
        "        \"receta_titulo\",\n",
        "        \"ingrediente_linea\",\n",
        "        \"ingrediente_nombre\"\n",
        "    ]\n",
        ").copy()\n",
        "\n",
        "# Eliminar duplicados completos\n",
        "recetas_reducidas = recetas_reducidas.drop_duplicates().copy()\n",
        "\n",
        "# Eliminar líneas excesivamente largas\n",
        "# (normalmente corresponden a ingredientes concatenados\n",
        "# o recetas mal estructuradas)\n",
        "\n",
        "recetas_reducidas = recetas_reducidas[\n",
        "    recetas_reducidas[\"ingrediente_linea\"]\n",
        "    .astype(str)\n",
        "    .str.len() <= 180\n",
        "].copy()\n",
        "\n",
        "print(\"Dimensiones después de limpieza:\", recetas_reducidas.shape)\n",
        "print(\"Número de recetas:\", recetas_reducidas[\"receta_uuid\"].nunique())"
      ],
      "metadata": {
        "id": "_rhmc6S55uXL"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "# ------------------------------------------------------------\n",
        "# 6. Conservar recetas con cantidad razonable de ingredientes\n",
        "# ------------------------------------------------------------\n",
        "# Se eliminan:\n",
        "# - recetas demasiado pequeñas\n",
        "# - recetas extremadamente largas/problemáticas\n",
        "# ------------------------------------------------------------\n",
        "\n",
        "conteo_ingredientes = (\n",
        "    recetas_reducidas\n",
        "    .groupby(\"receta_uuid\")\n",
        "    .size()\n",
        "    .reset_index(name=\"n_ingredientes\")\n",
        ")\n",
        "\n",
        "display(conteo_ingredientes.describe())\n",
        "\n",
        "# Mantener recetas entre 3 y 25 ingredientes\n",
        "\n",
        "recetas_validas = conteo_ingredientes[\n",
        "    conteo_ingredientes[\"n_ingredientes\"].between(3, 25)\n",
        "][\"receta_uuid\"]\n",
        "\n",
        "recetas_reducidas = recetas_reducidas[\n",
        "    recetas_reducidas[\"receta_uuid\"].isin(recetas_validas)\n",
        "].copy()\n",
        "\n",
        "print(\"Dimensiones finales:\", recetas_reducidas.shape)\n",
        "print(\"Número final de recetas:\", recetas_reducidas[\"receta_uuid\"].nunique())"
      ],
      "metadata": {
        "id": "i66n3K9S51Av"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "# ------------------------------------------------------------\n",
        "# 7. Eliminar columnas ya no necesarias\n",
        "# ------------------------------------------------------------\n",
        "\n",
        "recetas_reducidas = recetas_reducidas.drop(\n",
        "    columns=[\n",
        "        \"receta_url\",\n",
        "        \"fuente\"\n",
        "    ]\n",
        ")\n",
        "\n",
        "print(\"Columnas finales:\")\n",
        "print(recetas_reducidas.columns.tolist())"
      ],
      "metadata": {
        "id": "HGPR0_B254--"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "# ------------------------------------------------------------\n",
        "# 8. Vista final del dataset reducido\n",
        "# ------------------------------------------------------------\n",
        "\n",
        "recetas_reducidas.head()"
      ],
      "metadata": {
        "id": "rfDkPgU35-my"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "# ------------------------------------------------------------\n",
        "# 9. Guardar dataset reducido\n",
        "# ------------------------------------------------------------\n",
        "\n",
        "Path(\"data/interim\").mkdir(parents=True, exist_ok=True)\n",
        "\n",
        "output_path = \"data/interim/recetas_reducidas.csv\"\n",
        "\n",
        "recetas_reducidas.to_csv(\n",
        "    output_path,\n",
        "    index=False,\n",
        "    encoding=\"utf-8-sig\"\n",
        ")\n",
        "\n",
        "print(\"Dataset reducido guardado en:\")\n",
        "print(output_path)\n",
        "\n",
        "print(\"\\nResumen final:\")\n",
        "print(\"Filas:\", recetas_reducidas.shape[0])\n",
        "print(\"Recetas:\", recetas_reducidas[\"receta_uuid\"].nunique())"
      ],
      "metadata": {
        "id": "hsmURY646GO6"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "source": [
        "# Reducción y depuración del dataset\n",
        "\n",
        "Antes de iniciar las etapas avanzadas del pipeline NLP, se realiza una reducción y depuración del dataset original con el fin de:\n",
        "\n",
        "- Disminuir el ruido textual.\n",
        "- Eliminar columnas irrelevantes para el proyecto.\n",
        "- Conservar únicamente recetas con mejor estructura.\n",
        "- Reducir el costo computacional.\n",
        "- Facilitar la validación y desarrollo del pipeline.\n",
        "\n",
        "Se seleccionan únicamente recetas provenientes de fuentes con estructura ingrediente-por-fila, eliminando registros problemáticos, recetas excesivamente largas y columnas vacías relacionadas con la integración futura con la TCAC."
      ],
      "metadata": {
        "id": "q5QwhRqD6RR0"
      }
    }
  ]
}