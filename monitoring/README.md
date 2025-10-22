Dokku Apps → Docker Logs → Promtail → Loki → Grafana
(generate)  (stdout/err)  (collect)  (store) (visualize)

---

## 1. **Promtail** - Colector de logs (Shipper)

**Lo que hace:**
- Monitore los contenedores de docker del sistema
- Lee los archivos de logs y los transmite desde docker
- Descubre los contenedores automaticamente usando las etiquetas (labels) de docker
- Añade metada (labels) a los logs como app name, container name, etc.
- Entrega los logs a loki

**Como funciona:**
1. Promtail se conecta al socket de Docker (/var/run/docker.sock)
2. Descubre todos los contenedores que estan corriendo
3. Lee los logs desde /var/lib/docker/containers/
4. Aplica las reglas de etiquetas (labels) (extracts Dokku app name, process type, etc.)
5. Envia los logs via HTTP a la API de Loki

**Imaginarlo como:** Un camión de entregas que recoge los logs de varias fuentes y los entrega al depósito de Loki

**En nuestro setup:**
- Lee los logs de todos los contenedores
- Específicamente etiqueta las apps de Dokku usando la etiquet `com.dokku.app-name`
- Y lo envía `http://loki:3100/loki/api/v1/push`

---

## 2. **Loki** - El alamacenamiento de logs (Database)


**Lo que hace:**
- Recibe los logs desde Promtail
- Almacena los logs eficientemente (como una Base de datos de time-series)
- Indexa solo la metadata (labels), No el contenido del log
- Provee una API para la busqueda de logs (Queries)


**Como funciona:**
1. Recibe la transmisión de logs de Promtail
2. Comprime y almacena el contenido del log en trozos (chunks)
3. Crea indices basados en las etiquetas (app, container, stream, etc.)
4. Sirve las solicitudes mediante una API HTTP a grafana

**Conceptos clave - Indexado basado en etiquetas (Label):**
- Logs tradicionales: Indexa TODO (lento, costoso)
- Loki: Sólo indexa las etiquetas (rápido, barato)
- La busqueda del contenido ocurre en el momento del query usando patrones al estilo grep

**Ejemplo:**

Log entry: {app="backend", stream="stdout"} "Error: Database connection failed"

Indexed:     app="backend", stream="stdout"  ← Fast to search
Not indexed: "Error: Database connection failed"  ← Searched on-demand

**Imaginarlo como:** Una biblioteca que organiza libros (logs) por etiquetas de categorias (labels) pero no indexa cada palabra del libro. Cuando se busca, encuentra la categoria adecuada rapidamente, y escanea el contenido.

---

## 3. **Grafana** - La capa de Visualización (UI)


**Lo que hace:**
- Provee una interfaz Web para solicitar y visualizar los logs
- Se conecta a Loki como una fuente de datos (data source)
- Permite escribie solicitudes LogQL
- Crea Dashboards, alertas y visualizaciones

**Como funciona:**
1. El usuario abre grafana en el navegador
2. El usuario escribe solicitudes tipo LogQL (e.g., {app="backend"} |= "error")
3. Grafana lo envía a la API de Loki
4. Loki Retorna  los logs que coincidieron con la busqueda
5. Grafana muestra los resultados como logs, gráficos, tablas.

**Imaginarlo como:** El panel de control/dashboard en dónde haces preguntas y ves las respuestas en una forma visual.



## 4. **Diagrama visual**


```
┌─────────────────────────────────────────┐
│  Your VM                                │
│                                         │
│  ┌────────────────┐                     │
│  │ Dokku Apps     │                     │
│  │ (Containers)   │                     │
│  │  - backend     │                     │
│  │  - frontend    │                     │
│  │  - Database    │                     │
│  └───────┬────────┘                     │
│          │ logs (stdout/stderr)         │
│          ↓                              │
│  ┌────────────────┐                     │
│  │ Docker Engine  │                     │
│  │ /var/lib/      │                     │
│  │  docker/       │                     │
│  │  containers/   │                     │
│  └───────┬────────┘                     │
│          │ reads                        │
│          ↓                              │
│  ┌────────────────┐                     │
│  │ Promtail       │                     │
│  │ (Collector)    │                     │
│  │ - Discovers    │                     │
│  │ - Labels       │                     │
│  │ - Ships        │                     │
│  └───────┬────────┘                     │
│          │ HTTP push                    │
│          ↓                              │
│  ┌────────────────┐                     │
│  │ Loki           │                     │
│  │ (Storage)      │                     │
│  │ - Receives     │                     │
│  │ - Indexes      │                     │
│  │ - Stores       │                     │
│  │                │                     │
│  │ /loki-data/    │                     │
│  └───────┬────────┘                     │
│          │ HTTP query                   │
│          ↓                              │
│  ┌────────────────┐                     │
│  │ Grafana        │                     │
│  │ (UI)           │                     │
│  │ - Queries      │                     │
│  │ - Visualizes   │                     │
│  │ - Dashboards   │                     │
│  └────────────────┘                     │
│          ↑                              │
└──────────┼──────────────────────────────┘
           │ HTTP (port 3000)
           │
      ┌────┴────┐
      │ Browser │
      │  (You)  │
      └─────────┘
```


## 5. **Permisos de las carpetas**

Arbol de archivos

```
.
├── configs
│   ├── grafana-provisioning
│   │   └── datasources
│   │       └── datasources.yml
│   ├── loki-config.yml
│   ├── prometheus.yml
│   └── promtail-config.yml
├── docker-compose.yml
├── grafana-data
├── loki-data
├── prometheus-data
├── promtail-data
└── README.md

```

```
sudo chown -R 10001:10001 loki-data
sudo chown -R 472:472 grafana-data
sudo chmod -R 755 promtail-data
sudo chown -R 65534:65534 prometheus-data
sudo chown -R 472:472 configs/grafana-provisioning
```

