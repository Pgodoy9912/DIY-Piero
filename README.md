# PIERO-PABLO GODOY CALDERÓN
 
El objetivo principal de este documento es proporcionar una descripción de lo aprendido y mi experiencia realizando el proyecto "Piero". Se centra en la creación y el progreso de Piero, así como en la integración de programas que le permitan recorrer un camino predeterminado, como salir del aula. A su vez se le implementa una navegación reactiva, es decir, que mientras realiza la trayectoria, será capaz de esquivar objetos.

# CREACIÓN DEL ROBOT PIERO

Tanto yo como mis compañeros del grupo 16 hemos creado nuestros propios Piero_DIY individuales. A continuación proporciono los esquemas de hardware de las distintas partes de nuestro robot:

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/683a2609-8d5f-4eac-8fd4-c7931030f116)

### Vídeos de la trayectoria y el comportamiento reactivo ( enlace a Youtube para verlo con mayor calidad)
- Normal: https://youtu.be/p8Euk4crhm0

- Cámara rápida: https://youtu.be/LMlTnfIM1Uo

## Sonars

En este circuito observamos las distintas conexiones que tienen los sensores de ultrasonidos US-016 :

![Sonars](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/8b4b711b-042c-42f2-868b-c063c63e6f1d)<br>
Figura 1 : Esquemático Hardware de los sensores US-016, creado en Fritzing

## Motores

Se han implementado dos motores con sus encoders mediante el módulo L298N. A su vez, en este esquema se muestra la fuente de alimentación de 12 V que tiene nuestro robot, la cuál viene controlada por un interruptor. Para visualizar el voltaje que tienen las baterías se han implementado tanto un voltímetro visual como un sensor de voltaje. En la siguiente figura se ve su conexionado:

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/792fb6df-e6a1-462f-8fde-d0a7d9b2d875)<br>
Figura 2 : Esquemático Hardware de los motores y las baterías, creado en Fritzing

## Leds

En este esquema hardware se muestran las conexiones del módulo de led RGB:

![Leds](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/89b4e361-fa01-49d4-9ce1-0cfb1d600e63)<br>
Figura 3 : Esquemático Hardware del módulo Led RGB, creado en Fritzing


# ARDUINO EN SIMULINK

Ahora mostraré como hemos ido creando los distintos bloques que usaremos en simulink para simular nuestro Hardware. Principalmente usaremos la libreria *"Simulink Support Package for Arduino Hardware"*.

## Sonars
Bloques que hemos usado:
 - Analog Input: mide el voltaje del pin de la entrada analógica, que puede tomar un valor en el rango entre 0 y 1023. En nuestro caso, los pines analógicos que leeremos serán el 2 ( sensor izquierdo) y el 3 ( sensor derecho).
 - Gain: multiplica la entrada por un valor constante. Lo usamos para pasar de voltaje a cm. 

Procedimiento:

 1. Primero creamos un subsistema vacío, en el implementaremos la lógica necesaria para recoger los datos de entrada de los pines analógicos 2 y 3, juntarlos en un multiplexor (los datos del lado izquierdo siempre irán primero), multiplicarlos por una ganancia para pasarlos a cm y sacarlos por una salida.

 ![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/fc4f7c2f-fe79-4fb2-8c66-c25290b7c25a)<br>
Figura 4 : Subsistema  SonarsAG, Simulink

 2. Calculamos la ganancia, que finalmente tomo un valor de $\frac{1}{3.46}$. Al principio teníamos el razonamiento de que la ganancia tendría que ser $\frac{1}{10.23}$, ya que al ser una entrada analógica, va de 0 a 1023, y para pasarlo a cm pues multiplicamos por 100. Pero tras hacer pruebas, nos dimos cuenta que no dama lecturas correctas, así que aplicando relaciones de proporcionalidad con la medida real y la obtenida, llegamos a la ganancia mencionada anteriormente.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Para poder realizar dichas pruebas, hemos tenido que crear un nuevo modelo en simulink, llamado "TestSonar". En él usamos el bloque Subsystem reference, el cual nos permite cargar cualquier subsistema creado, en nuestro caso el de SonarsAG, y otorgarle un display a la salida para poder verla lectura.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/a6ce5854-ab81-49fb-b53c-c543e47b0a81)<br>
Figura 5 : Model  TestSonar, Simulink

## Motores
Bloques que hemos usado:
 - PWM: lo usamos para proporcionar la salida que dictará el movimiento de los motores. Su entrada tendrá que estar entre 0 y 255.
 - Digital Output: este bloque, al ser de salida digital, solo toma valores de 0 y 1. En nuestro caso es usado como un enable para que los motores puedan funcionar.
 - Saturation:  permite que la señal que le sale este limitada a unos valores inferiores y superiores. Es decir, si le entra una señal dentro de los límites, devuelve la misma, pero si le entra una señal superior o inferior que los límites devuelve los límites.

 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;En nuestro subsistema tiene la función de hacer que la rueda se mueva para adelante o para atrás. Si a la entrada introducimos un valor de -100, es porque queremos que se mueva hacia atrás, este bloque permite que al PWM que controla el movimiento hacia adelante le entre un 0, y el que controla el movimiento hacia atrás le entre el -100. Nuestros limites serán 0 a 255 para adelante y -255 a 0 para atrás.
 - Abs: sirve para poder obtener el valor absoluto de la señal que queramos.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Como el PWM no lee valores negativos, solo positivos ( ya que lo que modificamos es la velocidad, y no puede ser negativa), se coloca delante de la saturación de -255 a 0 ya que devolvería valores negativos.
 - Compare to constant: hace la función de una especie de if. Si la entrada que le llega es distinta de la constante asignada, este devolverá un 1, pero si es igual, devolverá un 0.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Nos permite que si la entrada a un motor es 0, es decir, que no se mueva, el enable esté desactivado ya que le llegará un valor lógico de 0.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/90b55d65-2852-4d14-81f2-a7d0b044cc68)<br>
Figura 6 : Subsistema  MotoresJP, Simulink

Procedimiento:

 1. Primero creamos un subsistema vacío y colocamos 4 PWM, dos por cada rueda y uno para cada dirección. A su entrada se le colocará la saturación y si es pertinente, el valor absoluto, para dejar las señales a valores de PWM ( de 0 a 255). También incluimos dos Digital Outputs para que hagan la función de enable de los motores, y a su entrada el bloque Compare to constant para poder activar o desactivar dicho enable según la entrada el motor. Finalmente se colocará un demultiplexor, ya que la entrada otorgará una señal con valores para ambos motores, y habrá que separarlos en izquierdo y derecho.


 2. Para poder observar si funciona todo correctamente, se usará el bloque Subsystem Reference para poder usar el subsistema de motores.  A su entrada se conectará un bloque de Constante, en el cual escribiremos dos números con el formato [100 100]. El primer numero introducido determinará el movimiento de la rueda izquierda, y el segundo el de la derecha.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/6157f054-0041-40cf-bd06-443b0d5acb81)<br>
Figura 7 : Model  TestMotores, Simulink


## Leds
Bloques que hemos usado:
 - Digital Output: bloque de salida digital que pondrá a 1 o a 0 el pin digital seleccionado. Lo usamos para encender o apagar el led, según si se cumplen o las condiciones.
 - Puerta lógica AND : hace la operación de la multiplicación, con que una de sus entradas sea 0, su salida ya será 0.
 - Puerta lógica OR : hace la operación de la suma, con que una de sus entradas sea 1, su salida ya será 1.
 - Compare to constant: hace la función de una especie de if. Si la entrada que le llega es distinta de la constante asignada, este devolverá un 1, pero si es igual, devolverá un 0.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;El valor al que se compara es 20, es decir, que ocurrirá algún cambio en la iluminación cuando se detecte un objeto a menos de 20 cm.
 - Pulse Generator: genera una señal de pulsos cuadrados.
 
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;La usamos para generar el parpadeo del led. Los valores que hemos seleccionado han sido un periodo de 1 segundo y un duty cicle de 50%, para que la frecuencia de parpadeo del led fuera de medio segundo.
  - Switch: tiene tres entradas, si la segunda entrada cumple con la condición impuesta, la salida será la primera entrada, si no la cumple, la tercera.
 
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Su función es la de hacer que el led parpadee o no dependiendo de si se detecta obstáculo por los dos lados o no.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/0011a321-dd9d-4ebc-9cf7-b84d8fab441b)<br>
Figura 8 : Subsistema  LedsPGC, Simulink

Razonamiento:

La función que debe cumplir este bloque es la de cambiar el color del led o su frecuencia de parpadeo siguiendo.
 - Si el obstáculo es detectado por la izquierda, debe encenderse el led en rojo sin parpadear.
 - Si el obstáculo es detectado por la izquierda, debe encenderse el led en azul sin parpadear.
 - Si el obstáculo es detectado en frente, es decir, por izquierda y por derecha, se debe poner el led en rojo y a parpadear.
 - Si no se detecta obstáculo, el led se encenderá en verde y sin parpadear.

Para llevar este razonamiento a Piero hemos realizado el siguiente conexionado:
 - Si la lectura del lado izquierdo es de menos de 20 cm, se mandará un 1 a la puerta OR1, la que devolverá un 1, el cuál irá a la AND5, la cuál devolverá 0 o 1. Esto dependerá de si la otra señal es un 1 o el pulso cuadrado, es decir, de si se detecta obstáculo por los dos lados ( pulso cuadrado) o no ( constante 1).
 - Si se lee que la lectura del lado derecho es menor de 20 cm, se mandará un 1 a la puerta AND3. Si se cumple que solo se detecta por el lado derecho, tanto la NOT1 como el Switch devolverán un 1, lo que encenderá el led azul. La función de la AND1 y de la NOT1 es hacer que solo se encienda el azul si el rojo no está ya encendido, ya que teníamos el problema que cuando detectaba a ambos lado, el led se ponía morado.
 - Si no se detecta un obstáculo a ningún lado a menos de 20 cm, la puerta AND2 dará a su salida un valor de 1, que junto con el 1 que otorga el Switch, hará que el led se encienda verde.
 - Si sí se detecta a ambos lados, la puerta AND1 junto con la NOT1 devolverá un 0, apagando así le led azul, pero dejando encendido el led rojo, ya que se está detectando algo por el lado izquierdo. Esto a su vez cambia la salida del Switch, que pasa a ser el pulso cuadrado, y por lo tanto, el led parpadeará.

Procedimiento:

1. Primero creamos un subsistema vacío y colocamos toda la lógica mencionada anteriormente. Para separar entre lado izquierdo y derecho, volveremos a recurrir a un demultiplexor.

2. Para realizar la comprobación, se procede a poner en un modelo vacío dos Subsystem Reference, una para el subsistema de SonarsAG y otra para el subsistema de LedsPGC, siendo la salida del primero la entrada del segundo.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/d9127bb0-fd40-4f6c-8da1-c24ff9fa59da)<br>
Figura 9 : Model  TestLed, Simulink

## Navegación Reactiva Básica
Con estos bloques ya se puede realizar una inicial navegación reactiva que evite obstáculos. A la entrada de los motores irá la lectura de los sonars multiplicada por una ganancia ( para que la velocidadnos ea tan baja). Si se detecta un obstaculo por un lateral, esa distancia será más pequeña, lo que hara que la velocidad de la rueda del lado detectado baje y el robot gire hacia el otro lado, evitando así el objeto.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/c8ce8906-28fe-4acc-b726-9903411daeef)<br>
Figura 10 : Model  ControladorPGC, Simulink

## Encoders (Hardware)
Bloques que hemos usado:
 - Encoder: lee la posición y la velocidad rotacional del motor desde un encoder conectado a un MKR Motor Carrier. La posición es medida en ticks y la velocidad en ticks por segundo. Cada incremento de la cuenta de ticks indica una rotación en el sentido de las agujas del reloj, y cada decremento en la cuenta de ticks indica uan rotación en el sentido contrario a las agujas del reloj.
 - Gain: multiplica la entrada por un valor constante. Lo usamos para pasar de voltaje a $\frac{m}{s}$. 
 - Discrete derivative: sirve para calcular como varía su entrada en el tiempo y aproximarla.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/c1fe23a5-9d16-45a6-a345-09ef2e32691f)<br>
Figura 11 : Subsistema  HEncodersV, Simulink

Prodecimiento:

 1. Primero creamos un subsistema vacío, en el que añadiremos dos bloques de encoders ( derecho e izquierdo). En ellos determinaremos los pines de lectura (18 y 19 para la rueda izquierda, y 2 y 3 para la derecha) y los multiplicamos por una ganancia para pasar la lectura que nos otorgan a $\frac{m}{s}$.
 
 2. Calculamos la ganancia. Los encoders nos proporcionan los ticks generados en una rotación, es decir, un pequeño movimiento angular. Para pasar dicho movimiento angular a distancia, la multiplicamos por l circunferencia de la rueda, es decir $\pi$ x diámetro. Eso lo dividiremos entre el número de ticks medidos en 10 vueltas entre 10, para normalizar. Finalmente, añadiremos el bloque de Discrete Derivative para poder obtener la variación de la distancia en el tiempo y aproximarla, pasando así a $\frac{m}{s}$.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Para poder calcular el numero de ticks en 10 vueltas, debemos crear un modelo en blanco y añadir dos Subsystem Reference, una para los encoders y otra para los motores. Luego se añadirá una constante a los motores para que empiecen a girar las ruedas y se contaran 10 vueltas y se parará el sistema. Dicho valor obtenido en un display a la salida del encoder será el que usemos para la ganancia.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/b7222de2-cdf5-426e-a3db-007ea9597d7e)<br>
Figura 12 : Model  TestHEncodersV, Simulink

## Encoders 
Este subsistema tendrá la misma función que el explicado anteriormente, pero con la particularidad de que en vez de usar directamente los bloques de lectura de encoder que nos proporciona la librería, vamos a usar un S-Function Builder.

Primero explicar que una S-Function es una función que permire ampliar simulink con bloques codificados manualmente, es decir, si no existe un bloque, lo podemos programar. En nuestro caso vamos a intentar codificar un bloque que haga de lectura de encoder. Para ello usaremos un código en C ya otorgado por el profesor en campus y añadiremos solo las secciones necesarias al nuestro código.

El procedimiento para calcular la ganancia es el mismo que para el subsistema HEncodersV.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/cc57f248-8d78-45ec-ad0b-9305b41b04b1)<br>
Figura 13 : Subsistema  EncodersV, Simulink


# SIMULACIÓN DEL SISTEMA
Para poder realizar pruebas del sistema en modo simulación, primero debemos obtener una aproximación del sistema de forma de bloque o función.

Para ello estudiaremos la respuesta del sistema a una señal escalón y a una señal rampa. Para generar dichas funciones usaremos el bloque *"Signal Builder"*, donde pondremos una máxima amplitud de 1, ya que para llegar a los valores que queramos usaremos una ganancia de 127 ( debería ser 255 ya que es el valor máximo de PWM, pero siempre que intentamos aumentarla de 127 nos salta un error). Las señales quedarán:

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/90480d8a-b381-496e-876d-2fa6964fb9e2)<br>
Figura 14 : Señal rampa creada en Signal Builder

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/91fe3ff2-5e0d-4fa1-baca-77fcbd640234)<br>
Figura 15 : Señal escalon creada en Signal Builder

La salida de esta señal, multiplicada por la ganancia, será la que guardemos en una tabla creada gracias al bloque "To Workspace". Además de la rampa o el escalón, en la tabla que creamos y mandamos al workspace también guardaremos las velocidades de ambas ruedas (gracias al subsistema EncodersV) y el voltaje proporcionado por las baterías. Este último lo medimos gracias al sensor de voltaje, que manda su medida al pin analógico 7. La lectura proporcionada por este pin es multiplicada por una ganancia de $\frac{25}{1023}$ (la explicación es que el sensor mide hasta 25V y la entrada hay que pasarla de valor analógico (0 a 1023) a número).

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/38ff76c5-9349-4613-9d49-98c53ce9b9bb)<br>
Figura 16 : Model  DatosIdent, Simulink

## Ident
Con los datos obtenidos de la simulación frente a un escalón obtendremos dos señales de transferencia que imiten el comportamiento de nuestro Piero. Para poder lograr esto deberemos usar la herramienta de matlab *"System Identification"*, donde cargaremos tanto la respuesta al escalón de la rueda izquierda como de la derecha ( escogiendo la columna 1 para izquierda y 2 para derecha). Una vez hecho esto, procederemos a seleccionar solo la parte que nos interesa.

Con las señales deseadas procedemos a crear funciones de transferencia, en especial nos quedaremos con la que tiene un polo y ningún cero.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/79a78f3c-d8ae-40e1-9d14-f3e7648415e0)<br>
Figura 17: Herramienta System Identification

Una vez hemos obtenido las señales de transferencia, creamos un subsistema que hará de nuestro robot en las simulaciones. Dicho subsistema contendrá dos Discrete Transfer Fcn, las cuales modificaremos y pondremos los valores obtenidos en las funciones de transferencia creadas anteriormente.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/2eac8c02-b046-4f9f-b3a4-92bc74ce3aac)<br>
Figura 18 : Subsistema  PieroPGCModel, Simulink

Para facilitar la tarea y no tener que modificar de manera manual el cambio a simulacion y a hardware, crearemos un subsistema llamado PieroPGC, el cual simula a nuestro robot en ambos casos. Para que esto sea posible de manera automática, usamos el bloque *"Variant Source"*, el cual tiene dos entradas, y según si estamos en simulación o en hardware, pone a la salida una u otra.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/c0639ead-6d8b-4b5d-8ec9-446d86b78467)<br>
Figura 19 : Subsistema  PieroPGC, Simulink


# CONTROL BUCLE ABIERTO
El primer control que realizaremos será el de bucle abierto, es decir, sin retroalimentación. Para ello crearemos una especie de controlador, el cual será un subsistema que constará de dos LookUp Tables ( una para cada rueda). Los valores de la tabla vendrán de la simulación frente a rampa que realizamos anteriormente.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/cf609002-b925-49fb-a8ec-afc03fa186f3)<br>
Figura 20 : Subsistema  ControladorBA, Simulink

Para comprobar su funcionamiento y que todo está correcto, procedemos a crear un modelo en blanco al que denominaremos ControlBA. En el añadiremos los subsistemas de ControladorBA y de PieroPGC. El primero será el encargado de mandar al modelo PieroPGC el valor de PWM correspondiente a la entrada de velocidad, establecida mediante una constante. El segundo tendrá la función de devolver el comportamiento de nuestro robot. Para visualizar el resultado, usaremos tanto un dsplay para ver el valor final como un scope, para ver la evolución temporal.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/c8093e94-7b0d-4f52-bbc5-1457afa0f9d6)<br>
Figura 21 : Model ControlBA y salida en simulación

Como se observa en la figura, el sistema realiza su función más o menos decente, ya que no llega a la consigna deseada pero se queda cerca.

# CONTROL BUCLE CERRADO
Tras el control el bucle abierto, toca realizar el de bucle cerrado, ya que si alguna perturbación llega al sistema, con el bucle abierto no podremos corregirla, pero con el cerrado sí.

Los pasos a seguir son similares al control en bucle abierto. Primero crearemos un subsistema ControladorBC, el cual consistirá de dos PID, uno para cada lado. A su salida pondremos el subsistema PieroPGC, que sería nuestro robot. A su salida hacemos la realimentación del sistema. Tras la realimentación, colocaremos de nuevo un display y un scope para visualizarla salida.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/a506dd62-a103-4afa-806a-611939191376)<br>
Figura 22 : Subsistema ControladorBC

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/cd6234be-81cd-4192-87c9-f7e86268bc11)<br>
Figura 23 : Model ControlBC y salida en simulación


# CINEMÁTICA Y CONTROL DE ORIENTACIÓN
## Cinemática
Haremos un estudio de la cinemática diferencial, es decir, ignorando las fuerzas involucradas. Esto nos facilitará el entender como cómo se mueve nuestro robot y como varían sus velocidades y orientaciones según que consigna de control. Consideramos que el robot se mueve en un suelo plano ( plano XY).

Al modelo cinemático directo le debemos dar en la entrada las velocidades articulares ( de cada rueda) y  nos devolverá las velocidades cartesianas (V y W). En nuestro proyecto, crearemos el modelo cinemático directo con un subsistema. En él crearemos una matriz que lleve las operaciones necesarias para pasar de velocidades articulares a cartesianas ( la d equivale a la distancia entre las ruedas).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;En la imagen se puede observar un Variant Source y un bloque que realiza la traspuesta, esto es debido a que cada vez que hacíamos la simulación no nos daba error, pero al ejecutarlo en el hardware si nos daba un error de dimensiones. Como desconocíamos el origen de este error, decimos poner esta solución.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/372a8034-3def-4389-882c-b3d904d5c492)<br>
Figura 24 : Subsistema MCD

El modelo cinemático inverso realizará la función contraria. A unas velocidades cartesianas devuelve sus correspondientes velocidades articulares. En nuestro proyecto, también crearemos el modelo cinemático inverso como otro subsistema. En él crearemos una matriz que lleve las operaciones necesarias para pasar de velocidades articulares a cartesianas ( la d equivale a la distancia entre las ruedas).

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/543c2816-d187-4139-9b69-2ae879e6ba8f)<br>
Figura 25 : Subsistema MCI

Para poder obtener una representación en el plano XY del movimiento que lleva recorrido nuestro robot, necesitamos saber su odometría, es decir, la estimación de la posición y la orientación según las velocidades cartesianas. Para ello primero deberemos separar la velocidad V en las dos direcciones del plano Vx y Vy, para ello usaremos el coseno y el seno. Tras obtener esto, integraos velocidades para obtener la x,y y el ángulo theta. Finalmente dichas magnitudes se podrán representar en un XY graph.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/7c3cce81-8b61-4605-b5c8-5583c87013e1)<br>
Figura 26 : Subsistema Odometria

Una vez hemos creado todos los subsistemas pertinentes, podemos proceder a crear un modelo para ver el comportamiento del sistema. En el la consigna de entrada será una velocidad linear y una angular, por lo que deberemos aplicar el MCI para pasarlo a velocidades articulares y que nuestro Piero pueda leerlas y realizar los movmientos pertinentes. Tras esto se aplicará el MCD con el fín ed volver a obtener velocidades cartesianas y poedr tanto ver la salida más clara como poder representar correctamente el movimiento en el XY graph.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/e17ff99e-2cf7-43aa-be1d-7c8af4a39ee7)<br>
Figura 27 : Model PieroCinemática


## Control de la orientación
Pero al no tener ningún bucle de retroalimentación, el anterior modelo tendrá los mismos fallos que en bucle abierto. Por ello, para poder girar el ángulo correcto y poder seguir la trayectoria deseada, debemos realimentar en ángulo de rotación theta. Para eso usaremos un demultiplexor.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/d80192c4-24e5-4e59-8694-35300a101b20)<br>
Figura 28 : Model PieroHorientacion

# CONTROL DE TRAYECTORIAS
Para poder implementar trayectorias y que nuestro robot pueda seguirlas, lo que cambiaremos serán las consignas de entrada, que pasarán a ser señales más complejas que varían su valor según nos convenga. En nuestro caso debemos realizar una trayectoria que siga los siguientes pasos:
 1. 4 metros en línea recta hasta llegar al pasillo.
 2. Giro antihorario de 90º
 3. 11,2 metros de línea recta
 4. Giro antihorario de 90º
 5. 0.2 metros de linea recta
 6. Giro horario de 90º
 7. 4.1 metros de linea recta
 8. Giro horario de 90º
 9. Salimos de clase

## Signal Builder
Para poder realizar estos pasos, se ha procedido ha implementar la siguiente función, donde la velocidad permanecerá constante y solo variamos la orientación.
![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/6f73c08e-e44e-4d3a-853a-1c7710050509)<br>
Figura 29 : Señal encargada de realizar la trayectoria

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/765653c3-89f3-49d9-8556-67ccedd7a0eb)<br>
Figura 30 : Model SalirClaseSB


## Diagrama de estados
Otra forma de implementar una trayectoria es, en vez de con una señal, con un diagrama de estados. El diagrama de estado aporta mejor visibilidad de la trayectoria y un manejo más intuitivo. Para realizar un diagrama de estados en Simulink recurrimos a la librería *"Stateflow"*, en específico el bloque Chart. El razonamiento será parecido. Segun la velocidad que llevamos, como sabemos la distancia a recorrer, pondremos condiciones de tiempo para poder desplazarnos la distancia correcta. La trayectoria final quedaría de la siguiente forma:

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/0dd9a3eb-ba60-4f15-87c1-925c9c2a2745)<br>
Figura 31 : Diagrama de estados de la trayectoria.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/7d2cd1bd-ce98-4902-a4e9-4653e36d3df3)<br>
Figura 32 : Model SalirClaseDE

# SEGUIMIENTO DE TRAYECTORIAS
Otra forma de seguir una trayectoria puede ser dividirla en puntos, llamados waypoints, e ir desplazandote de waypoint a waypoint. En nuestro caso, los waypoint corresponderán a los puntos donde hay un cambio significativo de trayectoria ( un cambio de dirección, de sentido...)

## Persecución Pura
Para este caso usaremos el boque de Simulink *"Pure  Pursuit"*, de la librería *"Robotics System Toolbox"*. Sus entradas son la pose actual donde estamos y los waypoint a los que hay que ir. El bloque propio calcula la velocidad angular y linear necesaria para ir hasta el waypoint.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/7afb8911-b350-4657-bbe1-65ecabcca053)<br>
Figura 33 : Model SalirClaseWaypointsPP

## Mfunction
Otro bloque que podemos usar es el bloque Mfunciton, propio de Simulink, en el cual podemos escribir una función en su interior (sistema parecido al del SFunction ). La función que escribiremos en este bloque tiene como salida la velocidad, el ángulo theta, un índice de posición. Para poder calcular estas salidas, necesitará el camino que vamos a recorrer (waypoints), la posición (x,y) actual y una variable i, que será la que nos permita pasar por los distintos waypoints.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/79b3dd5e-14e5-4111-9325-54c2d00ff18c)<br>
Figura 34 : MFunction MyPurePursuit

La función tiene el siguiente funcionamiento:
 - Definimos las variables de velocidad y angulo inicial, el índice i y una distancia L (que determinará como de cerca debe estar nuestro robot de un waypoint para pasar al siguiente).
 - Comprobamos que todavía nos quedan waypoints por recorrer, si la respuesta es afirmativa, calculamos la distancia entre nuestra posición actual y nuestro siguiente punto.
 - Si la distancia es menor que L y sigue habiendo waypoints, nos pasamos al siguiente waypoint, y volvemos a calcular la distancia y el ángulo entre la posición actual y el siguiente waypoint y ponemos una velocidad linear de 0.205 para que el robot se desplace al siguiente waypoint. Cuando esta condicion se deje de cumplir y cambiemos de waypoint al que dirigirnos, se calculará el ángulo entre la posición actual y el siguiente punto al que tenemos que desplazarnos.
 - Si no quedan waypoints por recorrer, se ponen ambas velocidades a 0, ya que hemos llegado al destino.
 - Finalmente comprobamos si el indice i es menor al número de waypoints que tenemos, es caso de ser así, el indice il de la salida será el valor de i-1, ya que hemos recorrido un waypoint más. Pero si i es mayor, significa que todavía no hemos pasado un waypoint, por lo que i=il.


![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/11fdb1fb-c335-43a5-9a96-1586e67c19e0)<br>
Figura 35 : Model SalirClaseWaypointsMF

## Aceleración limitada con evitación de obstáculos
Finalmente, si queremos añadir a nuestro Piero una navegacón reactiva, es decir, que esquive obstaculos, añadiremos al modelo SalirClaseWaypointsPP un Diagrama de estados que se encarge de esquivar los obstáculos. La lógica de control es fácil, si se detecta algún objeto, el switch hará que se cambie de modo navegación a modo reactivo. Cuando se haya esquivado el objeto, se seguirá con la trayectoria normal volviendo a ir al waypoint al que inicialmente ibamos a ir antes de detectar el obstáculo.

El diagrama de estados tendrá como estado incial la no detección de obstaculos, la cual mandará un 0 como señal de control y por lo tanto, el comportamiento reactivo estará desactivado. En cuanto detecte algún obstaculo por la izquierda, se irá al estado *"ObstaculoIzq"*, en donde se procede a girar a la derecha. Cada tres segndos se comprueba si ya se ha esquivado o no dicho obstáculo, si además de detectarlo por la izquierda lo detecta por la derecha tambien, quiere decir que el obstáculo está de frente, por lo que habrá que girar más. Si se detecta por la derecha, el razonamiento es el mismo, salvo que esta vez estaremos en el estado *"ObstaculoDcha"* y el giro se hará a la izquierda.

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/842ef49c-97a0-45f9-967d-61e62285c9a9)<br>
Figura 36 : Diagrama de estados para esquivar obstáculos

![image](https://github.com/Escuela-de-Ingenierias-Industriales/LaboratorioRobotica23-Pgodoy9912/assets/145780052/3f2bbb28-dba1-4bae-9e68-d99310f5f834)<br>
Figura 37 : Model SalirClaseReactivo
