# Extracción de Datos: Premier League y Fantasy Premier League (2025/2026)

Este documento explica paso a paso cómo replicar el proceso de extracción de datos de la **Fantasy Premier League (FPL)** y de **football-data.co.uk** para la temporada actual (2025/2026).

---

## 1. Requisitos Previos

Para ejecutar el código de extracción en otro equipo o entorno, necesitas tener instalado Python y la librería `requests`.

1. **Instalar Python:** Asegúrate de tener Python 3.7 o superior instalado.
2. **Crear un entorno virtual (Opcional pero Recomendado):**
   ```console
   python3 -m venv venv
   source venv/bin/activate  # En Mac/Linux
   venv\Scripts\activate     # En Windows
   ```
3. **Instalar la librería `requests`:**
   ```console
   pip install requests pandas
   ```

---

## 2. El Código de Extracción (Script)

El script `extract_data.py` se encarga de descargar:
1. El archivo JSON masivo de la API de la Fantasy Premier League que contiene a todos los jugadores, sus precios, puntos y popularidad.
2. El archivo CSV con el registro de todos los partidos y cuotas (1X2, Over/Under, etc.) de la temporada 2025/2026 desde `football-data.co.uk`.

Crea un archivo llamado `extract_data.py` y copia el siguiente código:

```python
import requests
import json
import os

# Obtiene la ruta absoluta de la carpeta donde se encuentra este script
SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))

def download_fpl_data():
    print("Descargando datos de la Fantasy Premier League...")
    url = "https://fantasy.premierleague.com/api/bootstrap-static/"
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)'
    }
    
    try:
        response = requests.get(url, headers=headers)
        response.raise_for_status()
        data = response.json()
        
        # Guarda el JSON masivo asegurando que cae en la carpeta premier league
        filename = os.path.join(SCRIPT_DIR, 'fpl_data_2025_2026.json')
        with open(filename, 'w', encoding='utf-8') as f:
            json.dump(data, f, indent=4, ensure_ascii=False)
            
        print(f"✅ Éxito: Datos FPL descargados. ({len(data['elements'])} jugadores)")
    except Exception as e:
        print(f"❌ Error al descargar datos de FPL: {e}")

def download_football_data():
    print("\nDescargando datos CSV de football-data.co.uk (Premier League 25/26)...")
    # Es crucial utilizar http:// en lugar de https:// para eludir los errores 
    # de los certificados de seguridad "SSL handshake error" 
    # que bloquean la extracción en algunos sistemas operativos
    url = "http://www.football-data.co.uk/mmz4281/2526/E0.csv"
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)'
    }
    
    try:
        # Se agrega permitere_redirects y un timeout generoso
        response = requests.get(url, headers=headers, timeout=15, allow_redirects=True)
        response.raise_for_status()
        
        # Guardamos el CSV explícitamente en el mismo directorio del script
        filename = os.path.join(SCRIPT_DIR, 'premier_league_25_26_results.csv')
        with open(filename, 'wb') as f:
            f.write(response.content)
            
        print(f"✅ Éxito: CSV guardado sin inconvenientes.")
        
    except requests.exceptions.ConnectionError:
        print("❌ Fallo en la conexión. Descarga manual aquí:")
        print(f"   -> {url}")
    except Exception as e:
        print(f"❌ Error: {e}")

if __name__ == "__main__":
    download_fpl_data()
    download_football_data()
```

---

## 3. ¿Cómo Ejecutarlo y Ver los Resultados?

Para correr tu script y descargar los archivos, debes ir a la terminal y asegurarte de estar en la carpeta donde tienes guardado `extract_data.py`. Allí, ejecuta:

```console
python3 extract_data.py
```

### Resultados Esperados
1. **Archivo JSON (`fpl_data_2025_2026.json`)**: Verás un nuevo archivo creado. ¡Ese es el archivo con más de 800 jugadores! La data de interés se encuentra concretamente dentro del array asociativo `"elements"`.
2. **Archivo CSV (`premier_league_25_26_results.csv`)**: Un archivo plano que puedes abrir en Excel con historial de disparos y cuotas.

> **⚠️ Importante sobre `football-data.co.uk`:** Esta web utiliza servidores que restringen de forma estricta las peticiones por Python limitando las IPs. Si el script arroja un error de *Connection reset by peer* en la descarga del CSV, el script lo capturará e indicará la URL (ej. `https://www.football-data.co.uk/mmz4281/2526/E0.csv`) para que te descargues el CSV abriéndola de forma tradicional a través del navegador web (Chrome/Safari). 

---

## 4. Estructura de la Data Extraída (FPL)

Una vez que tengas el `fpl_data_2025_2026.json`, notarás que es gigantesco. Si en un futuro quieres cargar esa data de jugadores usando `pandas` en Python, el código sería así:

```python
import json
import pandas as pd

# 1. Cargamos el JSON extraído localmente
with open('fpl_data_2025_2026.json', 'r', encoding='utf-8') as f:
    data = json.load(f)

# 2. Los jugadores están en la llave 'elements'
players_data = data['elements']

# 3. Lo llevamos a Pandas DataFrame
df_players = pd.DataFrame(players_data)

# Visualizamos columnas clave
cols = ['first_name', 'second_name', 'total_points', 'now_cost']
print(df_players[cols].sort_values(by='total_points', ascending=False).head())
```

Con este paso a paso, cualquier integrante de tu equipo o de otro ordenador podrá replicar todo y tener la información normalizada para la temporada actual de manera rápida y sin problemas.
