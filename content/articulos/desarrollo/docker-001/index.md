--- 
author: agomezguru
date: 2023-04-21T10:01:43-06:00 
title: Despliegue moderno de aplicaciones web 
tags: 
  - Docker
  - Contenedores
  - SaaS
  - DevOps
description: Despliegue moderno de aplicaciones web - SaaS/Contenedores Docker
summary: Despliegue moderno de aplicaciones web - SaaS/Contenedores Docker,
  a mi manera
--- 

{{< bigcaps P >}}robablemente a estas alturas del partido ya te hayas percatado que hay 1001 maneras de matar una mosca, lo mismo podemos decir de las formas de desplegar un sitio/app web.
{.alphabet .mb-4}

Al igual que exterminar una mosca con un cañón puede ser efectivo, no es necesariamente eficiente en términos de empleo de recursos y en el mundo de las [micro, pequeñas y medianas empresas](https://www.certus.edu.pe/blog/que-significa-mipymes/), los recursos tales como tiempo, dinero y esfuerzo no son precisamente los más abundantes. Por ello han aparecido una miríada de aplicaciones web que son entregadas como SaaS (Software as a Service) ó CMS (Content Management Systems) administrados por terceros y los cuales son muy fáciles de utilizar.
{.mb-4}

Probablemente el 95% de los proyectos web empiezan así, hasta que requieren de alguna funcionalidad extra y tienen que cambiar el plan de servicios contratado, entonces los costos empiezan a subir como la leche hirviendo ó también pasa muy seguido: La gerencia; Mercadotecnia ó Recursos Humanos empiezan a pedir cosas y muchas de ellas se instalan en servidores, pero el _departamento de sistemas_ (aún muchas [MIPyME's](https://www.certus.edu.pe/blog/que-significa-mipymes/) les llaman así) no tienen el presupuesto, un sitio adecuado o incumplen un sin número de otros factores, como la seguridad, que hacen inviable transitar por esa ruta (ya se que hasta ahora has realizado una excelente labor virtualizando las cargas de trabajo, pero todo tiene un límite). Para este tipo de situaciones existe Docker, un estándar de industrial para programar (**Dev**) y desplegar (**Ops**) servicios web. Docker hace que todo el proceso sea rápido, elegante y conveniente.
{.mb-4}

Es por ello que empiezo esta serie de post por los contenedores Docker, ya que constituyen los cimientos de infraestructura con los que se construirá todo el edificio y no cualquier edificio, puede llegar a alturas inimaginables dónde la única limitante es tu creatividad.
{.mb-4}

Pero antes debo hacerte un advertencia, para ser proeficiente en la entrega de aplicaciones de Software como Servicio requieres tener los siguientes conocimientos:
{.mb-4}

{{< list >}}

- Ducho en el uso de la línea de comandos de Linux (dentro de los propios contenedores no existe más que esto, en el mejor de los casos)

- Conocimientos de la sintaxis básica de YAMEL (no te preocupes haremos una breve intro)

- Estar familiarizado con toda la pila de despliegue (LAMP ó MAMP)

- Bases sólidas en el uso de Docker y Docker Compose

{{< /list >}}

### ¿Qué es el despliegue moderno Apps/Servicios Web?
{.blog-desc-big .m-0 .mb-4}

Cuando hablo del _despliegue moderno_ de apps/servicios web, es inevitable voltear a ver a los equipos de élite engendrados por las [FAANG](https://blog.selfbank.es/que-es-faang/) y el cúmulo de herramientas, metodologías, marcos de trabajo y acrónimos con que nos han inundado en la última década.
{.mb-4}

Así para mí, en la serie de artículos que iré publicando paulatinamente al decir: _despliegue moderno_ de apps/servicios web me refiero a emular, en la medida de lo posible, las buenas prácticas ampliamente documentadas que han llevado al éxito tecnológico a estas empresas de élite (así los llamo el propio Google en su reporte anual [DORA](https://www.devops-research.com/research.html)) y que se resumen con el siguiente acrónimo: _CI/CD_
{.mb-4}

Echamos un vistazo rápido al modelo propuesto por [DORA](https://www.devops-research.com/research.html) para lograr ser un equipo _'Elite Performance'_
{.mb-4}

{{< middle-image >}}

Ciertamente el panorama luce intimidante y parecería está fuera del alcance de los ínfimos equipos de desarrollo de software que laboran en las [MIPyME's](https://www.certus.edu.pe/blog/que-significa-mipymes/), pero esto no deber en forma alguna así.
{.mb-4}

Esta serie de artículos está focalizado en ayudarte a lograr que tu equipo de desarrolladores llegue a ser un equipo de alto desempeño pués, entre más pequeño es, está más obligado a emplear eficientemente las metodologías, herramientas y marcos de trabajo disponibles, incidiendo así en el día a día en tu organización para que puede ser una organización tecnológica de alto desempeño, que logre una transformación digital exitosa.
{.mb-4}

### Docker. El primer eslabón
{.blog-desc-big .m-0 .mb-4}

Empaquetar todo lo requerido para el desarrollo, desde la conceptualización hasta la puesta en producción implica coordinar un cúmulo de metodologías, herramientas y maneras de hacer las cosas totalmente dispares. Del lado Front-End del desarrollo tenemos: http; CSS, Bootstrap, Sass, Less; JavaScript, TypeScript, Node.js, React, Vue, Svelte; PHP, Laravel, Symfony; Phyton, Djago; etc. del lado Back-End: servidores Apache, NGINX, Tomcat ¿alguien todavía lo usa?; bases de datos SQL server, MySQL, Oracle, PostgreSQL; background workers, Node.js, PhantomJS; servicios de indexación como Apache Solr, Eelasticsearch; caché de aplicaciones Memcached, Redis y un largo etc.
{.mb-4}

Definitivamente el panorama se vislumbra sumamente fragmentado, las herramientas empleadas para el desarrollo y para la puesta en producción son muy diferentes y es aquí donde empieza el dilema. Precisamente encapsular una aplicación, si se hace a la manera moderna, implica no hacer conceciones en ninguna parte del ciclo de vida del servicio pero ¿cómo lograrlo de manera eficiente?
{.mb-4}

Solo hay una respuesta correcta posible, empleando contenedores de Docker.
{.mb-4}

### GitOps. La joya de la corona
{.blog-desc-big .m-0 .mb-4}

El despliegue moderno de aplicaciones web no sería posible, ni se entendería, sin el sistema de control de versiones más utilizado actualmente en el mundo, [Git](https://git-scm.com/book/es/v2).
{.mb-4}

En [DevOps ese viejo conocido](https://agomezguru.github.io/articulos/desarrollo/devops-001/) hice referencia a todas las ventajas ofrecidas por este paradigma como parte indispensable de cualquier estrategia de transformación digital y al ser [Git](https://git-scm.com/book/es/v2) la herramienta de trabajo habitual ¿por qué no extender su uso? Así lo entendió [WeaveWorks](https://www.weave.works/), empresa de gestión empresarial de Kubernetes y desde entonces ha proliferado en toda la comunidad de DevOps dando lugar al surgimiento de GitOps.
{.mb-4}

Aun cuando GitOps y DevOps comparten principios y objetivos similares. DevOps concentra sus esfuerzos en el cambio cultural y en ofrecer a los equipos de desarrollo (**Dev**) y de operaciones (**Ops**) la posibilidad de trabajar juntos de manera colaborativa.
{.mb-4}

En contraparte GitOps proporciona las herramientas y un marco de trabajo para que aplique las prácticas de DevOps (como la colaboración, la CI/CD y el control de versiones) a la automatización de la infraestructura y la implementación de las aplicaciones, es decir, el despliegue moderno de aplicaciones.
{.mb-4}

Ya se lo que te estarás preguntando a estas alturas ¿Usar Git para gestionar mis servidores?, este tipo debe estar loco. Si yo lo que hago es conectarme a mis máquinas por RDP o empleando la consola de VMWare, las provisiono actualizando el sistema operativo, reviso que no existan conflictos con los controladores, ejecuto algunos de mis scripts ¿será a esto a lo que se refiere?
{.mb-4}

De hecho no, me estoy refiriendo al empleo de archivos de configuración declarativos más que a código ejecutable. Para ello primero debemos entender que, aunque normalmente pensamos en Git como una herramienta de gestión de versiones empleada en el desarrollo de software, en realidad le es indiferente el contenido. Git tratará cualquier conjunto de archivos de texto e imagágenes como su "codebase" y yo lo utilizo, por ejemplo, para administrar este blog.
{.mb-4}

### Infraestructura como código (IaC)
{.blog-desc-big .m-0 .mb-4}

La infraestructura como código (IaC) es una de las más grandes aportaciones de DevOps. Anteriormente, los administradores de sistemas preferían los scripts imperativos personalizados para configurar servidores al estilo de: *Si sucede esto, haz tal cosa, sino, haz tal otra* siguiendo una secuencia de pasos para lógicos lograr el estado deseado del servidor.
{.mb-4}

Hacerlo así suele estar propenso a errores y es fácil que deje de funcionar al cambiar la secuencia de eventos. En contraposición, el despliegue moderno de servidores ha tendido a alejarse de los patrones imperativos para acercarse a los patrones de software declarativos.
{.mb-4}

El software declarativo sigue una declaración de un *estado esperado* en lugar de una secuencia de comandos. A continuación se muestra una comparación de declaraciones de DevOps imperativas y declarativas.
{.mb-4}

Mientras que en la forma tradicional las declaraciones imperativas suelen verse así:
{.mb-4}

```bash {style=friendly}
  Instala un sistema operativo en este servidor
  Instala estos controladores
  Descarga el código de esta URL
  Mueve el código a este directorio
  Haz esto 3 veces para otros 3 servidores
```

La versión declarativa sería algo como:
{.mt-4 .mb-4}

```bash {style=friendly}
  3 servidores tienen software de este repositorio (URL) instalado en este directorio.
```

[Terraform](https://www.terraform.io/) es la solución de [IaC](#infraestructura-como-código-iac) que he elegido para mis despliegues y verás muchos trzozos de código y ejemplos de su empleo en este blog. Como sus archivos de configuración son simplemente texto, me permite guardarlos empleando [Git](https://git-scm.com/book/es/v2) en una ubicación centralizada, están documentados y son accesibles todo mi equipo de trabajo.
{.mt-4 .mb-4}

### Conclusión
{.blog-desc-big .m-0 .mb-4}

No es fácil intentar resumir dos décadas de innovaciones en el despliegue de aplicaciones y servicios web en un sólo artículo, pero espero ferviertemente haber despertado tu curiosidad para lograr transitar desde una manera artesanal de hacer tus despliegues, al empleo de metodologías ampliamente difundidas y probadas por los equipos de alto desempeño _'Elite Performance'_
{.mb-4}

### Referencias
{.blog-desc-big .m-0 .mb-4}

{{< list >}}

- [Acceptance is the first step to being an Elite Performing Team](https://cd.foundation/blog/2020/11/06/ed-corner-weekly-acceptance-is-the-first-step-to-being-an-elite-performing-team/)

- [¿Qué es GitOps?](https://www.redhat.com/es/topics/devops/what-is-gitops)

- [¿Qué es GitOps? Extendiendo devops a Kubernetes y más allá](https://cioperu.pe/articulo/30703/que-es-gitops-extendiendo-devops-a-kubernetes-y-mas-alla/?p=4)

- [Una introducción a GitOps](https://geekflare.com/es/gitops-introduction/)

- [¿GitOps es la próxima gran innovación en DevOps?](https://www.atlassian.com/es/git/tutorials/gitops)
{{< /list >}}

### Créditos
{.blog-desc-big .m-0 .mb-4}

{{< list gray >}}

- Figura de portada. Originalmente publicada en la Web

- Figura "Capabilities to drive improvement from ACCELERATE". [Tomada de este artículo.](https://cd.foundation/blog/2020/11/06/ed-corner-weekly-acceptance-is-the-first-step-to-being-an-elite-performing-team/)
{.mb-4}
{{< /list >}}
