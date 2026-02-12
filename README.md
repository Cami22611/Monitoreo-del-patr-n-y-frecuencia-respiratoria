# LABORATORIO 1 : Monitoreo del patrón y frecuencia respiratoria

# INTRODUCCIÓN
En esta práctica se realizó el análisis de la señal respiratoria mediante la adquisición de datos con un sensor y su posterior procesamiento digital. El procedimiento incluyó la visualización en tiempo real, el filtrado de la señal y el análisis en el dominio de la frecuencia, permitiendo estimar la frecuencia respiratoria y evaluar su comportamiento de forma cuantitativa.

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

<img width="925" height="720" alt="image" src="https://github.com/user-attachments/assets/6cd49768-8f29-4b34-8762-d785aa05386b" />



Una vez establecido el diseño del sistema para la adquisición y digitalización de la señal respiratoria, se llevó a cabo su implementación física con el propósito de realizar la recolección de información. A continuación, se presenta el montaje desarrollado del sistema:

<img width="423" height="358" alt="image" src="https://github.com/user-attachments/assets/356df3e4-407a-4b83-9ad9-204a1cfff8b3" />

# Digitalización de la señal

La señal se digitalizó mediante el ADC integrado que posee la esp32, este se definio bajo los parametros expuestos a continuación 

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

# Primera parte: conteo manual de la frecuencia respiratoria 

Como parte inicial, la guía nos indica que se debe realizar la visualización de la señal dentro de un prompt, para este caso el de arduino, y seguido a esto montar de forma manual el número de respiraciones del sujeto en reposo y durante el proceso de habla.

A continuación se muestra una imagen en donde se observa la interfaz del prompt y como se ve la señal dentro de este medio:


<img width="980" height="613" alt="image" src="https://github.com/user-attachments/assets/747aeab9-58f1-46e4-9a20-2fec7cc047c3" />

Esta primera imagen corresponde al sujeto en estado de reposo, para este caso se presento una frecuencia respiratoria de 13 rpm.

<img width="979" height="621" alt="image" src="https://github.com/user-attachments/assets/a120b687-09e9-40f7-a2f4-4e9c7c87e299" />


Para el segundo caso expuesto en la guía, se indicó al sujeto que leyera en voz alta un texto durante 1 minuto y se realizo el mismo proceso de conteo manual, para este caso fueron 19 rpm.

# Segunda Parte: frecuencia respiratoria mediante el espectro de frecuencias

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

## Adecución de la señal

Una vez almacenada la señal en el archivo .mat se empleará en un script posterior, donde se aplicarán técnicas de filtrado con el fin de mejorar la claridad de la señal. Posteriormente, se obtendrá su espectro en frecuencia, a partir del cual se estimará la frecuencia respiratoria identificando el componente dominante del espectro. De acuerdo con lo anterior el código utilizado fue:

```bash
clc
clear
close all

r = load('senal_respiracion_5 N.mat');
x1 = r.datos;
fs1 = r.fs;
t1 = r.t;

h = load('senal_respiracion_1 leyendo.mat');
x2 = h.datos;
fs2 = h.fs;
t2 = h.t;

x1 = x1 - mean(x1);
x2 = x2 - mean(x2);

fc = 1;

N1 = length(x1);
F1 = fft(x1);
f1 = (0:N1-1)*(fs1/N1);
F1(f1 > fc & f1 < (fs1 - fc)) = 0;
x1f = real(ifft(F1));
x1f = x1f - mean(x1f);

N2 = length(x2);
F2 = fft(x2);
f2 = (0:N2-1)*(fs2/N2);
F2(f2 > fc & f2 < (fs2 - fc)) = 0;
x2f = real(ifft(F2));
x2f = x2f - mean(x2f);

figure
subplot(2,1,1)
plot(t1,x1,'LineWidth',1)
xlabel('Tiempo (s)')
ylabel('Amplitud')
title('Señal Respiratoria en Reposo - Dominio del Tiempo')
grid on

subplot(2,1,2)
plot(t1,x1f,'LineWidth',1)
xlabel('Tiempo (s)')
ylabel('Amplitud')
title('Señal Respiratoria Filtrada en Reposo')
grid on

figure
subplot(2,1,1)
plot(t2,x2,'LineWidth',1)
xlabel('Tiempo (s)')
ylabel('Amplitud')
title('Señal Respiratoria Hablando - Dominio del Tiempo')
grid on

subplot(2,1,2)
plot(t2,x2f,'LineWidth',1)
xlabel('Tiempo (s)')
ylabel('Amplitud')
title('Señal Respiratoria Filtrada Hablando')
grid on

X1 = abs(fft(x1f))/N1;
f1p = f1(1:floor(N1/2));
X1 = X1(1:floor(N1/2));

figure
plot(f1p,X1,'LineWidth',1.5)
xlabel('Frecuencia (Hz)')
ylabel('Magnitud de la Transformada de Fourier')
title('Espectro de Magnitud - Señal Respiratoria en Reposo')
xlim([0 2])
grid on

b1 = f1p >= 0.1 & f1p <= 0.6;
[~,i1] = max(X1(b1));
fb1 = f1p(b1);
fdom1 = fb1(i1)

rpm1 = fdom1*60

X2 = abs(fft(x2f))/N2;
f2p = f2(1:floor(N2/2));
X2 = X2(1:floor(N2/2));

figure
plot(f2p,X2,'LineWidth',1.5)
xlabel('Frecuencia (Hz)')
ylabel('Magnitud de la Transformada de Fourier')
title('Espectro de Magnitud - Señal Respiratoria Hablando')
xlim([0 2])
grid on

b2 = f2p >= 0.1 & f2p <= 0.6;
[~,i2] = max(X2(b2));
fb2 = f2p(b2);
fdom2 = fb2(i2)

rpm2 = fdom2*60
```
## Caso 1: sujeto en reposo
### Señal original vs señal filtrada
En la gráfica de la señal respiratoria en reposo en el dominio del tiempo, se observa un comportamiento claramente periódico, correspondiente a los ciclos normales de inspiración y espiración. La señal original presenta pequeñas fluctuaciones rápidas superpuestas a la forma principal, las cuales corresponden a ruido o interferencias del sistema de adquisición. Sin embargo, a pesar de estas variaciones, se distingue un patrón repetitivo bastante uniforme, lo que indica que la respiración del sujeto es regular y estable. No se evidencian cambios bruscos en amplitud ni en el periodo entre ciclos, lo que sugiere un estado fisiológico tranquilo.

Después de aplicar el filtro pasa-bajas, la señal se vuelve mucho más suave y limpia. Se eliminan las componentes de alta frecuencia y se resalta únicamente la dinámica respiratoria principal. En la señal filtrada se distinguen claramente los ciclos completos de respiración, con una forma de onda más definida y continua. La periodicidad se mantiene constante y la amplitud presenta variaciones leves entre ciclos, lo que es característico de una respiración en estado de reposo. Este resultado confirma que el filtrado fue adecuado para mejorar la calidad de la señal sin alterar su comportamiento fisiológico principal.

<img width="690" height="523" alt="image" src="https://github.com/user-attachments/assets/a8d71e5d-0ea6-4d04-a25e-2d8e8d3f513d" />

### Espectro de magnitud 

En la representación en frecuencia, el espectro muestra un pico dominante bien definido alrededor de 0.2 Hz. Esto indica que la mayor parte de la energía de la señal está concentrada en una única frecuencia fundamental, correspondiente al ritmo respiratorio. La ausencia de múltiples picos significativos en otras frecuencias confirma que la respiración es estable y que no existen componentes adicionales relevantes. Al convertir esta frecuencia dominante a respiraciones por minuto, se obtiene un valor aproximado de 12 respiraciones por minuto, lo cual se encuentra dentro del rango normal en reposo para un adulto.

<img width="690" height="523" alt="image" src="https://github.com/user-attachments/assets/7a0879ea-b5ea-41cf-bbcb-3ed74ee60127" />

En términos generales, tanto el análisis temporal como el espectral evidencian un patrón respiratorio regular, controlado y fisiológicamente normal. La señal presenta baja variabilidad y una distribución de energía concentrada en una frecuencia principal, lo que es coherente con un estado de reposo y estabilidad autonómica.

## Caso 2: sujeto hablando
### Señal original vs señal filtrada
En la señal respiratoria registrada durante el habla se observa inicialmente una mayor variabilidad en comparación con la condición de reposo. Esto se debe a que al hablar intervienen movimientos adicionales del sistema respiratorio, generando componentes de mayor frecuencia y pequeñas perturbaciones superpuestas a la señal principal. Por esta razón fue necesario aplicar un filtrado pasa-bajas con frecuencia de corte de 1 Hz, considerando que la frecuencia respiratoria fisiológica se encuentra aproximadamente entre 0.1 y 0.6 Hz. Este filtro permite conservar la componente asociada al ciclo respiratorio y eliminar componentes de mayor frecuencia relacionadas con ruido, vibraciones o artefactos producidos por la fonación.

Después del filtrado, la señal en el dominio del tiempo se observa más suave y con una forma más periódica, lo que facilita la identificación de los ciclos de inspiración y espiración. Se reduce notablemente la presencia de oscilaciones rápidas que no corresponden al patrón respiratorio principal, mejorando así la calidad de la señal para su análisis.

<img width="690" height="523" alt="image" src="https://github.com/user-attachments/assets/6eca1f4d-981c-4582-bd80-834f78660934" />

### Espectro de magnitud
En cuanto a la representación en el dominio de la frecuencia, el espectro de magnitud muestra cómo la energía de la señal se distribuye según la frecuencia. En la condición de habla se observa un pico dominante dentro del rango respiratorio fisiológico, lo cual corresponde a la frecuencia respiratoria principal. Sin embargo, en comparación con el reposo, el espectro puede presentar una distribución ligeramente más amplia o componentes adicionales de menor magnitud, debido a la modulación respiratoria causada por el habla. La frecuencia dominante identificada en el espectro permite calcular las respiraciones por minuto, proporcionando una medida cuantitativa del ritmo respiratorio durante la actividad de hablar,para este caso FR= 10.

<img width="690" height="523" alt="image" src="https://github.com/user-attachments/assets/944ec5cc-0005-44e5-8032-894acd5c84c0" />


En general, el filtrado aplicado permitió aislar adecuadamente la componente respiratoria principal, mientras que el análisis espectral permitió identificar de manera clara la frecuencia dominante asociada al patrón respiratorio en esta condición.

# Respuesta a las preguntas planteadas en la guía 

- ### ¿Son los patrones respiratorios y frecuencias respiratorias iguales o diferentes en cada caso? ¿A qué se debe esto?

Los patrones respiratorios son diferentes entre ambas condiciones.En reposo, el patrón es más regular, periódico y estable, con una frecuencia respiratoria constante y menor variabilidad. Esto se debe a que el control respiratorio está principalmente regulado por mecanismos automáticos del sistema nervioso autónomo, sin interferencias externas.
Durante el habla, el patrón se vuelve menos uniforme y presenta mayor variabilidad. Aunque la frecuencia dominante puede mantenerse dentro del rango fisiológico, la respiración se adapta a las necesidades de la fonación, generando modificaciones en la duración de las fases respiratorias y en la amplitud de la señal. El habla requiere control voluntario del flujo de aire, lo que introduce irregularidades y componentes adicionales en la señal.Por lo tanto, las diferencias se deben a la interacción entre el control automático de la respiración y el control voluntario asociado a la producción del lenguaje.

- ### ¿Cuáles serían las ventajas y desventajas de emplear múltiples sensores para el monitoreo del proceso respiratorio? ¿Cuáles podrían ser las razones?
  
El uso de múltiples sensores para el monitoreo del proceso respiratorio presenta varias ventajas, entre ellas una mayor precisión y confiabilidad de las mediciones al permitir la validación cruzada de la información obtenida, así como una mejor caracterización del patrón respiratorio al integrar señales como el movimiento torácico, abdominal o el flujo de aire, lo que resulta especialmente útil en aplicaciones clínicas y de investigación. No obstante, esta estrategia también implica desventajas, como el aumento en el costo del sistema, una mayor complejidad en el procesamiento de señales, mayor susceptibilidad a artefactos por movimiento y una posible incomodidad para el paciente debido al incremento de dispositivos. Por estas razones, el empleo de múltiples sensores suele justificarse en contextos donde se requiere un análisis respiratorio detallado, mientras que para aplicaciones básicas un solo sensor adecuadamente seleccionado y filtrado puede ser suficiente.

# Conclusiones 

El análisis de la señal respiratoria en las condiciones de reposo y habla permitió identificar diferencias claras en el comportamiento del patrón respiratorio, tanto en el dominio del tiempo como en el dominio de la frecuencia. En estado de reposo, la señal presentó un comportamiento periódico, estable y con baja variabilidad, con una frecuencia aproximada de 12 respiraciones por minuto, valor que se encuentra dentro del rango fisiológico normal para un adulto en reposo.
En la condición de habla, la señal mostró mayor variabilidad y menor regularidad en comparación con el reposo. El análisis espectral indicó una frecuencia respiratoria aproximada entre 7 y 10 respiraciones por minuto, valor ligeramente inferior al esperado. Esta disminución podría deberse no solo a la modulación respiratoria propia de la fonación, sino también a posibles errores asociados al sensor de presión, como sensibilidad limitada, desplazamiento de línea base o presencia de artefactos durante los movimientos al hablar.
La aplicación del filtro pasa-bajas fue fundamental para mejorar la calidad de la señal en ambas condiciones, ya que permitió eliminar componentes de alta frecuencia sin afectar la dinámica respiratoria principal. En general, los resultados evidencian la influencia de la actividad funcional sobre el patrón respiratorio y resaltan la importancia del adecuado acondicionamiento y validación del sistema de adquisición para obtener mediciones confiables.

# Referencias


