# 🌌 TP Sistemas Operativos - 1C2025
### *Episode III: Revenge of the Cth / Episode IX: The Rise of Gopher*

Trabajo Práctico Cuatrimestral de la cátedra de **Sistemas Operativos (UTN FRBA)**, desarrollado íntegramente en **Go (Golang)**. El proyecto consiste en la simulación de un sistema operativo distribuido compuesto por cuatro módulos independientes que se comunican entre sí mediante peticiones HTTP / conexiones TCP/IP.

---

## 🏗️ Arquitectura del Sistema

El sistema está dividido en 4 módulos principales más una biblioteca de utilidades compartidas (`utils`), gestionados mediante un **Go Workspace (`go.work`)**:

### 1. 🧠 Kernel (`ssoo-kernel`)
Encargado de la gestión y planificación de procesos e interfaces de Entrada/Salida:
* **Modelo de Planificación de 7 Estados:** `NEW`, `READY`, `EXEC`, `BLOCKED`, `SUSP_READY`, `SUSP_BLOCKED` y `EXIT`.
* **Planificador de Largo y Mediano Plazo:** Control del grado de multiprogramación dependiendo del espacio disponible en Memoria y suspensión de procesos mediante *timers* configurables (`TIEMPO_SUSPENSION`). Algoritmos soportados:
  * `FIFO` (First In, First Out)
  * `PMCP` (Proceso Más Chico Primero)
* **Planificador de Corto Plazo:** Soporta ejecución multiprocesador (múltiples instancias de CPU) con los algoritmos:
  * `FIFO`
  * `SJF` (Shortest Job First - Sin desalojo)
  * `SRT` (Shortest Remaining Time - Con desalojo mediante interrupciones a CPU)
* **Gestión de Syscalls e I/O:** Manejo de llamadas al sistema y administración de múltiples instancias de dispositivos de Entrada/Salida conectadas y desconectadas dinámicamente.

### 2. ⚙️ CPU (`ssoo-cpu`)
Simula el ciclo de instrucción del procesador y la traducción de direcciones de memoria:
* **Ciclo de Instrucción:** Etapas de *Fetch*, *Decode*, *Execute* y verificación de interrupciones (*Check Interrupt*).
* **MMU (Memory Management Unit):** Traducción de direcciones lógicas a físicas a través de tablas de páginas multinivel.
* **TLB (Translation Lookaside Buffer):** Caché de traducciones de páginas con cantidad de entradas configurable y algoritmos de reemplazo `FIFO` y `LRU`.
* **Caché de Páginas:** Memoria caché local de datos con retardo configurable y algoritmos de reemplazo `CLOCK` y `CLOCK-M` (Clock Modificado).

### 3. 💾 Memoria (`ssoo-memoria`)
Administra el espacio de memoria física de usuario, el almacenamiento secundario (SWAP) y las estructuras de paginación:
* **Espacio de Usuario Contiguo:** Representado mediante un único `[]byte` (*slice*), accedido exclusivamente mediante direcciones físicas.
* **Paginación Jerárquica Multinivel:** Cantidad de niveles, tamaño de página y entradas por tabla configurables por archivo.
* **Gestión de SWAP:** Manejo de un único archivo de paginación en disco para almacenar páginas de procesos suspendidos (`SUSP_BLOCKED` / `SUSP_READY`), respetando los tiempos de retardo configurados.
* **Memory Dump:** Generación de archivos de volcado de memoria para auditoría y verificación de consistencia.

### 4. 🔌 IO (`ssoo-io`)
Módulo que simula dispositivos de Entrada/Salida (como `DISCO`):
* Permite levantar múltiples instancias en simultáneo.
* Se registra dinámicamente ante el Kernel y atiende peticiones bloqueantes de los procesos aplicando los retardos correspondientes.

---

## 📂 Estructura del Repositorio

```text
├── code/                  # Scripts de pseudocódigo para las pruebas de la cátedra
├── configs/               # Perfiles de configuración JSON separados por cada prueba final
│   ├── 1-PlanificacionCortoPlazo/
│   ├── 2-PlanificacionMedioLargoPlazo/
│   ├── 3-MemoriaSWAP/
│   ├── 4-MemoriaCache/
│   ├── 5-MemoriaTLB/
│   └── 6-EstabilidadGeneral/
├── cpu/                   # Módulo CPU (MMU, TLB, Caché, Ciclo de instrucción)
├── io/                    # Módulo de Entrada/Salida
├── kernel/                # Módulo Kernel (Planificadores, Colas, PCB, Syscalls, API)
├── memoria/               # Módulo Memoria (Tablas multinivel, Espacio de usuario, SWAP)
├── utils/                 # Paquetes compartidos (Logger, HTTP, ConfigManager, PCB, Build)
└── go.work                # Definición del workspace de Go
```

---

## 🚀 Requisitos y Compilación

### Prerrequisitos
* **Go:** Versión `1.24` o superior.
* Entorno Linux (Ubuntu Server / UTN SO VM recomendada).

### Compilación Rápida
El proyecto cuenta con una herramienta automatizada en `utils/build_all` para compilar todos los módulos sin necesidad de hacerlo uno por uno, ideal para el deploy en el laboratorio:

```bash
go run ./utils/build_all/build_all.go
```

También podés compilar cada módulo manualmente desde su respectivo directorio:

```bash
cd memoria && go build -o bin/memoria .
cd ../kernel && go build -o bin/kernel .
cd ../cpu && go build -o bin/cpu .
cd ../io && go build -o bin/io .
```
*(Recordá que por criterio de la cátedra no se deben subir archivos binarios ni ejecutables al repositorio).*

---

## ▶️ Ejecución del Sistema

Para levantar el sistema completo, se recomienda respetar el siguiente orden de inicio de módulos:

1. **Levantar Memoria:**
   ```bash
   cd memoria
   go run memoria.go [ruta_config_opcional]
   ```
2. **Levantar CPU(s):**
   ```bash
   cd cpu
   go run cpu.go [id_cpu] [ruta_config_opcional]
   ```
3. **Levantar Instancia(s) de IO:**
   ```bash
   cd io
   go run io.go [nombre_interfaz] [ruta_config_opcional]
   ```
4. **Levantar Kernel:**
   ```bash
   cd kernel
   go run kernel.go <archivo_pseudocodigo> <tamanio_proceso> [ruta_config_opcional]
   ```

---

## 🧪 Guía de Pruebas Finales

En el directorio `configs/` se encuentran preconfigurados los escenarios de las **Pruebas Finales** especificadas por la cátedra, cuyos scripts de pseudocódigo residen en la carpeta `code/`:

| # | Prueba | Script Inicial (`code/`) | Tamaño | Aspectos Evaluados |
| :-: | :--- | :--- | :-: | :--- |
| **1** | **Planificación Corto Plazo** | `PLANI_CORTO_PLAZO` | `0` | Ejecución con 1 y 2 CPUs, paralelismo de 2 instancias de IO `DISCO`, comparación de tiempos de espera entre `FIFO`, `SJF` y `SRT` (con desalojo). |
| **2** | **Planificación Mediano/Largo Plazo** | `PLANI_LYM_PLAZO` | `0` | Grado de multiprogramación, suspensión de procesos `PLANI_LYM_IO` a SWAP e ingreso a `READY` mediante `FIFO` y `PMCP`. |
| **3** | **Memoria - SWAP** | `MEMORIA_IO` | `90` | Paginación de 1 nivel, suspensión a disco y verificación de consistencia en archivos de `DUMP` y `SWAP`. |
| **4** | **Memoria - Caché** | `MEMORIA_BASE` | `256` | Paginación de 3 niveles y reemplazo de páginas en Caché de CPU utilizando los algoritmos `CLOCK` y `CLOCK-M`. |
| **5** | **Memoria - TLB** | `MEMORIA_BASE_TLB` | `256` | Traducción de direcciones en tablas de 3 niveles y reemplazo de entradas en TLB bajo los algoritmos `FIFO` y `LRU`. |
| **6** | **Estabilidad General** | `ESTABILIDAD_GENERAL` | `0` | Prueba de estrés prolongada con 4 CPUs (distintas configuraciones de TLB y Caché) y 4 instancias de IO `DISCO` bajo `SRT` y `PMCP`, verificando ausencia de espera activa y *memory leaks*. |

---

## ⚙️ Configuración para Entorno Distribuido (Deploy)

Antes de ejecutar las pruebas en distintas máquinas del laboratorio:
1. Verificar las direcciones IP de cada máquina (`ip a` o `ifconfig`).
2. Actualizar los campos `ip_memoria`, `ip_kernel` e `ip_cpu` en los archivos `.json` correspondientes dentro de `configs/`.
3. Asegurarse de que el `log_level` de todos los módulos se encuentre seteado en `"INFO"`, tal como lo exige el documento de evaluación.

