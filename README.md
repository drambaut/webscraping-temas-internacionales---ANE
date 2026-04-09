# 🛰️ Telecom Web Searcher

Herramienta Python para **buscar automáticamente** dentro de los sitios web de los principales reguladores de telecomunicaciones del mundo, sin usar APIs pagas ni Google. Usa **Playwright** (Chromium real) para simular un navegador humano.

---

## 📋 Organismos cubiertos

| ID      | Organismo                         | País/Región     | Idioma    |
|---------|-----------------------------------|-----------------|-----------|
| `itu`   | UIT / ITU                         | Internacional   | Inglés    |
| `cept`  | CEPT                              | Europa          | Inglés    |
| `ised`  | ISED Canada                       | Canadá          | Inglés    |
| `fcc`   | FCC                               | EE.UU.          | Inglés    |
| `ofcom` | Ofcom                             | Reino Unido     | Inglés    |
| `crt`   | CRT                               | México          | Español   |
| `anatel`| Anatel                            | Brasil          | Portugués |
| `subtel`| Subtel                            | Chile           | Español   |
| `acma`  | ACMA                              | Australia       | Inglés    |
| `msit`  | MSIT                              | Corea del Sur   | Coreano   |

---

## 📁 Estructura del proyecto

```
telecom_searcher/
│
├── config/
│   └── sites.json          ← Configuración de los 10 sitios (selectores CSS, URLs)
│
├── src/
│   ├── main.py             ← Punto de entrada (CLI)
│   ├── translator.py       ← Traducción gratuita (deep_translator)
│   ├── browser.py          ← Gestión de Playwright/Chromium
│   ├── scraper.py          ← Lógica de búsqueda y extracción de links
│   └── exporter.py         ← Exportación a Excel / CSV
│
├── logs/                   ← Logs de ejecución (se generan automáticamente)
├── output/                 ← Archivos de resultados (Excel/CSV)
│
├── requirements.txt
└── README.md
```

---

## ⚙️ Instalación (una sola vez)

### 1. Requisitos previos
- Python 3.9 o superior
- pip actualizado: `python -m pip install --upgrade pip`

### 2. Instalar dependencias Python

```bash
pip install -r requirements.txt
```

### 3. Instalar el navegador Chromium de Playwright

```bash
playwright install chromium
```

> ⚠️ Este paso descarga ~150MB. Solo se hace una vez.

---

## 🚀 Uso

### Búsqueda básica (en los 10 sitios, con traducción automática)

```bash
python src/main.py --query "spectrum"
```

### Buscar solo en sitios específicos

```bash
python src/main.py --query "spectrum management" --sites itu fcc ofcom
```

### Ver el navegador en acción (útil para depuración)

```bash
python src/main.py --query "5G" --headless false
```

### Sin traducción automática (usar la query tal como está)

```bash
python src/main.py --query "gestión del espectro" --no-translate
```

### Exportar en CSV en lugar de Excel

```bash
python src/main.py --query "spectrum" --format csv
```

### Especificar directorio de salida y máximo de links

```bash
python src/main.py --query "frequency" --output-dir resultados --max-links 100
```

### Ver los sitios disponibles

```bash
python src/main.py --list-sites
```

---

## 📊 Salida

Se genera un archivo Excel en `output/` con:

- **Hoja "Resultados"**: todos los links encontrados con columnas:
  - Organismo (ID), Nombre completo, Query usada, Idioma, URL (con hipervínculo), Texto del link, Fecha/Hora

- **Hoja "Resumen"**: conteo de links por organismo, con indicador verde/rojo.

---

## 🌐 Traducción automática

La herramienta traduce automáticamente tu query a:

| Idioma   | Código | Usado para       |
|----------|--------|------------------|
| Inglés   | `en`   | ITU, CEPT, ISED, FCC, Ofcom, ACMA |
| Español  | `es`   | CRT, Subtel, y como alternativa |
| Coreano  | `ko`   | MSIT (Corea)     |
| Portugués| `pt`   | Anatel (Brasil)  |

Usa **deep_translator** (Google Translate gratuito, sin API key).

---

## 🔧 Personalizar selectores CSS (`config/sites.json`)

Si un sitio no devuelve resultados, lo más probable es que el selector CSS
de la barra de búsqueda haya cambiado. Para actualizarlo:

1. Abre el sitio en Chrome.
2. Haz clic derecho sobre la barra de búsqueda → "Inspeccionar".
3. Copia el selector CSS del elemento `<input>`.
4. Actualiza `search_box` en `config/sites.json`.

Campos configurables por sitio:

```json
{
  "id": "fcc",
  "name": "FCC",
  "url": "https://www.fcc.gov/",
  "language": "en",
  "search_url": "https://www.fcc.gov/search/#q={query}",
  "search_method": "url",
  "search_box": "input[name='keys']",
  "search_button": "button[type='submit']",
  "results_container": ".view-content",
  "result_links": "a[href]",
  "result_link_filter": "fcc.gov",
  "wait_for": ".view-content"
}
```

| Campo               | Descripción |
|---------------------|-------------|
| `search_url`        | URL de búsqueda directa. `{query}` se reemplaza con la búsqueda codificada. |
| `search_method`     | `"url"` (preferido) o `"form"` (formulario interactivo). |
| `search_box`        | Selector CSS de la caja de búsqueda (separar alternativas con `,`). |
| `search_button`     | Selector CSS del botón de búsqueda. |
| `result_link_filter`| Solo guardar links que contengan este texto (p.ej. el dominio). |
| `wait_for`          | Selector que indica que los resultados ya cargaron. |

---

## 🛠️ Solución de problemas

### "No se encontraron resultados"

1. Ejecuta con `--headless false` para ver qué pasa visualmente.
2. Revisa el log en `logs/` para ver mensajes de error específicos.
3. El selector CSS pudo haber cambiado → actualiza `sites.json`.
4. El sitio puede bloquear bots → aumenta los tiempos de espera en `browser.py`.

### "Error de timeout"

- Aumenta el timeout en `browser.py` → función `safe_goto()`, parámetro `timeout`.
- Verifica tu conexión a internet.

### El sitio en coreano (MSIT) no funciona

- MSIT puede requerir la URL de búsqueda actualizada.
- Entra manualmente a `https://www.msit.go.kr/` y busca algo.
- Copia la URL resultante y úsala como template en `search_url` de `sites.json`.

### "playwright install chromium" falla

- En Linux puede requerir dependencias del sistema:
  ```bash
  playwright install-deps chromium
  ```

---

## 📅 Uso mensual recomendado

```bash
# Mes a mes, solo ejecuta:
python src/main.py --query "TU TEMA DEL MES"

# Los resultados quedan en output/ con fecha y hora en el nombre
```

---

## 📄 Licencia

Herramienta de uso interno. Gratuita y de código abierto.
