# LABORATORIO 1 : Monitoreo del patrón y frecuencia respiratoria

# INTRODUCCIÓN



# REQUERIMIENTOS

 -Matlab

-ESP32

-Arduino IDE

# FUNDAMENTOS TEÓRICOS

Desde el punto de vista fisiológico, la ventilación pulmonar se basa en el movimiento del aire hacia el interior y el exterior de los pulmones como consecuencia de gradientes de presión entre la atmósfera, los alvéolos y el espacio intrapleural. Durante la inspiración, la expansión de la caja torácica genera una presión intrapleural negativa que favorece la entrada de aire, mientras que en la espiración el retroceso elástico pulmonar incrementa la presión alveolar, permitiendo la salida del aire hacia el exterior.

El proceso respiratorio puede describirse mediante diversas variables físicas medibles, entre las que se destacan la presión, el volumen y el flujo respiratorio. Estas variables permiten caracterizar la ventilación pulmonar y el comportamiento del aire durante las fases de inspiración y espiración. Adicionalmente, variables temporales como la frecuencia respiratoria y la duración de cada fase del ciclo respiratorio permiten describir la dinámica del proceso en el tiempo y evaluar la respuesta del sistema respiratorio ante diferentes demandas fisiológicas.

Si bien existen múltiples variables asociadas al proceso respiratorio, no todas son fácilmente accesibles para su medición en un entorno de laboratorio sin instrumentación clínica especializada. En esta práctica se seleccionó la expansión mecánica de la caja torácica como variable indirecta de interés, debido a que refleja de manera periódica las fases de inspiración y espiración y puede ser capturada de forma no invasiva. Esta variable permite obtener el patrón respiratorio y estimar la frecuencia respiratoria, cumpliendo con los objetivos propuestos en la guía de laboratorio.

## Sensor FSR-40

El proceso respiratorio se manifiesta externamente a través de variaciones mecánicas asociadas a la expansión y contracción de la caja torácica durante las fases de inspiración y espiración. Estas variaciones presentan una frecuencia baja y un comportamiento periódico, lo que permite su detección mediante sensores externos no invasivos que respondan a cambios mecánicos inducidos por la respiración.

Para el desarrollo de esta práctica se seleccionó un sensor de fuerza resistivo (FSR-40), el cual modifica su resistencia eléctrica en función de la presión aplicada sobre su superficie. Este tipo de sensor resulta adecuado para la detección indirecta del patrón respiratorio, ya que permite convertir las variaciones mecánicas producidas por la expansión torácica en una señal eléctrica proporcional, susceptible de ser acondicionada y digitalizada.

El FSR-40 presenta ventajas relevantes para su aplicación en un entorno de laboratorio, entre las que se destacan su bajo costo, flexibilidad, facilidad de integración y compatibilidad con sistemas de adquisición basados en microcontroladores. Al tratarse de un sensor pasivo, puede ser alimentado con tensiones entre 3.3 y 5 V, cumpliendo con las especificaciones del sistema utilizado en esta práctica.

No obstante, es importante señalar que el FSR-40 responde a presión y no directamente a desplazamiento, lo que implica que las variaciones asociadas a la respiración pueden presentar amplitudes reducidas. Por esta razón, fue necesario aplicar una precarga mecánica mediante una banda elástica y un material flexible, con el fin de aumentar la sensibilidad del sensor a los cambios producidos durante el ciclo respiratorio.

En conjunto, el sensor FSR-40 permitió la adquisición del patrón respiratorio de manera no invasiva, constituyéndose en una alternativa adecuada para cumplir los objetivos planteados en la guía de laboratorio

# Montaje del sensor 

El sensor FSR-40, se conectó a la entrada analógica de una ESP32 mediante una configuración de divisor de voltaje alimentada a 3.3 V. En este montaje, el sensor se dispone en serie con una resistencia fija conectada a tierra, generando en el nodo intermedio una señal de voltaje proporcional a las variaciones de resistencia inducidas por el estímulo físico aplicado (fuerza o presión). Dicho voltaje es adquirido directamente por el ADC de la ESP32, permitiendo la conversión de la magnitud física en una señal digital sin necesidad de etapas adicionales de acondicionamiento.


Una vez establecido el diseño del sistema para la adquisición y digitalización de la señal respiratoria, se llevó a cabo su implementación física con el propósito de realizar la recolección de información. A continuación, se presenta el montaje desarrollado del sistema:





# Digitalización, Adquisición y visualización de la señal

la señal se digitalizo mediante el ADC integrado que posee la esp32, este se definio bajo los parametros expuestos a continuación 

```bash
const int fsrPin = 34; 

void setup() {
  Serial.begin(115200); 
  delay(1000);

  analogReadResolution(12);        
  analogSetAttenuation(ADC_11db);  
}

void loop() {
  long suma = 0;

  for (int i = 0; i < 10; i++) {
    suma += analogRead(fsrPin);
    delayMicroseconds(200);
  }

  int fsrValue = suma / 10;
  Serial.println(fsrValue);

  delay(10);  
}

```
El código lee de forma continua la señal del sensor, promedia varias muestras consecutivas (10 muestras) para reducir el ruido y envía el valor resultante por comunicación serial. La configuración del ADC permite medir correctamente el rango de voltaje del sensor, y el proceso se repite cada 10 ms, estableciendo una frecuencia de muestreo aproximada de 100 Hz, adecuada para el análisis temporal de la señal respiratoria.

## visualización y guardado

Luego de enviar los datos por comunicación serial, estos son recibidos en una interfaz de matlab para ser procesada

```bash
clc
clear
close all

puerto   = "COM3";
baudrate = 115200;

T = input('Ingrese el tiempo que desea visualizar (s): ');

s = serialport(puerto, baudrate);
configureTerminator(s,"LF");
flush(s);

disp("Recepción de datos en tiempo real...")

data = [];
time = [];
fig = figure;
h = plot(nan, nan, 'b', 'LineWidth', 1.5);
xlabel('Tiempo [s]')
ylabel('Señal respiratoria (ADC normalizado)')
title('Señal respiratoria en tiempo real')
grid on

xlim([0 T])
ylim([0 1])

refreshRate = 0.05;
lastUpdate = tic;
tStart = tic;

while isvalid(fig) && toc(tStart) < T

    if s.NumBytesAvailable > 0
        valor = str2double(readline(s));

        if ~isnan(valor)
            data(end+1) = valor / 4095;   
            time(end+1) = toc(tStart);
        end
    end

    if toc(lastUpdate) > refreshRate
        h.XData = time;
        h.YData = data;
        drawnow limitrate
        lastUpdate = tic;
    end
end

fechaHora = datestr(now,'yyyy_mm_dd_HH_MM_SS');
nombreArchivo = ['senal_respiratoria_' fechaHora '.mat'];

senal = data;     
tiempo = time;       
duracion = T;       
fs_aprox = 1/mean(diff(time)); 

save(nombreArchivo, 'senal', 'tiempo', 'duracion', 'fs_aprox');

disp(['Datos guardados en el archivo: ' nombreArchivo])

if isvalid(fig)
    close(fig)
end

clear s
disp("Visualización finalizada y puerto cerrado correctamente")
```
Este script implementa un sistema de adquisición y visualización en tiempo real de la señal recibida por puerto serial. El programa establece la comunicación con el dispositivo, captura de manera continua las muestras provenientes del ADC, las normaliza y las representa gráficamente en función del tiempo durante un periodo definido por el usuario. Al finalizar el proceso, los datos obtenidos, junto con el eje temporal, la duración del registro y una estimación de la frecuencia de muestreo, se almacenan automáticamente en un archivo .mat, lo que facilita su análisis.






