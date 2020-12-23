# Banco del futuro

A continuación se explica el diagrama Archimate con el planteamiento de la solución a las problemáticas de Banco del Futuro. El diagrama se puede encontrar en: [Banco del futuro](Hard_Skills.pdf).

## Motivación
En la primera capa se encuentra explicada la Motivación del proyecto. Se seleccionaron 4 motivadores de la lista planteada en el enunciado. 
-	Tiempo de construcción de nuevos servicios con entidades externas (Time to Market)
-	Alta demanda en integración con Sistemas externos
-	Aumentar el nivel de servicios con respecto a exigencias del mercado
-	Inconformidad de socios estratégicos del banco.

## Estratégia
Estos cuatro motivadores de negocio impulsan estrategias que permiten alcanzar los objetivos planteados por los stakeholders (Segunda capa). Para poder cumplir con los objetivos de negocio, se plantea:

-	Mejorar los procesos de selección de proveedores
-	Mejorar la comunicación entre equipos de riesgo y operaciones
-	Flexibilidad en los sistemas de información para modificar reglas de negocio
-	Unificar información dispersada en diferentes sistemas de información
-	Adquisición de Banco Promisorio

## Negocio
En la tercera capa se modelan algunos procesos que se describen en el enunciado. Se resaltan los más relevantes en la operación normal del banco.

## Aplicación
En la capa de aplicación se plantean los componentes de Software que comprenden el sistema y se dividen en dos grandes grupos, frontend y backend. En la solución planteada, el estilo de arquitectura que dirige el diseño es una combinación de Arquitectura orientada a microservicios y EDA (Event Driven Architecture).

### Frontend
En la parte izquierda se pueden encontrar los componentes relacionados con el Frontend del banco. Canales transaccionales expuestos a clientes finales y algunos operados por asesores de servicio o comercios formales. Para la comunicación entre los canales transaccionales y el backend, se plantea utilizar un patrón de microservicios llamado Backend for Frontends (BFF).

El portal transaccional sería desarrollado en frameworks y lenguajes especializados para front como Angular, React, Django.

Para el portal transaccional móvil se plantea desarrollar una aplicación en frameworks para aplicaciones híbridas, teniendo en cuenta que se favorece la mantenibilidad del sistema aunque se puede sacrificar la flexibilidad y personalización en cada uno de los sistemas operativos móviles.

 El API gateway transaccional (el que expone información al portal transaccional web y al móvil), expondría un API GraphQL utilizando [Apollo](https://www.apollographql.com/docs/federation/api/apollo-gateway/). Este API permite garantizar que la información que se solicita al backend es estrictamente la que se necesita mostrar en cada frontend y optimiza el uso de la red transportando datos, sin importar que la informacións ea extraída de backends que exponen APIs tipo REST o SOAP o incluso suscribirse a una cola de mensajería.

En el API Gateway de los canales físicos (Oficinas y corresponsales bancarios) se plantea exponer un API tipo REST que concentraría la operación para estos dos canales transaccionales.

Para los canales de interacción automática como los ATMs y la mensajería SMS, se plantea utilizar un API Gaetway especializado, incluso uno para cada canal, por ejemplo en el canal ATM se necesita un componente especializado que pueda interpretar el protocolo ISO 8583.

### Backend
En la zona derecha de la capa de Aplicación se encuentran los componentes relacionados con el Backend y que contemplan una arquitectura orientada a eventos. En el centro se puede observar un Broker de mensajería que se encarga de integrar los diferentes componentes de los que se transporta y procesa información. Se plantea que la comunicación con el Broker de mensajería sea asyncrona y orientada a eventos para que cada componente interesado en un tipo de mensaje pueda suscribirse al tópico, procesar la información y emitir eventos para los demás componetnes interesados. Teniendo en cuenta la complejidad de un sistema bancario, se recomienda utilizar un patrón de Orquestración de servicios. Adicionalmente en cada uno de los componentes se sugiere implementar mecanismos de detección, correción y enmascaramiento de fallas con el fin de favorecer la disponibilidad del sistema, sobretodo en horas pico. 

El sistema debe favorecer además la escalabilidad, teniendo en cuenta que próximamente la operación del banco aumentará considerablemente con la adquisición del Banco Promisorio.

Para solucionar el problema de la dispersión de información en diferentes sistemas, se debe ejecutar un plan de unificación de datos posiblemente delegando responsabilidades de gestión de clientes exclusivamente al CRM y responsabilidades de autenticación a un componente especializado, peude ser de terceros o hecho in House, se peude evaluar en una siguiente iteración. 

Para apalancar la creciente operación del banco en el momento de la fusión con banco promisorio, se debe plantear un compoente Intérprete que se encargue de la interoperabilidad del sistema legacy de banco promisorio y el sistema principal de banco del futuro. 

Podemos observar en el diagrama un componente de gestión documental, el cuál se conecta con una base de datos No SQL donde se almacenan digitalizados os documentos legales que exige la super intendencia bancaria.

Adicionalmente podemos encontrar un componente de Alertas y notificaciones que se encarga exclusivamente de enviar comunicaciones por diferentes canales de comunicación (email, SMS, notiicaciones Push, etc) a los clientes y operadores del banco.

Como en toda entidad bancaria, no puede faltar el componente que se encarga de analizar y prevenir el fraude transaccional. Para este caso se recomienda adquirir en el mercado soluciones de fabricantes expertos en el monitoreo transaccional, autenticación de doble factor y navegación segura en canales transaccionales, para poder minimizar el riesgo asociado al robo de dinero. Este proceso de protección y prevención se puede fortalecer con soluciones desarrolladas in house que implementen algoritmos de perfilamiento y machine learning con el fin de arrojar score de riesgo por transacción.

## Tecnología
Se plantea que el sistema sea desplegado en AWS para ahorrar costos en infraestructura y mantenimiento de harwdare. No sin antes resaltar que se debe tener especial cuidado en la selección de las instancias EC2 que se pretendan utilizar. Par ael despliegue del frontend se sugiere utilizar contenedores docker orquestrados con un Kubernetes o con ala suite de Hashicorp (Nomad & consul), depende del presupuesto del proyecto.

Los API Gateways de AWS se pueden utilizar en el caso de exponer APIs tipo rest. Y se recomienda hacer uso de los servicios RDS para la gestión de bases de datos, teniendo en cuenta que el servicio de AWS cuenta con soporte para los motores relacionales y no relacionales más usados en e mercado.

En cada caso la selección de la tecnología depende de las licencias con las que cuente actualmente Banco del futuro, en ausencia de licencias se recomienda el uso de software libre para el desarrollo de cada uno de los componentes. 

Teniendo en cuenta la confiabilidad del broker de mensajería de AWS, se puede utilizar en este proyecto.

Para minimizar los riesgos de seguridad de la plataforma se recomienda utilizar VPC provadas para la comunicación entre los diferentes componentes del sistema y exponer la información públicamente únicamente a través de los canales transaccionales.

## Generalidades
Para el desarrollo del proyecto se recomienda establecer grupos de trabajo de no más de 10 personas (idealmente 5), donde cada grupo se dedique al desarrollo de cada uno de os componentes. Al ser una arquitectura de alto nivel es necesario entrar en detalle de los componentes críticos donde posiblemente se tenga que asignar más equipos por la responsabiliad tecnología de cada pieza de software. cada grupo estaría formado por mínimo un lider técnico, 2 desarroladores y 2 automatizadores de pruebas. 

El desarrollo debe priorizarse por los componentes más críticos y que soportan la operación del banco. 

Para satisfacer las necesidades de banco del futuro, es necesario tener más información detallada del funcionamiento actual de cada uno de los componentes del sistema, esta solución es planteada con el mejor entenimiento del enunciado entregado por el equipo consultor.
