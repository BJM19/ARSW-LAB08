### Escuela Colombiana de Ingeniería

### Arquitecturas de Software - ARSW

### Integrantes:

#### Brayan Jiménez
#### Mateo Quintero

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
   > Resultado espacio normal
   * 1000000
   > ![](./images/part1/7-01-2.png)
   * 1010000
   > ![](./images/part1/7-02-2.png)
   * 1020000
   > ![](./images/part1/7-03-2.png)
   * 1030000
   > ![](./images/part1/7-04-2.png)
   * 1040000
   > ![](./images/part1/7-05-2.png)
   * 1050000
   > ![](./images/part1/7-06-2.png)
   * 1060000
   > ![](./images/part1/7-07-2.png)
   * 1070000
   > ![](./images/part1/7-08-2.png)
   * 1080000
   > ![](./images/part1/7-09-2.png)
   * 1090000
   > ![](./images/part1/7-10-2.png)
   > Resultado espacio agrandado
    * 1000000
   > ![](./images/part1/7-01.png)
    * 1010000
   > ![](./images/part1/7-02.png)
    * 1020000
   > ![](./images/part1/7-03.png)
    * 1030000
   > ![](./images/part1/7-04.png)
    * 1040000
   > ![](./images/part1/7-05.png)
    * 1050000
   > ![](./images/part1/7-06.png)
    * 1060000
   > ![](./images/part1/7-07.png)
    * 1070000
   > ![](./images/part1/7-08.png)
    * 1080000
   > ![](./images/part1/7-09.png)
    * 1090000
   > ![](./images/part1/7-10.png)

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

> Revisión espacio normal:
>
> ![](./images/part1/8-01-2.png)
> Revisión espacio agrandado:
>
> ![](./images/part1/8-01.png)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```
   > Resultados:
   > ![](./images/part1/9-01.png)
   > ![](./images/part1/9-02.png)

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

Respuesta:
> Respuesta:
> Consideramos que no se cumple el requerimiento funcional debido a que a pesar de haber hecho un escalamiento horizontal
> se siguen mostrando errores al momento de realizar la ejecución con los scripts de postman.
12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?

   > Se crean 4 recursos los cuales son:
   > * Direcciones IP públicas y privadas
   > * VNet(Virtual Network)
   > * NSG (Grupos de seguridad de red)
   
2. ¿Brevemente describa para qué sirve cada recurso?


> Azure Virtual Network (VNet): Es el bloque de construcción fundamental para su red privada en Azure. VNet permite
> que muchos tipos de recursos de Azure, como las máquinas virtuales (VM) de Azure, se comuniquen de forma segura entre sí,
> con Internet y con las redes locales.


> IP pública: Permiten que los recursos de Internet se comuniquen con los recursos de Azure.

> IP privada: Permite la comunicación entre recursos en Azure tales como:
> * Virtual machine network interfaces
> * Internal load balancers (ILBs)
> * Application gateways
> * Virtual network.
> * Red local a través de una pasarela VPN o un circuito ExpressRoute.

> NSG: Un grupo de seguridad de red contiene reglas de seguridad que permiten o deniegan el tráfico de red entrante hacia,
> o el tráfico de red saliente desde, varios tipos de recursos de Azure.

3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?


> Debido a que cuando ejecutamos por ssh todos los procesos quedan asociados a esta connexion, y al momento de cerrarla los procesos
> iniciados que no tengan una orden que dictamine que sigan en ejecución se cerraran.

> Porque con esta regla permitimos el acceso público a un puerto específico de la máquina virtual, para finalmente ejecutar
> nuestra aplicación por este puerto y que sea accesible por todos.

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

> ![](./images/part1/4-1.png)
>
> ![](./images/part1/4-2.png)
>
> La función tarda tanto tiempo y tiende a ser incremental debido a la cantidad de iteraciones que se realizan


5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.


> ![](./images/part1/8-01.png)
>
> ![](./images/part1/8-01-2.png)
>
> Consume tanta CPU debido a que se tiene una implementación no eficaz, la cual consume muchos recursos para realizar
> muchas iteraciones.
> 
6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    * Si hubo fallos documentelos y explique.
> Sin aumento de espacio:
>
> ![](./images/part1/9-01-2.png)
>
> ![](./images/part1/9-02-2.png)
>
> Se obtienen fallos de desconexión y los tiempos de ejecución varían entre 21s y 44s
> Con aumento de espacio:
>
> ![](./images/part1/9-01.png)
>
> ![](./images/part1/9-02.png)
>
> Se siguen obteniendo fallos de desconexión aunque un poco menores y los tiempos disminuyen y varían entre 14s y 30s
>
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

> Se diferencian en su capacidad en memoria, uno con 0.5Gb y otro con 8Gb respectivamente, además de su gran diferencia en la CPU, ya que
> B2ms es el doble de B1ls, esto ocasiona que el precio por mes del B2ms sea mucho mayor que el de B1ls.

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

> No es una buena solución debido a que las pruebas siguen fallando debido a la mala implementación que hay en FibonacciApp,
> el cambio que se genera es que hay un menor consumo de CPU debido a que esta es más grande,

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

> Cuando se realiza algún tipo de cambio 'físico' en la VM esta debe ser reiniciada por completo, por lo cual
> cualquier servicio que esta esté prestando se verá afectado debido a que no habrá disponibilidad del mismo

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

> En términos de porcentaje hubo mejora, ya que se redujo el consumo en casi un 40%, pero en tiempo no se aprecia, la diferencia es casi nula.

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

> No, debido a que se siguen presentando la misma cantidad de errores al momento de ejecutarlo.

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

![](./images/part2/parte2-1.png)

![](./images/part2/parte2-2.png)

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.
   
![](./images/part2/parte2-3.png)

> Como podemos observar en peticiones aceptadas hay una gran diferencia ya que con el escalamiento vertical todas las peticiones realizadas fueron aceptadas,  mientras que en escalamiento horizontal algunas de estas fallaron.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 

4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
```

![](./images/part2/parte2-4.png)
![](./images/part2/parte2-5.png)
![](./images/part2/parte2-6.png)
![](./images/part2/parte2-7.png)

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?

> Existen 2 tipos de balanceadores de carga en Azure, el balanceador público y el balanceador privado (interno), se
> diferencian en que el balanceador público está hecho para dar conexiones de salida para las máquinas virtuales, mientras que
> el interno se utilizan para realizar el equilibrio de la carga dentro de una red virtual.

* ¿Qué es SKU, qué tipos hay y en qué se diferencian?

> Representa una posibilidad para comprar Existencias (SKU) por debajo de un producto.

* ¿Por qué el balanceador de carga necesita una IP pública?

> Añadir un punto de conexión a los perfiles Traffic Manager

* ¿Cuál es el propósito del *Backend Pool*?

> Define el grupo de recursos que servirá el tráfico para una regla de equilibrio de carga determinada

* ¿Cuál es el propósito del *Health Probe*?

> Permitir que él, Load Balancer detecte el estado del extremo del backend

* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
> Definir el tráfico de red atreves de las máquinas virtuales, las sesiones que este permite son sesiones sin definir
> persistencia o con persistencia definida, esto significa que cuando se realice una misma petition esta será redirigida
> al cliente original y no a uno nuevo, esto puede afectar a aplicaciones que trabajen con sesiones guardadas en memoria,
> o en aplicaciones sticky.

* ¿Qué es unaVirtual Network ?

> Es el bloque de creación fundamental de una red privada en Azure

* ¿Qué es una *Subnet*?

> Es un rango de direcciones lógicas, que se utiliza normalmente cuando se tienen redes demasiado grandes, estos son divididas en redes más pequeñas

* ¿Para qué sirven los *address space* y *address range*?

> Los address space son aquellas direcciones de red asignables dentro de una Vnt y los address range son las redes asignables dentro de una subnet.

* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?

> Una zona de disponibilidad es una oferta de alta disponibilidad que protege sus aplicaciones y datos de los fallos
> del centro de datos. Las zonas de disponibilidad son ubicaciones físicas únicas dentro de una región de Azure.

* ¿Cuál es el propósito del *Network Security Group*?

> Utilizar un grupo de seguridad de red de Azure para filtrar el tráfico de red hacia y desde los recursos de Azure
> en una red virtual de Azure
* Informe de newman 1 (Punto 2)

![](./images/part2/parte2-3.png)

* Presente el Diagrama de Despliegue de la solución.

> ![](./images/part2/diagrama.png)



