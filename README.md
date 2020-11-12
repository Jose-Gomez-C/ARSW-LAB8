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
    * 1000000
    ![Imagen1](https://i.ibb.co/sj6VgTV/image.png)
    * 1010000
    ![Imagen2](https://i.ibb.co/MZXT8YG/1.png)
    * 1020000
    ![Imagen3](https://i.ibb.co/XFxBG9z/2.png)
    * 1030000
    ![Imagen4](https://i.ibb.co/grqRC1W/3.png)
    * 1040000
    ![Imagen5](https://i.ibb.co/f93TKrr/4.png)
    * 1050000
    ![Imagen6](https://i.ibb.co/zbyr2zP/5.png)
    * 1060000
    ![Imagen7](https://i.ibb.co/9nt7Kfj/6.png)
    * 1070000
    ![Imagen8](https://i.ibb.co/Bz1DFqd/7.png)
    * 1080000
    ![Imagen9](https://i.ibb.co/d4FJ6Jq/8.png)
    * 1090000
    ![Imagen10](https://i.ibb.co/f2qPCC4/9.png)

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

* ![Imágen 2](images/part1/part1-vm-cpu.png)
* ![CPU1](https://i.ibb.co/3Ff9gh7/CPU.png)
* ![cpu2](https://i.ibb.co/yYkxprz/CPU2.png)
9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

* ![prueba](https://i.ibb.co/LCXsmzp/Pruebas1.png)
10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)


11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
    * 1000000
    ![Imagen1](https://i.ibb.co/8bTwJsm/b0.png)
    * 1010000
    ![Imagen2](https://i.ibb.co/6N2JL1P/b1.png)
    * 1020000
    ![Imagen3](https://i.ibb.co/0BpbtQT/b2.png)
    * 1030000
    ![Imagen4](https://i.ibb.co/F03KD8H/b3.png)
    * 1040000
    ![Imagen5](https://i.ibb.co/5K6Pm0G/b4.png)
    * 1050000
    ![Imagen6](https://i.ibb.co/dmRxxXz/b5.png)
    * 1060000
    ![Imagen7](https://i.ibb.co/92y80NF/b6.png)
    * 1070000
    ![Imagen8](https://i.ibb.co/HBCcswP/b7.png)
    * 1080000
    ![Imagen9](https://i.ibb.co/99cFzML/b8.png)
    * 1090000
    ![Imagen10](https://i.ibb.co/WtqgzT4/b9.png)
    > **Consumo de CPU:**
    * ![cpu1](https://i.ibb.co/VQSPBgx/bCPU.png)
    * ![cpu2](https://i.ibb.co/WBMRx9v/bcpu2.png)
    > **Resultado de las pruebas:**
    * ![prueba2](https://i.ibb.co/JCfwmyh/pruebas2.png)
12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
  > **Al realizar escalamiento vertical no se logra el objetivo, ya que el porcentaje de consumo de la CPU, disminuye con el cambio de temaño pero el tiempo de respuesta sigue       muy similar a la prueba realizada anteriormente lo que ocasiona que no pasen todas las pruebas.**
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
> Hay 5 recursos que se crean con la maquina virtual y son: 
* Dirección IP pública
* Grupo de seguridad de red
* Interfaz de red
* Disco
* Clave SSH

2. ¿Brevemente describa para qué sirve cada recurso?
> * Direccion IP pública: Permiten a los recursos de Internet la comunicación entrante a los recursos de Azure. Permiten que los recursos de Azure se comuniquen con los servicios de Azure orientados al público e Internet. Hasta que cancele la asignación, la dirección estará dedicada al recurso.
> * Grupo de seguridad de red: se utiliza para filtrar el tráfico de red hacia y desde los recursos de Azure de una red virtual de Azure. Un grupo de seguridad de red contiene reglas de seguridad que permiten o deniegan el tráfico de red entrante o el tráfico de red saliente de varios tipos de recursos de Azure.



> * Interfaz de red: permite que una máquina virtual de Azure se comunique con los recursos de Internet, Azure y locales. En su lugar, puede crear interfaces de red con una configuración personalizada y agregar una o varias interfaces de red a una máquina virtual al crearla, como también puede cambiar la configuración predeterminada de la interfaz de red en una interfaz de red existente.

> * Disco: Se encara del almacenamiento de la maquina virtual y nos indica la capacidad de la RAM, Discos de datos, vCPU, entre otros.

> * Clave SSH: es la clave por la cual se tendrá acceso a la VM  como método de autenticación esto hace que sea mucho mas difícil acceder a la maquina con ataques de adivinación por fuerza bruta 


3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?
> * Al momento de realizar la conexión con la aplicación se crea un proceso que ejecuta la aplicación, por ende cuando la cerremos, finaliza el proceso de ejecutar la aplicación. La inbound port rule se realiza por el puerto 3000 para desplegar la aplicación de NodeJS de manera publica y asi poder ver la aplicación en internet 

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.
* ![Tabla de tiempos](https://i.ibb.co/th7t4Qk/tabla-de-tiempos.png)
> * Aunque vemos una mejora en la mayoría de los casos a la mitad del tiempo, no creemos que sea lo correcto para una aplicación que solo debe hacer operaciones matemáticas. Como evidenciamos la implementación que se hace de Fibonacci no es la correcta para aprovechar el aumento en el hardware. 
5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    * Si hubo fallos documentelos y explique.
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?
> * El tamaño B1ls tiene menos capacidad y es mucho más económica que el B2ms. el tamaño B1ls esta solo disponible para el sistema operativo Linux. Estos tamaños por lo general entan pensados para  ser utilizados en ambientes de prueba a bajos costos
* ![Tabla de comparacion](https://i.ibb.co/6tHgz83/BsVsBm.png)
8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?
> **No se observa un drástico cambio en los tiempos, aunque aumente las características de hardware la diferencia es mínima. En cambio, si miramos los costos si se encuentra una gran diferencia. Según lo anterior evidenciamos que nos es una buena solución para este escenario. Cuando se cambia el tamaño se pierde la disponibilidad de la maquina mientras se realizan cambios.**
9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?
> **Se reiniciar nuestra instancia y la aplicación deja de correr por lo cual debemos ingresar a la misma y volver a poner en ejecución la aplicación. Los efectos que causan esto es que no estará disponible hasta que se vuelva a iniciar la maquina y la aplicación.**
10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?
> **SI hubo mejora ya que en la B1ls se consumió un total del 62.7611% y en la b2ms un total de 32.8231% esto debido a que tenemos mas recursos para hacer los cálculos de la aplicación**
11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?
* ![Pruebas](https://i.ibb.co/F6JFRGX/pruebas3.png)
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
> **Pruebas:**
* ![Hello world](https://i.ibb.co/GsCMs6d/load-Balancer1.png)
* ![prueba2](https://i.ibb.co/KjnXvRW/load-Balancer2.png)
2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.
### Informe
> **Parte 1**
* ![prueba](https://i.ibb.co/LCXsmzp/Pruebas1.png)
> **Parte 2**
* ![prueba2](https://i.ibb.co/dJN3jgs/pruebas4-Segunda-Parte.png)
> **Cuadro Comparativo**
*  ![Cuadro Comparativo](https://i.ibb.co/Vt9Wn0h/tabal.png)
3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```
> **VM 1**
* ![prueba2](https://i.ibb.co/QcS718m/VM1.png)
> **VM 2**
* ![prueba2](https://i.ibb.co/ZNJ7dDf/VM2.png)
> **VM 3**
* ![prueba2](https://i.ibb.co/8PR9Xyp/VM3.png)
> **VM 4**
* ![prueba2](https://i.ibb.co/yFcLpMb/VM4.png)
**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
* ¿Cuál es el propósito del *Backend Pool*?
* ¿Cuál es el propósito del *Health Probe*?
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
* ¿Cuál es el propósito del *Network Security Group*?
* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución.




