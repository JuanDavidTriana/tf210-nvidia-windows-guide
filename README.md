# Guía de Instalación de TensorFlow GPU en Windows

## Objetivo
Configurar un entorno de desarrollo con TensorFlow utilizando aceleración por GPU NVIDIA en Windows.

## Requisitos de Hardware

- Tarjeta gráfica NVIDIA compatible con CUDA.
- Drivers NVIDIA actualizados.
- Memoria RAM recomendada: 8 GB o superior.
- Espacio libre en disco: mínimo 10 GB.

## Versiones Utilizadas

| Componente | Versión |
| --- | --- |
| Python | 3.9.x |
| TensorFlow | 2.10.0 |
| NumPy | 1.23.5 |
| CUDA Toolkit | 11.2 |
| cuDNN | 8.1 |
| Sistema Operativo | Windows 10/11 64 bits |

## Verificación de la GPU

Ejecutar:

```bash
nvidia-smi
```

Ejemplo de salida:

```text
Driver Version: 596.08
CUDA Version: 13.2
GPU: NVIDIA GeForce RTX 4070 Laptop GPU
```

La GPU debe aparecer correctamente reconocida por el sistema.

## Creación del Entorno Virtual

```bash
conda create -n tf_windows python=3.9
conda activate tf_windows
```

## Instalación de TensorFlow

Actualizar herramientas de instalación:

```bash
pip install --upgrade pip setuptools wheel
```

Instalar TensorFlow:

```bash
pip install tensorflow==2.10.0
```

## Compatibilidad con NumPy

TensorFlow 2.10 no es compatible con NumPy 2.x.

Instalar una versión compatible:

```bash
pip install numpy==1.23.5
```

Verificar:

```bash
python -c "import numpy; print(numpy.__version__)"
```

Resultado esperado:

```text
1.23.5
```

## Descargar e Instalar CUDA Toolkit

TensorFlow 2.10 requiere **CUDA Toolkit 11.2**.

### Pasos de instalación:

1. **Descargar CUDA Toolkit 11.2** desde NVIDIA:
   - Ir a: https://developer.nvidia.com/cuda-11-2-0-download-archive
   - Seleccionar:
     - Sistema Operativo: Windows
     - Arquitectura: x86_64
     - Versión de Windows: Windows 10 o Windows 11
     - Tipo de instalador: exe (local)
   - Hacer clic en "Download"

2. **Ejecutar el instalador** y seguir el asistente de instalación.

3. **Verificar la instalación**:
   ```bash
   nvidia-smi
   ```
   Debe mostrar `CUDA Version: 11.2` o similar.

## Descargar e Instalar cuDNN

TensorFlow 2.10 requiere **cuDNN 8.1**.

### Pasos de instalación:

1. **Descargar cuDNN 8.1** desde NVIDIA:
   - Ir a: https://developer.nvidia.com/cudnn
   - Crear cuenta NVIDIA o iniciar sesión
   - Descargar: **cuDNN 8.1.x for CUDA 11.x**
   - Seleccionar la versión para Windows

2. **Extraer y configurar cuDNN**:
   - Extraer el archivo descargado
   - Copiar los archivos de `bin/`, `lib/`, e `include/` a la carpeta de instalación de CUDA:
     ```
     C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.2\
     ```

## Configuración de Variables de Entorno

CUDA y cuDNN deben estar configurados en las variables de entorno del sistema.

### Verificar las variables de entorno:

1. Abre Símbolo del sistema y ejecuta:
   ```bash
   echo %CUDA_PATH%
   ```
   Debe mostrar: `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.2`

2. Verifica que el PATH incluya:
   ```bash
   echo %PATH%
   ```
   Debe incluir la ruta de CUDA y cuDNN.

## Verificación de TensorFlow

```bash
python -c "import tensorflow as tf; print(tf.__version__)"
```

Resultado esperado:

```text
2.10.0
```

## Verificación de la GPU desde TensorFlow

```python
import tensorflow as tf
import time

print("=" * 50)
print("TensorFlow:", tf.__version__)
print("=" * 50)

gpus = tf.config.list_physical_devices('GPU')
print("GPUs detectadas:", gpus)

if gpus:
    print("✅ TensorFlow detectó la GPU")

    for gpu in gpus:
        print("GPU:", gpu)

    try:
        tf.config.experimental.set_memory_growth(gpus[0], True)
    except:
        pass
else:
    print("❌ TensorFlow NO detectó la GPU")

size = 5000

inicio = time.time()

with tf.device('/GPU:0' if gpus else '/CPU:0'):
    a = tf.random.normal([size, size])
    b = tf.random.normal([size, size])
    c = tf.matmul(a, b)

_ = c.numpy()

fin = time.time()

print("=" * 50)
print(f"Tiempo de ejecución: {fin - inicio:.2f} segundos")
print("Resultado:", c.shape)
print("=" * 50)
```

## Prueba de Rendimiento

```python
import tensorflow as tf
import time

gpus = tf.config.list_physical_devices('GPU')

def run_benchmark():
    with tf.device('/GPU:0' if gpus else '/CPU:0'):
        inicio = time.time()
        a = tf.random.normal([5000, 5000])
        b = tf.random.normal([5000, 5000])
        c = tf.matmul(a, b)
        _ = c.numpy()
        fin = time.time()
    print('Tiempo:', fin - inicio)

run_benchmark()
```

## Problemas Comunes

### Error: NumPy 2.x incompatible

Mensaje:

```text
A module that was compiled using NumPy 1.x cannot be run in NumPy 2.x
```

Solución:

```bash
pip uninstall numpy -y
pip install numpy==1.23.5
```

### Error: _ARRAY_API not found

Causa: TensorFlow compilado para NumPy 1.x y entorno ejecutando NumPy 2.x.

Solución:

```bash
pip install numpy==1.23.5
```

### Error: numpy.core._multiarray_umath failed to import

Causa: Incompatibilidad entre TensorFlow y NumPy.

Solución:

```bash
pip uninstall tensorflow numpy -y
pip install numpy==1.23.5
pip install tensorflow==2.10.0
```

### Error SSL con pypi.ngc.nvidia.com

Causa: Configuración residual de NVIDIA PyIndex en archivos pip.ini.

Solución: eliminar las entradas:

```ini
extra-index-url = https://pypi.ngc.nvidia.com
trusted-host = pypi.ngc.nvidia.com
```

de los archivos de configuración de pip.

## Configuración Final Recomendada

```text
Python        3.9.x
TensorFlow    2.10.0
NumPy         1.23.5
CUDA          11.2
cuDNN         8.1
GPU NVIDIA    Compatible con CUDA
Windows       10/11 64 bits
```

Con esta combinación se obtiene la máxima compatibilidad para TensorFlow GPU en Windows.

## Referencias Oficiales

- [Documentación oficial de instalación de TensorFlow (pip)](https://www.tensorflow.org/install/pip?hl=es-419&utm_source=chatgpt.com)

### Nota Importante

TensorFlow 2.10 es la última versión que ofrece soporte nativo para GPU en Windows. Las versiones posteriores de TensorFlow ya no incluyen soporte GPU nativo en Windows y recomiendan el uso de Linux o WSL2 (Windows Subsystem for Linux).

Antes de instalar TensorFlow, siempre se recomienda verificar la matriz de compatibilidad oficial entre TensorFlow, Python, CUDA Toolkit, cuDNN y el sistema operativo, ya que estos requisitos pueden cambiar con nuevas versiones.