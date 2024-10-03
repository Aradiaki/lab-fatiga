# Informe Fatiga Muscular
## Resumen
Con el objetivo de analizar la fatiga muscular se hace una captura electromiografía, se le realiza el respectivo tratamiento a la señal aplicando un filtro Butterworth pasa banda, luego se procede a analizar mediante la transformada de Fourier el espectro de frecuencias para determinar como varia el espectro en la fatiga muscular y así determinar cuando ocurre.
## Introducción
El objetivo de esta practica es analizar el comportamiento eléctrico de los músculos frente a esfuerzos que le generen un estado de fatiga, para ello se pretende hacer una captura electromiografía y determinar sus características para analizar el cambio durante todo el tiempo de la captura para determinar en qué momento ocurre la fatiga.
## Procedimiento 
## Captura de la señal 
Para la captura se utilizó un Arduino mega para capturar la señal EMG, mediante el uso del ADC de la placa, además se utilizó el entorno Matlab para realizar la comunicación serial y observar los datos en tiempo real, luego fueron guardados y exportados a Python para su posterior tratamiento.
## Código Matlab
          clc;
          clear;
          close all;
          
          % Establecer conexión con el puerto serial
          s = serialport('COM4', 9600);
          pause(2); % Esperar a que la conexión se establezca
          
          % Inicializar la gráfica
          figure; % Crear una nueva figura
          hplot = plot(NaN, NaN); % Inicializar el objeto de la gráfica
          axis([0 1000 -1 100]); % Ajustar los límites de los ejes (ajusta según tus datos)
          xlabel('Muestras');
          ylabel('Valor');
          grid on; % Activar la cuadrícula
          
          % Inicializar variables
          x = 0; % Contador para el eje X
          yData = []; % Inicializar un vector vacío para los datos del eje Y
          allData = []; % Inicializar un vector vacío para almacenar todos los datos leídos
          
          % Filtro
          fs = 9600; % Frecuencia de muestreo
          Wn = [20 200] / (fs / 2); % Normalizar la frecuencia pasabanda
          
          % Rango de frecuencias para el filtro notch (rechazo)
          Wn2 = [55 65] / (fs / 2); % Normalizar la frecuencia de rechazo (ejemplo 55-65 Hz)
          
          [num, den] = butter(1, Wn, 'bandpass'); % Filtro pasabanda
          [nume, dene] = butter(1, Wn2, 'stop'); % Filtro de rechazo de banda
          
          % Bucle para la actualización en tiempo real
          
              while true
                  % Leer una línea del puerto serial
                  str = readline(s);
                  valor = str2double(str); % Convertir la cadena a número
          
                  % Comprobar que el valor leído es válido
                  if ~isnan(valor)
                      % Aplicar los filtros
                      valorf = filter(num, den, valor); % Aplicar el filtro pasabanda
                      valorf1 = filter(nume, dene, valorf); % Aplicar el filtro de rechazo
          
                      x = x + 1; % Incrementar el contador
                      yData(end + 1) = valorf1; % Agregar el nuevo valor al vector yData
                      allData(end + 1) = valorf1; % Almacenar todos los valores leídos
          
                      % Actualizar los datos de la gráfica
                      hplot.XData = 1:length(yData); % Eje X con todas las muestras
                      hplot.YData = yData; % Eje Y con todos los valores
          
                      % Ajustar los límites del eje X para mostrar solo los últimos 100 puntos
                      if length(yData) > 1000
                          axis([length(yData) - 1000 length(yData) - 1 -1 100]); % Limitar a los últimos 100 puntos
                      end
                  end
          
                  % Actualizar la gráfica
                  drawnow; % Forzar a MATLAB a actualizar la figura
              end

## Guardado de la señal 
      % Guardar el vector 'data' en un archivo .mat
      save('datosmusculo.mat', 'allData');
      
      Y=load('datosmusculo.mat');
      dat=Y.allData;
      figure()
      plot(dat(1:1119))

## Exportación 
## Código en Python
## Librerias utilizadas
    import scipy.io
    import numpy as np
    import matplotlib.pyplot as plt
    from scipy.signal import butter, filtfilt
    exportación archivo .mat 
##  Cargar el archivo .mat
          Y = scipy.io.loadmat('datosmusculo.mat')
          
          # Extraer la variable 'allData' del archivo .mat y aplanarla
          data1 = Y['allData'].flatten()
          Tratamiento de la señal
          
          # Definir parámetros del filtro
          fs = 9600  # Frecuencia de muestreo en Hz
          lowcut = 20.0  # Frecuencia de corte inferior
          highcut = 200.0  # Frecuencia de corte superior
          
          # Diseñar el filtro Butterworth
          def butter_bandpass(lowcut, highcut, fs, order=1):
              nyquist = 0.5 * fs
              low = lowcut / nyquist
              high = highcut / nyquist
              b, a = butter(order, [low, high], btype='band')
              return b, a
          
          # Aplicar el filtro a los datos
          def bandpass_filter(data, lowcut, highcut, fs, order=1):
              b, a = butter_bandpass(lowcut, highcut, fs, order=order)
              y = filtfilt(b, a, data)
              return y
          
          # Filtrar los datos
          data_filtered = bandpass_filter(data1, lowcut, highcut, fs)
          
          # Calcular la media de la señal filtrada
          mean_filtered = np.mean(data_filtered)
          
          # Detectar picos en la señal filtrada que superen la media (contracciones musculares)
          # Añadimos 'distance' para agrupar picos cercanos
          min_distance = int(0.5 * fs)  # Distancia mínima entre picos en muestras (0.5 segundos)
          peaks, _ = find_peaks(data_filtered, height=mean_filtered, distance=min_distance)
          
          # Imprimir el número de contracciones musculares
          num_contractions = len(peaks)
          print(f'Número de contracciones musculares detectadas: {num_contractions}')
          
          # Graficar la señal filtrada con los picos marcados
          plt.figure(figsize=(12, 6))
          plt.plot(data_filtered, label='Datos Filtrados', color='blue')
          plt.plot(peaks, data_filtered[peaks], 'ro', label='Contracciones (Picos)')
          plt.axhline(mean_filtered, color='green', linestyle='--', label=f'Media: {mean_filtered:.2f}')
          plt.title('Señal Filtrada con Picos Detectados (Contracciones Musculares)')
          plt.xlabel('Muestras')
          plt.ylabel('Amplitud')
          plt.legend()
          plt.grid()
          plt.show()
          
          # Longitud total de la señal
          signal_length = len(data_filtered)
          
          # Ajustar el tamaño de la ventana para el número de picos detectados (contracciones)
          num_segments = num_contractions
          window_size = signal_length // num_segments  # Tamaño de la ventana
          overlap = window_size // 2  # Solapamiento del 50%
          
          # Crear una ventana Hamming manualmente
          def hamming_window(size):
              return 0.54 - 0.46 * np.cos(2 * np.pi * np.arange(size) / (size - 1))
          
          # Aplicar la ventana Hamming a la señal filtrada en intervalos y calcular la Transformada de Fourier
          for start in range(0, signal_length - window_size + 1, window_size - overlap):
              window = hamming_window(window_size)
              segment = data_filtered[start:start + window_size]
              windowed_segment = segment * window
          
              # Calcular la Transformada de Fourier
              spectrum = np.fft.fft(windowed_segment)
              freqs = np.fft.fftfreq(window_size, d=1/fs)  # Frecuencias asociadas
          
              # Obtener la magnitud del espectro (solo la parte positiva)
              magnitude = np.abs(spectrum[:window_size // 2])
              positive_freqs = freqs[:window_size // 2]
          
              # Calcular la frecuencia principal (pico más alto)
              peak_index = np.argmax(magnitude)  # Índice del pico más alto
              frequency_main = positive_freqs[peak_index]  # Frecuencia principal
          
              # Calcular la frecuencia media
              frequency_mean = np.sum(positive_freqs * magnitude) / np.sum(magnitude)
          
              # Calcular la desviación estándar
              std_dev = np.std(magnitude)
          
              # Imprimir los resultados
              print(f'Segmento {start // (window_size - overlap) + 1}:')
              print(f'  Frecuencia Principal: {frequency_main:.2f} Hz')
              print(f'  Frecuencia Media: {frequency_mean:.2f} Hz')
              print(f'  Desviación Estándar: {std_dev:.2f}\n')
          
              # Graficar cada segmento con la ventana Hamming aplicada
              plt.figure(figsize=(12, 6))
              plt.subplot(2, 1, 1)  # Gráfica del segmento
              plt.plot(segment, label='Segmento Filtrado', color='blue', alpha=0.5)
              plt.plot(windowed_segment, label='Segmento con Ventana Hamming', color='orange')
              plt.title(f'Segmento {start // (window_size - overlap) + 1}')
              plt.xlabel('Muestras')
              plt.ylabel('Amplitud')
              plt.legend()
              plt.grid()
          
              # Gráfica del espectro de frecuencias
              plt.subplot(2, 1, 2)  # Gráfica del espectro
              plt.plot(positive_freqs, magnitude, color='green')
              plt.scatter(frequency_main, magnitude[peak_index], color='red', zorder=5, label='Frecuencia Principal')  # Pico máximo
              plt.title('Espectro de Frecuencias')
              plt.xlabel('Frecuencia (Hz)')
              plt.ylabel('Magnitud')
              plt.xlim(0, fs / 2)  # Limitar a la mitad de la frecuencia de muestreo
              plt.legend()
              plt.grid()
          
              plt.tight_layout()  # Ajustar el layout
              plt.show()
## explicacion del codigo

con este código se realiza primero el calculo del numero de contracciones que que hay en la señal mediante la función findpeaks esto respecto a la media de la señal tomada como umbral para determinar un pico, a su vez con ese dato de contracciones se toma este valor como numero de segmentos, es decir numero de ventanas que quiero tomar en la señal, se realiza el calculo del intervalo que debe tener la ventana y su tamaño, se utilizó una ventana haming, a cada segmento se aplicó transformada de Fourier y se calcularon sus respectivas frecuencias principal, media, y desviación estándar.
## resultados
resultados obtenidos de los espectros de fourier para cada segmento realizado


Segmento 1:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 224.33 Hz
  Desviación Estándar: 18.66

Segmento 2:
  Frecuencia Principal: 0.00 Hz
  Frecuencia Media: 160.67 Hz
  Desviación Estándar: 25.01

Segmento 3:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 395.20 Hz
  Desviación Estándar: 7.30

Segmento 4:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 314.51 Hz
  Desviación Estándar: 8.73

Segmento 5:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 243.34 Hz
  Desviación Estándar: 15.97

Segmento 6:
  Frecuencia Principal: 0.00 Hz
  Frecuencia Media: 334.64 Hz
  Desviación Estándar: 9.20

Segmento 7:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 345.68 Hz
  Desviación Estándar: 7.23

Segmento 8:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 225.29 Hz
  Desviación Estándar: 14.93

Segmento 9:
  Frecuencia Principal: 0.00 Hz
  Frecuencia Media: 178.50 Hz
  Desviación Estándar: 11.58

Segmento 10:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 305.37 Hz
  Desviación Estándar: 11.38

Segmento 11:
  Frecuencia Principal: 0.00 Hz
  Frecuencia Media: 216.71 Hz
  Desviación Estándar: 12.74

Segmento 12:
  Frecuencia Principal: 0.00 Hz
  Frecuencia Media: 253.69 Hz
  Desviación Estándar: 8.48

Segmento 13:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 397.63 Hz
  Desviación Estándar: 3.49

Segmento 14:
  Frecuencia Principal: 0.00 Hz
  Frecuencia Media: 351.51 Hz
  Desviación Estándar: 1.04

Segmento 15:
  Frecuencia Principal: 0.00 Hz
  Frecuencia Media: 518.96 Hz
  Desviación Estándar: 1.86

Segmento 16:
  Frecuencia Principal: 0.00 Hz
  Frecuencia Media: 148.32 Hz
  Desviación Estándar: 12.28

Segmento 17:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 296.59 Hz
  Desviación Estándar: 4.09

Segmento 18:
  Frecuencia Principal: 0.00 Hz
  Frecuencia Media: 141.63 Hz
  Desviación Estándar: 6.56

Segmento 19:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 389.71 Hz
  Desviación Estándar: 2.41

Segmento 20:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 278.95 Hz
  Desviación Estándar: 2.40

Segmento 21:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 220.87 Hz
  Desviación Estándar: 1.56

Segmento 22:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 350.78 Hz
  Desviación Estándar: 0.89

Segmento 23:
  Frecuencia Principal: 0.00 Hz
  Frecuencia Media: 249.41 Hz
  Desviación Estándar: 1.44

Segmento 24:
  Frecuencia Principal: 0.00 Hz
  Frecuencia Media: 337.87 Hz
  Desviación Estándar: 1.23

Segmento 25:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 352.16 Hz
  Desviación Estándar: 1.93

Segmento 26:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 255.56 Hz
  Desviación Estándar: 2.98

Segmento 27:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 286.72 Hz
  Desviación Estándar: 2.06

Segmento 28:
  Frecuencia Principal: 0.00 Hz
  Frecuencia Media: 205.09 Hz
  Desviación Estándar: 3.68

Segmento 29:
  Frecuencia Principal: 0.00 Hz
  Frecuencia Media: 187.41 Hz
  Desviación Estándar: 14.67

Segmento 30:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 192.44 Hz
  Desviación Estándar: 9.66

Segmento 31:
  Frecuencia Principal: 139.13 Hz
  Frecuencia Media: 314.45 Hz
  Desviación Estándar: 4.15
## archivo adjunto imagenes graficos
https://unimilitareduco-my.sharepoint.com/:w:/g/personal/est_juliane_arias_unimilitar_edu_co/Eajw249y_uBHtY3gzNqM3WkBJftBzkq4-0Q2P8h7nLjG9A?e=HsFHiM
