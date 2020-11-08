# LAB8_ARSW 🚀
**_Integrantes:_**


* _Angie Daniela Ruiz Alfonso_
* _Juan Sebastian Díaz Salamanca_ 

### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/en-us/free/search/?&ef_id=Cj0KCQiA2ITuBRDkARIsAMK9Q7MuvuTqIfK15LWfaM7bLL_QsBbC5XhJJezUbcfx-qAnfPjH568chTMaAkAsEALw_wcB:G:s&OCID=AID2000068_SEM_alOkB9ZE&MarinID=alOkB9ZE_368060503322_%2Bazure_b_c__79187603991_kwd-23159435208&lnkd=Google_Azure_Brand&dclid=CjgKEAiA2ITuBRDchty8lqPlzS4SJAC3x4k1mAxU7XNhWdOSESfffUnMNjLWcAIuikQnj3C4U8xRG_D_BwE). Al hacerlo usted contará con $200 USD para gastar durante 1 mes.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica


![Imágen 1](images/part1/part1-vm-basic-config.png)


2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM.

    `ssh scalability_lab@xxx.xxx.xxx.xxx`


3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).


4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`


5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    `npm install forever -g`

    `forever start FibinacciApp.js`


6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.


![](images/part1/part1-vm-3000InboudRule.png)


7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:


    * 1000000 - tiempo: 23 segundos.
    * 1010000 - tiempo: 23.39 segundos.
    * 1020000 - tiempo: 24.16 segundos.
    * 1030000 - tiempo: 24.51 segundos.
    * 1040000 - tiempo: 25.46 segundos.
    * 1050000 - tiempo: 26.13 segundos.
    * 1060000 - tiempo: 26.71 segundos.
    * 1070000 - tiempo: 27.10 segundos.
    * 1080000 - tiempo: 27.82 segundos.
    * 1090000 - tiempo: 28.74 segundos.
    

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).


![Imágen 2](images/part1/part1-vm-cpu.png)


9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ``
    `

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.


![Imágen 3](images/part1/part1-vm-resize.png)


11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.


    * 1000000 - el tiempo anterior de 23 segundos pasó a 20.90 segundos ahora.
    * 1010000 - el tiempo anterior de 23.39 segundos pasó a 20.70 segundos ahora.
    * 1020000 - el tiempo anterior de 24.16 segundos pasó a 21.65 segundos ahora.
    * 1030000 - el tiempo anterior de 24.51 segundos pasó a 22.03 segundos ahora.
    * 1040000 - el tiempo anterior de 25.46 segundos pasó a 21.82 segundos ahora.
    * 1050000 - el tiempo anterior de 26.13 segundos pasó a 22.36 segundos ahora.
    * 1060000 - el tiempo anterior de 26.71 segundos pasó a 21.63 segundos ahora.
    * 1070000 - el tiempo anterior de 27.10 segundos pasó a 23.08 segundos ahora.
    * 1080000 - el tiempo anterior de 27.82 segundos pasó a 23.29 segundos ahora.
    * 1090000 - el tiempo anterior de 28.74 segundos pasó a 23.48 segundos ahora.


12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo. _Rta: Sí se cumple, este hace un mayor consumo de CPU pero realiza el trabajo de manera más rápida._


13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM? _Creó 3 recursos, los cuales son: la máquina virtual, un grupo encargado de la seguridad de red, un grupo encargado de los recursos, una red virtual, una interfaz de red, una dirección IP pública, un disco y una cuenta de almacenamiento para los datos almacenados de Azure._


2. ¿Brevemente describa para qué sirve cada recurso? _La máquina nos sirve para trabajar sobre el sistema operativo Linux externo. El grupo de seguridad de red nos sirve para el manejo del control de las reglas de seguridad de entrada y salida de la máquina. La interfaz de red permite la comunicación de la máquina virtual con Internet y con los recursos locales. El grupo de recursos nos sirve para la administración de todos los recursos que tenemos a nuestra disposición. El disco es el que se encarga del almacenamiento de la máquina virtual. La red virtual nos permite la comunicación entre los recursos de las máquinas virtuales de Azur, como una red común pero con servicios de escalabilidad, disponibilidad y aislamiento. La dirección IP píblica se nos asigna hasta que la máquina se haya eliminado, ésta nos permite que los recursos de azure se comuniquen a través de internet y con los servicios que son publicos en azure, por ejemplo: Interfaces de red de máquinas virtuales, balanceadores de carga orientados a Internet, pasarelas VPN, pasarelas de aplicación y cortafuegos Azure. La cuenta de almacenamiento para los datos almacenados de Azure nos proporciona un espacio de nombres únicos para los datos de Azure Storage y que se pueda acceder a ellos desde cualquier lugar del mundo a través de HTTP o HTTPS, datos que cumplen con alta disponibilidad, seguridad y escalabilidad._


3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? _Esto se debe a que ya no hay conexión con la máquina que nos esta brindando este servicio._ ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio? _Debemos crear ésta porque por defecto la máquina solo tiene acceso externo a los puertos 22 y 80, y este servicio corre por el puerto 3000._


4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo. _La función tarda tanto tiempo debido a que tiene una CPU y una memoria muy limitada, además de ello la aplicación no esta construida de manera óptima (no aprovecha de manera adecuada los recursos del sistema al estar implementada iterativamente y al no usar más hilos, además de ello realiza cálcuos innecesarios y estos resultados no son almacenados en memoria)._


![alt text](https://raw.githubusercontent.com/angiedanielar/LAB8_ARSW/main/images/1.png)


5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU. _La función consume esa cantidad de CPU debido a que la aplicación no esta construida de manera óptima (se realizan múltiples cálculos innecesarios, además de no manejar concurrencia), por ende consume una cantiad elevada de recursos._


![alt text](https://raw.githubusercontent.com/angiedanielar/LAB8_ARSW/main/images/2.png)


6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición. - 19 segundos. 
    * Si hubo fallos documentelos y explique. - 5 fallos iguales, al no soportar concurrencia.
    
    
    ![alt text](https://raw.githubusercontent.com/angiedanielar/LAB8_ARSW/main/images/3.png)
    
    
    * _Fallo1: Error al momento de la conexión, debido a que ésta se reinció._
    * _Fallo2: Error al momento de la conexión, debido a que ésta se reinció._
    * _Fallo3: Error al momento de la conexión, debido a que ésta se reinció._
    * _Fallo4: Error al momento de la conexión, debido a que ésta se reinció._
    * _Fallo5: Error al momento de la conexión, debido a que ésta se reinció._
    
    
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)? _Las diferencias entre B2ms y B1ls son:._

   * La CPU. 
   * La RAM.
   * Los sistemas operativos que soportan. 
   * La velocidad de transmisión de datos por segundo.
   * **Éstas anteriores pueden afectar de manera significativa el desempeño de la máquina virtual.**
   * El precio del uso de la infraestructura de Azure.
   * El uso general que se le da.
 

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM? _Si funciona para que la aplicaicón se ejecute de manera más rápida pero no es una buena solución porque el consumo de CPU aumenta a la par. Además cuando cambiamos el tamaño de la máquina virtual es necesario reiniciarla, por ende se pierde el prinicpio de disponibilidad de la aplicación por lo que esta deja de funcionar mientras se reinicia._


9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica? _La máquina se vuelve más rápida, pero el efecto negativo es el aumento del consumo por parte de la CPU, además de la afectación en el principio de disponibilidad de la aplicación mientras se restablece._


10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué? _En el consumo de CPU no hubo mejora debido a que en ambos casos se consume bastante CPU. Pero en los tiempos de ejecución si hubo mejora pero no significativa._


11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor? _El comportamiento no es porcentualmente mejor, el programa falla totalmente desde el principio._

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.


1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.


![](images/part2/part2-lb-create.png)


2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.


![](images/part2/part2-lb-bp-create.png)


3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.


![](images/part2/part2-lb-hp-create.png)


4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.


![](images/part2/part2-lb-lbr-create.png)


5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.


![](images/part2/part2-vn-create.png)


#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.


1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.


![](images/part2/part2-vm-create1.png)


2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.


![](images/part2/part2-vm-create2.png)


3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.


![](images/part2/part2-vm-create3.png)


4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.


![](images/part2/part2-vm-create4.png)


5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto


```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```


Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:


```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1

```

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.


    * 1000000 - tiempo: 29.55 segundos.
    * 1010000 - tiempo: 27.63 segundos.
    * 1020000 - tiempo: 30.91 segundos.
    * 1030000 - tiempo: 30.07 segundos.
    * 1040000 - tiempo: 34.40 segundos.
    * 1050000 - tiempo: 34.41 segundos.
    * 1060000 - tiempo: 32.08 segundos.
    * 1070000 - tiempo: 35.01 segundos.
    * 1080000 - tiempo: 32.44 segundos.
    * 1090000 - tiempo: 32.55 segundos.
    
    ![alt text](https://raw.githubusercontent.com/angiedanielar/LAB8_ARSW/main/images/4.png)
        

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.


```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

_VM1:_


![alt text](https://raw.githubusercontent.com/angiedanielar/LAB8_ARSW/main/images/5.png)


_VM2:_


![alt text](https://raw.githubusercontent.com/angiedanielar/LAB8_ARSW/main/images/6.png)


_VM3:_


![alt text](https://raw.githubusercontent.com/angiedanielar/LAB8_ARSW/main/images/7.png)


_VM4:_


![alt text](https://raw.githubusercontent.com/angiedanielar/LAB8_ARSW/main/images/8.png)

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?


   _Azure Load Balancer: Tiene la misma función que un balanceador de carga de tipo NLB, donde las máquinas virtuales se conectan al back-end del balanceador de carga y la dirección IP pública se asigna al front-end. Este ambién implementa el servicio de NAT, el NAT inverso recibe la petición en la IP pública y la redirige a la máquina que el algoritmo considere como óptima en cada momento._


   _Azure Application Gateway: Este ofrece capacidades de balanceo de carga geográfica, con lo cual podemos balancear carga entre diferentes regiones de Azure y garantizar así que los usuarios siempre acceden a la región que tienen más cercana, también tiene capacidades de un Web Application Firewall, SSL Offloading, descarga a los servidores del proceso de cifrar y descifrar la información mediante SSL y enrutamiento basado en cookies de sesión._

   _Azure Traffic Manager: Es la parte del Application Gateway que se encarga del balanceo de carga entre las regiones de Azure._


* ¿Cuál es el propósito del *Backend Pool*? _Este es un componente que tiene el balacenador de carga, el cual define el grupo de recursos que brindarán tráfico para una Load Balancing Rule determinada, este es un grupo de máquinas virtuales (o de instancias de ellas) que atienden las solicitudes entrantes, con el fun de escalar de manera rentable y satisfacer así laos grandes volúmenes de tráfico entrante._


* ¿Cuál es el propósito del *Health Probe*? _Cuando se configura un nuevo balanceador de carga se crea un Health Probe el cual usará el balanceador para determinar si las instancias dentro del Backend Pool se encuentran en buen estado, si una de las instancias falla un determinado número de veces entonces esta dejará de dirigir tráfico hacia ella hasta que empiece a pasar las pruebas de estado nuevamente._


* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema? _Las load balancing rules se usan para definir cómo se distribuye el tráfico a las VM. Se define la configuración de IP de frontend para el tráfico entrante y el grupo de IP de backend para recibir el tráfico. El puerto de origen y destino se definen en la regla._


* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*? 


   _Virtual Network: Esta es una tecnología de red la cual permite extender la red de área local sobre una red pública o no controlada (como Internet), permite enviar y recibir datos sobre redes compartidas o públicas comportandose como una red privada aprovechando esta funcionalidad, seguridad y políticas de gestión de una red privada, y se implementan realizando conexiones dedicadas y/o cifrado._

   _Subnet: Es una segmentación de la red virtual (o cualquier red en general), permitiendo asignar una o varias subredes a la misma red, estas subredes cuentan con un rango de direcciones apropiadas para la organización._

   _Address space: Cuando se crea una red virtual, se debe especificar un rango de direcciones ip privadas personalizadas (RFC 1918). Azure asigna a los recursos de una red virtual una dirección IP privada desde el espacio de direcciones que asigne. Por ejemplo; si implementa una máquina virtual en una red virtual con espacio de direcciones, 10.0.0.0/16, a la máquina virtual se le asignará una dirección IP privada como 10.0.0.4._

   _Address range: Indica cuantas direcciones se tienen en un address space y dependiendo de la cantidad de recursos que se necesiten en la red virtual, el rango aumentará o disminuirá._


* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*? 


   _Zonas de disponibilidad: Son las ubicaciones geográficas únicas dentro de una región, donde cada zona se compone de uno o más centros de datos equipados con alimentación, refrigeración y redes independientes._

   _Seleccionamos 3 zonas de disponibilidad diferentes para poder tener una mejor disponibiliad y tolerancia a fallos dentro del sistema, haciendo así que en caso de que falle alguno de los centros de datos, el load balancer utilizará otro nodo de la red que se encontrará ubicado en otra ubicación geográfica, logrando de esta manera garantizar la resiliencia y disminuir la probabilidad de que el sistema se encuentre no disponible, sin verse afectado el principio de disponibilidad de la aplicación._

   _IP zone-redundant: Un gateway zone-redundant aporta resistencia, escalabilidad y disponibilidad al sistema, cuando se utiliza una ip zone-redundant azure esta separa física y lógicamente el gateway dentro de una región, lo cual permite mejorar la conectividad de la red privada y disminuir los fallos a nivel de zona de disponibilidad._


* Presente el Diagrama de Despliegue de la solución.

![alt text](https://raw.githubusercontent.com/angiedanielar/LAB8_ARSW/main/images/9.png)









