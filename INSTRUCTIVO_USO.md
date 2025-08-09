
# 📘 Instructivo de uso — Paquete de Python (competencia_tools / competencia-mexico)

Guía rápida y clara para instalar, importar y utilizar las funciones del paquete que incluye:
- **Índices de competencia** (IHH, dominancia y variaciones por fusión)
- **Precios** (aumentos, sobreprecios y pérdida por colusión)
- **Gráficas** para asuntos resueltos (con CSV empacado)
- **Búsqueda de artículos** (con CSV empacado)
- **RAG con DeepSeek** (índice FAISS empacado)

> Nota: En TestPyPI tu proyecto puede llamarse `competencia-mexico`, mientras que el módulo se importa como `competencia_tools`. Si tu import es distinto (p. ej. `competencia_mexico`), ajusta los ejemplos.

---

## 1) Instalación

### 1.1 Desde TestPyPI (recomendado para pruebas)
```bash
pip install -i https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple competencia-mexico
```
El `--extra-index-url` permite resolver dependencias desde PyPI cuando no estén en TestPyPI.

### 1.2 Desde PyPI (cuando publiques estable)
```bash
pip install competencia-mexico
```
o el nombre que elijas en PyPI.

### 1.3 Requisitos principales
- Python ≥ 3.8
- Dependencias: `pandas`, `numpy`, `matplotlib`, `requests`, `langchain-community`, `sentence-transformers`, `faiss-cpu`  
  (Se instalan automáticamente con `pip`).

---

## 2) Importación

```python
# Opción típica (módulo):
import competencia_tools as ct

# Si tu paquete se importa con otro nombre, por ejemplo:
# import competencia_mexico as ct
```

Comprueba la versión:
```python
print(ct.__version__)
```

---

## 3) Funcionalidades y ejemplos

### 3.1 Índices de competencia (interactivos)
Funciones: `calcular_ihh_interactivo`, `calcular_id_interactivo`, `variacion_ihh_interactiva`, `variacion_dominancia_interactiva`.

```python
# IHH (sigue los prompts en consola / notebook):
ct.calcular_ihh_interactivo()

# Índice de dominancia:
ct.calcular_id_interactivo()

# Variación de IHH por fusión:
ct.variacion_ihh_interactiva()

# Variación del índice de dominancia por fusión:
ct.variacion_dominancia_interactiva()
```

**Sugerencias**  
- Si eliges “d” (DataFrame) te pedirá ruta CSV con columnas `empresa`, `participacion`.  
- Si eliges “m” (manual), captura número de empresas y sus valores o porcentajes.  
- Los resultados se imprimen en consola; en variaciones se muestra antes/después y la diferencia.

---

### 3.2 Precios: aumentar, sobreprecio y pérdida por colusión
Funciones: `aumentar_precios`, `calcular_sobreprecio`, `perdida_colusion`.

```python
import pandas as pd

# Datos de ejemplo
df = pd.DataFrame({
    "Producto": ["A", "B", "C"],
    "Precio2024": [100, 200, 150],
    "Precio2025": [110, 210, 155],
}).set_index("Producto")

# Aumentar precios 10% (sin gráficos)
df_aum = ct.aumentar_precios(df, ["Precio2024", "Precio2025"], 0.10, graficar=False)

# Calcular sobreprecio entre dos columnas
df_sp = ct.calcular_sobreprecio(df, "Precio2024", "Precio2025")

# Pérdida por colusión (genera gráfico)
area_A, area_B = ct.perdida_colusion(
    precio_competencia=100,
    precio_colusion=120,
    cantidad_competencia=1000,
    elasticidad=-1.2
)
```

---

### 3.3 Gráficas de asuntos (CSV empacado)
Función: `graficar_y_resumir_asuntos_interactiva`  
- Lee el archivo **`estadisticas_final.csv`** empacado dentro del paquete (`competencia_tools/data`).  
- Requiere columnas: `FechaResolucion`, `Rubro`, `Decision`.

```python
ct.graficar_y_resumir_asuntos_interactiva()
# El flujo te pedirá:
# - Rubro (p. ej. DE, IO, OPN)
# - Desagregación (mes/año)
# - Si deseas desglosar por tipo de decisión
# Muestra la gráfica y una tabla resumen.
```

---

### 3.4 Búsqueda de artículos (CSV empacado)
Función: `buscar_articulos_csv`  
- Lee **`articulos_final.csv`** empacado en `competencia_tools/data`.  
- Busca una palabra clave en la columna `texto` y muestra títulos; opcionalmente imprime el texto con resaltado (ANSI).

```python
# Busca la palabra "competencia"
df_hits = ct.buscar_articulos_csv("competencia")
```

---

### 3.5 RAG con DeepSeek (índice FAISS empacado)
Función: `consultar_con_rag_deepseek`  
- El índice FAISS debe estar en `competencia_tools/data/faiss_index/` (mismos archivos creados por `FAISS.save_local`).  
- Requiere **API Key** de DeepSeek y conexión a internet.

```python
ct.consultar_con_rag_deepseek()
# Prompts:
# - API Key de DeepSeek
# - Pregunta
# Devuelve una respuesta y lista de fuentes (metadatos del índice).
```

**Notas RAG**  
- Si el índice es muy grande (>50MB), considera distribuirlo fuera del wheel y descargarlo en tiempo de ejecución.  
- El modelo de embeddings por defecto: `"sentence-transformers/all-MiniLM-L6-v2"` (se descarga automáticamente).

---

## 4) Uso en Google Colab

### 4.1 Instalar desde TestPyPI
```python
!pip install -i https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple competencia-mexico
```

### 4.2 Importar y probar
```python
import competencia_tools as ct
print(ct.__version__)
```

> Si no puedes importar, revisa el nombre exacto del módulo (puede variar según cómo lo hayas publicado).

---

## 5) Problemas comunes (FAQ)

- **No encuentro el CSV empacado.**  
  Asegúrate de que `MANIFEST.in` y `pyproject.toml`/`setup.py` incluyan:
  - `include competencia_tools/data/*.csv`
  - `include competencia_tools/data/faiss_index/*`
  - `include_package_data = True` y `package_data` para esos patrones.

- **Error al cargar FAISS / índice no encontrado.**  
  Verifica la ruta interna usada por la función y que los archivos (`.pkl`, `.faiss`, JSONs) estén presentes.  
  Si el índice es muy pesado, considera descargarlo en runtime y apuntar la función a esa ruta.

- **Colab no muestra gráficos.**  
  Ejecuta en una celda previa:  
  ```python
  %matplotlib inline
  ```

- **Reinstalar después de actualizar el paquete.**  
  Reinicia el runtime de Colab y vuelve a instalar.

---

## 6) Versionado y publicación

- Para publicar en **TestPyPI** con GitHub Actions, crea un tag tipo `test-v0.1.0` y empuja el tag.  
- Para **PyPI**, usa tags tipo `v0.1.0`.  
- Revisa la pestaña **Actions** del repo para ver el estado del pipeline.

---

## 7) Contacto
Autor: **Iván Paredes Reséndiz**  
Soporte / dudas: abre un *issue* en tu repositorio o agrega tu correo aquí.

---

¡Listo! Con esto deberías poder instalar, importar y usar el paquete en local, Colab o servidores.
