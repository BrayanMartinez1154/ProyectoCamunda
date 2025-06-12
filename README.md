Proyecto: Automatización de Solicitud de Certificados con Camunda y Microservicios
Este proyecto demuestra la implementación de un proceso de negocio para la solicitud de certificados universitarios, utilizando una arquitectura de microservicios orquestada por el motor de procesos Camunda BPM.

1. Descripción General
El sistema automatiza el flujo de trabajo desde que un estudiante solicita un certificado hasta que se emite y se le envía, pasando por diversas validaciones y tareas humanas asignadas a diferentes roles administrativos.

La arquitectura está diseñada para ser desacoplada y escalable, separando las interfaces de usuario, la lógica de negocio y el motor de procesos en servicios independientes que se comunican a través de API REST y patrones de workers externos.

2. Arquitectura de Microservicios
El proyecto se compone de cuatro microservicios principales, gestionados y conectados a través de Docker Compose:

Motor de Procesos (motor-procesos-universidad)

Tecnología: Spring Boot + Camunda BPM Engine.
Puerto: 9000.
Responsabilidad: Es el cerebro de la operación. Ejecuta la lógica del proceso BPMN, gestiona el estado de las solicitudes, ejecuta tareas automáticas (Java Delegates) y asigna tareas a usuarios o grupos. Expone las Webapps de Camunda (Cockpit, Tasklist).
Portal del Estudiante (portal-estudiante)

Tecnología: Spring Boot + Thymeleaf + Spring Security.
Puerto: 9001.
Responsabilidad: Interfaz web para los estudiantes. Permite a los usuarios autenticarse, iniciar nuevas solicitudes de certificados y completar las tareas que se les asignan durante el proceso (ej. realizar un pago).
Portal Administrativo (portal-administrativo)

Tecnología: Spring Boot + Thymeleaf + Spring Security.
Puerto: 9003.
Responsabilidad: Interfaz web para el personal de la universidad. Los usuarios administrativos se autentican y gestionan las tareas asignadas a su rol (ej. SECRETARIA, PROFESIONAL_ENCARGADO, TESORERIA) para revisar y aprobar las solicitudes.
Servicio de Tesorería y Certificados (servicio-tesoreria-certificados)

Tecnología: Spring Boot (Worker Externo de Camunda).
Puerto: 9004.
Responsabilidad: Funciona como un "worker" desacoplado. Se suscribe a tópicos específicos del motor de procesos (como emision-certificado) para ejecutar lógica de negocio intensiva o que requiere dependencias externas, como la generación de archivos PDF y el envío de correos electrónicos.
3. Características Principales
Orquestación de Procesos: Utiliza BPMN 2.0 para modelar y ejecutar el flujo de negocio de forma visual y mantenible.
Patrón de Worker Externo: Desacopla la lógica de negocio pesada (generación de PDF, emails) del motor de procesos, mejorando la resiliencia y escalabilidad.
Interfaces de Usuario por Rol: Portales web dedicados para estudiantes y personal administrativo, con vistas y acciones específicas para cada uno.
Seguridad Integrada: Autenticación y autorización por cada portal utilizando Spring Security.
Containerización: Todos los servicios están containerizados con Docker y orquestados con Docker Compose para un despliegue y desarrollo sencillos.
Comunicación Basada en API REST: Los portales interactúan con el motor de Camunda a través de su API REST.
4. Pila Tecnológica
Lenguaje: Java 17
Framework: Spring Boot 3.3.3
Motor de Procesos: Camunda BPM 7.22.0
Base de Datos: H2 (para desarrollo en memoria y en archivo)
Frontend: Thymeleaf
Build: Apache Maven
Contenedores: Docker & Docker Compose
Librerías Clave:
Spring Web, Spring Data JPA, Spring Security
OpenPDF (para generar los certificados en PDF)
Java Mail Sender (para el envío de correos)
Camunda External Task Client
5. Requisitos Previos
Para ejecutar este proyecto, necesitas tener instalado lo siguiente:

Java Development Kit (JDK) 17 o superior.
Apache Maven 3.8 o superior.
Docker y Docker Compose.
6. Cómo Ejecutar el Proyecto
El proyecto está configurado para levantarse fácilmente con un solo comando gracias a Docker Compose.

Clona o Descomprime el Proyecto
Asegúrate de tener la carpeta camunda_def_project con todos los microservicios dentro.

Construye y Levanta los Contenedores
Abre una terminal en la carpeta raíz del proyecto (camunda_def_project) y ejecuta el siguiente comando:

Bash

docker-compose up --build
Este comando hará lo siguiente:

Descargará las imágenes base necesarias.
Construirá la imagen de Docker para cada uno de los cuatro microservicios. Durante la construcción, Maven descargará las dependencias y compilará el código de cada servicio.
Creará una red interna para que los contenedores se comuniquen entre sí.
Iniciará los cuatro contenedores. depends_on asegura que el motor de procesos se inicie antes que los portales que dependen de él.
Verifica que todo esté funcionando
Una vez que los logs en la terminal se estabilicen, los servicios estarán disponibles en las siguientes URLs:

Motor de Camunda (Webapps): http://localhost:9000
Portal del Estudiante: http://localhost:9001
Portal Administrativo: http://localhost:9003
7. Cómo Usar la Aplicación
Credenciales de Acceso
Camunda Cockpit:

Usuario: admin
Contraseña: admin
Portal del Estudiante (definidos en portal-estudiante/src/main/resources/data.sql):

Usuario: EST001, Contraseña: est123
Usuario: EST002, Contraseña: est123
Portal Administrativo (definidos en portal-administrativo/src/main/resources/data.sql):

Usuario: secretaria1, Contraseña: secXYZ (Rol: SECRETARIA)
Usuario: profesor1, Contraseña: prof# instituto (Rol: PROFESIONAL_ENCARGADO)
Usuario: tesorero1, Contraseña: tes789 (Rol: TESORERIA)
Flujo de Ejemplo
Estudiante Inicia Solicitud:

Ve a http://localhost:9001.
Inicia sesión como EST001 con la contraseña est123.
Haz clic en "Solicitar Nuevo Certificado".
Selecciona un tipo de certificado y envía el formulario.
Secretaría Revisa la Solicitud:

Ve a http://localhost:9003.
Inicia sesión como secretaria1 con la contraseña secXYZ.
Ve a "Ver Tareas de Secretaría". Deberías ver la tarea de revisión.
Haz clic en la tarea, marca la casilla de aprobación y complétala.
Profesional Encargado Valida Requisitos:

Cierra sesión y vuelve a entrar en http://localhost:9003 como profesor1 (contraseña prof# instituto).
Ve a "Ver Tareas de Profesional Encargado", selecciona la tarea y aprueba los requisitos.
Tesorería Valida Estado Financiero:

Cierra sesión y vuelve a entrar en http://localhost:9003 como tesorero1 (contraseña tes789).
Ve a "Ver Tareas de Tesorería", selecciona la tarea y confirma que no hay deudas.
Estudiante Realiza el Pago:

Vuelve al portal del estudiante en http://localhost:9001 (puede que necesites iniciar sesión de nuevo como EST001).
Ve a "Ver Mis Tareas Pendientes". Aparecerá la tarea de pago.
Completa el formulario de pago simulado.
Recepción del Certificado:

Una vez completada la tarea de pago, el worker externo (servicio-tesoreria-certificados) será invocado.
Revisa la consola de logs de Docker. Verás los mensajes del CertificadoWorker indicando la generación del PDF y el envío del correo. Como el servicio de correo está configurado con credenciales de prueba, el correo se enviará realmente.
Monitoreo en Camunda Cockpit:

En cualquier momento, puedes ir a http://localhost:9000/camunda/app/cockpit/ e iniciar sesión como admin/admin para ver el estado visual del proceso, las variables y el historial de ejecución.
