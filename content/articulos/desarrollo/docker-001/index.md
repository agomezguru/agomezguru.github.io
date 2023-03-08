--- 
author: agomezguru
date: 2023-03-02T14:27:43-06:00 
title: Despliegue moderno de aplicaciones web 
tags: 
  - Docker
  - Containers
  - SaaS
description: Despliegue moderno de aplicaciones web - SaaS/Contenedores Docker
summary: Despliegue moderno de aplicaciones web - SaaS/Contenedores Docker,
  a mi manera
categories: 
  - general
draft: true 
--- 

{{< bigcaps P >}}robablemente a estas alturas del partido ya te hayas percatado que hay 1001 maneras de matar una mosca, lo mismo podemos decir de las formas de crear un sitio web.
{.alphabet .mb-4}

Al igual que exterminar una mosca con un cañón puede ser efectivo, no es necesariamente eficiente en términos de empleo de recursos y en el mundo de las micro, pequeñas y medianas empresas ([MIPyME](https://www.certus.edu.pe/blog/que-significa-mipymes/)), los recursos tales como tiempo, dinero y esfuerzo no son precisamente los más abundantes. Por ello han aparecido una miríada de aplicaciones web que son entregadas como SaaS (Software as a Service) ó CMS (Content Management Systems) administrados por terceros y los cuales son muy fáciles de utilizar.
{.mb-4}

Probablemente el 95% de las empresas [MIPyME's](https://www.certus.edu.pe/blog/que-significa-mipymes/) empiezan así, hasta que requieren de alguna funcionalidad extra y tienen que cambiar el plan de servicios contratado, entonces los costos empiezan a subir como la leche hirviendo ó también pasa muy seguido, la gerencia, marketing ó recursos humanos empiezan a pedir cosas y muchas de ellas se instalan en servidores, pero el _departamento de sistemas_ (aún muchas [MIPyME's](https://www.certus.edu.pe/blog/que-significa-mipymes/) les llaman así) no tienen el presupuesto, un sitio adecuado o un sin número de otros factores, como la seguridad, que hacen inviable transitar por esa ruta (muchas micro, pequeñas y medianas empresas han realizado una excelente labor virtualizando las cargas de trabajo, pero todo tiene un límite). Para este tipo de situaciones existe Docker, un estándar de industrial para implementar aplicaciones web. Docker hace que todo el proceso sea rápido, elegante y conveniente.
{.mb-4}

Es por ello que empiezo esta serie de post por los contenedores Docker ya que constituyen los cimientos de infraestructura con los que se construirá todo el edificio y no cualquier edificio, puede llegar a alturas inimaginables dónde la única limitante es tu creatividad.
{.mb-4}

Pero antes debo hacerte un advertencia, para ser proeficiente en la entrega de aplicaciones de Software como Servicio requieres tener los siguientes conocimientos:
{.mb-4}

{{< list >}}
1. Ducho en el uso de la línea de comandos de Linux
1. Conocimientos de la sintaxis básica de YAMEL (no te preocupes haremos una breve intro)
1. Estar familiarizado con toda la pila de despliegue (LAMP ó MAMP)
{{< /list >}}


### ¿Qué son los contenedores de Docker?
{.blog-desc-big .m-0 .mb-4} 
