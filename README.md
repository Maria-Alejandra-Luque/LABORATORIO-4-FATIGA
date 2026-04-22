# LABORATORIO-4-FATIGA
## Objetivo General: 
Identificar cambios en las características espectrales de una
señal electromiográfica (EMG) cuando se alcanza la fatiga muscular.
## Objetivos Específicos:
1. Aplicar el filtrado de señales continuas para el procesamiento una señal
electromiográfica (EMG).

2. Detectar la aparición de fatiga muscular mediante el análisis espectral de
contracciones musculares individuales.

4. Comparar el comportamiento de una señal emulada y una señal real en
términos de frecuencia media y mediana.

5. Emplear herramientas computacionales para el procesamiento,
segmentación y análisis de señales biomédicas. 


## PARTE A 

## PARTE B 
En la parte B se realizará el procesamiento y análisis de una señal electromiográfica (EMG) adquirida por medio del BITalino y sus respectivos electrodos con el objetivo de
evaluar el comportamiento espectral asociado a la fatiga muscular. Para ello, la señal será preprocesada mediante la eliminación del componente DC y la aplicación de un
filtro pasa banda entre 20 y 450 Hz. Posteriormente, se dividirá en ventanas consecutivas de igual duración, a las cuales se les aplicará la Transformada Rápida de Fourier 
(FFT), considerando únicamente la parte positiva del espectro. La representación de los resultados se realizará en escala semilogarítmica en el eje de frecuencia.
Finalmente, se calcularán la frecuencia media y la frecuencia mediana para cada ventana, con el fin de analizar la evolución de estas magnitudes a lo largo del tiempo.<br>

### ALGORITMO 

### CODIGO 

```
import numpy as np
import matplotlib.pyplot as plt
from scipy.signal import butter, filtfilt
file = r"C:\Users\aleja\Downloads\alejaemg_2026-04-10_14-46-10.txt"
data = []
with open(file, 'r') as f:
    for line in f:
        if not line.startswith('#'):
            values = line.strip().split('\t')
            if len(values) > 5:
                data.append(float(values[5]))  # columna A1
data = np.array(data)
fs = 1000  # Hz
```
Este código carga una señal desde un archivo de texto para poder analizarla después.
Primero, importa las librerías necesarias y abre el archivo, leyendo línea por línea. Extrae los datos de la columna A1, 
que es la señal de interés. Luego, guarda esos valores en un arreglo de NumPy para facilitar su manejo. Finalmente, define
la frecuencia de muestreo en 1000 Hz, que es importante para el análisis de la señal.<br>

```
fs = 1000
low = 20
high = 450
order = 4
b, a = butter(order, [low/(fs/2), high/(fs/2)], btype='band')

def filtro_manual(x, b, a):
    y = np.zeros(len(x))    
    for n in range(len(x)):
        for k in range(len(b)):
            if n-k >= 0:
                y[n] += b[k] * x[n-k]
        for k in range(1, len(a)):
            if n-k >= 0:
                y[n] -= a[k] * y[n-k]
        y[n] = y[n] / a[0]
    return y
x_filt = filtro_manual(x, b, a)     
```
Este fragmento implementa el filtrado de la señal de forma manual. Primero, 
se definen los parámetros del filtro pasa banda (frecuencias de corte, frecuencia de muestreo y orden)
y se obtienen sus coeficientes con butter. Luego, en lugar de usar una función automática, 
se crea una función que aplica el filtro mediante la ecuación en diferencias, calculando cada muestra
de salida a partir de valores actuales y pasados de la entrada y de la salida. Finalmente, esta función 
se usa para obtener la señal filtrada.<br>

```
plt.figure()
plt.plot(x_filt)
plt.title("Señal EMG Filtrada")
plt.xlabel("Muestras")
plt.ylabel("Amplitud")
plt.grid()
plt.show()

```
Este fragmento se encarga de visualizar la señal filtrada. Primero, crea una nueva figura y luego grafica la 
señal x_filt en función de las muestras. Después, se añaden un título y etiquetas a los ejes para facilitar la 
interpretación de la gráfica, y se activa una cuadrícula para mejorar la lectura.Finalmente, se muestra la gráfica,
permitiendo observar el comportamiento de la señal EMG después del filtrado.<br>

```
num_windows = 6 
N = len(x_filt)
L = N // num_windows
f_mean = []
f_med = []

```
Este fragmento prepara el análisis de la señal por segmentos. Primero, se define el número de ventanas
en las que se va a dividir la señal (6). Luego, se calcula la longitud total de la señal filtrada 
y el tamaño de cada ventana, dividiendo el total entre el número de segmentos. Finalmente,
se crean dos listas vacías donde se almacenarán posteriormente la frecuencia media y la 
frecuencia mediana calculadas en cada ventana.<br>
```
for i in range(num_windows):
    seg = x_filt[i*L:(i+1)*L]

    Nseg = len(seg)
    Y = np.fft.fft(seg)
    P = np.abs(Y)**2

    # SOLO PARTE POSITIVA
    f = np.fft.fftfreq(Nseg, d=1/fs)
    mask = f >= 0

    f = f[mask]
    P = P[mask]
```
realizamos el análisis en frecuencia de cada segmento de la señal. Primero, recorriendo
cada una de las ventanas definidas y extrayendo el segmento correspondiente de la señal filtrada. 
Luego, calcula la Transformada Rápida de Fourier (FFT) de ese segmento para obtener su contenido en
frecuencia, y a partir de esto obtiene la potencia del espectro. Después, genera el vector de frecuencias 
asociado y se queda únicamente con la parte positiva del espectro.<br>

```
 plt.figure()
    plt.semilogx(f, P)
    plt.title(f"Espectro ventana {i+1}")
    plt.xlabel("Frecuencia (Hz) - escala log")
    plt.ylabel("Potencia")
    plt.grid()
    plt.show()
```
Este fragmento se encarga de graficar el espectro de potencia de cada ventana de la señal.
Primero, crea una nueva figura y luego utiliza una gráfica semilogarítmica en el eje de la frecuencia
para visualizar mejor el comportamiento del espectro en diferentes rangos. Después, agrega un título
que identifica la ventana analizada, junto con las etiquetas de los ejes y una cuadrícula para facilitar 
la interpretación. Finalmente, muestra la gráfica para observar cómo se distribuye la potencia en función de la frecuencia en cada segmento.<br>
```
    f_mean_i = np.sum(f * P) / np.sum(P)
    f_mean.append(f_mean_i)

    cumsumP = np.cumsum(P)
    idx = np.where(cumsumP >= cumsumP[-1]/2)[0][0]
    f_med_i = f[idx]
    f_med.append(f_med_i)

f_mean = np.array(f_mean)
f_med = np.array(f_med)

print("Frecuencia media:", f_mean)
print("Frecuencia mediana:", f_med)
```
Este fragmento calcula características importantes del espectro en cada ventana de la señal. Primero, 
obtiene la frecuencia media como un promedio ponderado usando la potencia, lo que indica en qué rango de frecuencias
se concentra la energía. Luego, calcula la frecuencia mediana, que corresponde al punto donde se acumula el 50% de la potencia
total del espectro.<br> 
Ambos valores se almacenan en listas para cada segmento. Al final, estas listas se convierten en arreglos de NumPy 
y se imprimen, mostrando cómo varían estas frecuencias a lo largo de la señal.<br>

```
plt.figure()
plt.plot(f_mean, '-o', label='Frecuencia media')
plt.plot(f_med, '-x', label='Frecuencia mediana')
plt.title("Evolución de la fatiga muscular")
plt.xlabel("Ventanas / Contracciones")
plt.ylabel("Frecuencia (Hz)")
plt.legend()
plt.grid()
plt.show()
```
Este fragmento grafica la evolución de la fatiga muscular a lo largo del tiempo. Para ello, 
muestra cómo cambian la frecuencia media y la frecuencia mediana en cada ventana de la señal, 
utilizando marcadores distintos para diferenciarlas. Además, se añaden título, etiquetas en los ejes,
leyenda y cuadrícula para facilitar la interpretación.<br>
Esta gráfica permite observar posibles disminuciones en la frecuencia, que suelen estar asociadas a la aparición de fatiga muscular.<br>
### GRAFICAS

## PARTE C 
