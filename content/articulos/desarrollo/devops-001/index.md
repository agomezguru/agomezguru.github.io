---
author: agomezguru
title: DevOps ese viejo conocido
date: 2023-01-23T14:53:45-06:00
tags: [DevOps]
description: Entendiendo DevOps y su relación con las metodologías ágiles, el cómputo en nube y los contenedores Docker.
summary: Entendiendo DevOps y su relación con las metodologías ágiles, el cómputo en nube y los contenedores Docker
---

{{< bigcaps P >}}ara la mayoría de los desarrolladores en latinoamérica o al menos en el sur de mi país, México, el concepto de DevOps no es algo ajeno a nosotros. Desde siempre las actividades de desarrollo y operaciones se han hecho por la misma o mismas personas y no por desconocimiento, sino por falta de recursos.
{.alphabet .mb-4}

Me explico, usualmente la función tradicional de los ingenieros de sistemas ó administradores de sistemas (aún así se les conoce) entre las micro, pequeñas y medianas empresas ([MIPyME](https://www.certus.edu.pe/blog/que-significa-mipymes/)) se circunscribe, en un 80%-95% del tiempo laborable, a realizar actividades de soporte técnico ([mesa de ayuda](https://www.zendesk.com.mx/blog/que-es-mesa-de-ayuda/)) y solo el pequeñísimo porcentaje restante a realizar actividades de aprovisionamiento de infraestructura y/o desarrollo. Siendo que muchas de estas actividades: Interconexiones de red con cableado estructurado; Segmentación de redes con hubs, switches y routers; Gestión de access point y accesos VPN; Programación de DNS; Compra, provisión y despliegue de servidores, etc. se tercerizan con proveedores externos e integradores de servicios, la mayoría de las veces. Lo mismo aplica a las labores de despliegue e interconexión de aplicaciones, ello sin hablar del desarrollo de aplicaciones, el cual muchas veces se realiza con herramientas que, ahora se les conoce como [low code/no code](https://www.genbeta.com/desarrollo/que-programacion-low-code-no-code-que-se-diferencian-como-estan-democratizando-creacion-aplicaciones) y en mis tiempos se reducían a automatizar tareas mediante el empleo de scripts de línea de comandos (un método que aún sigo utilizando hoy día de manera cotidiana, sobre todo en el mundo de los _contenedores de Docker_), grabación y programación de macros y en los casos más sofisticados, programación en el lenguaje [BASIC](https://es.wikipedia.org/wiki/BASIC) o alguna de sus variante visuales y el empleo de bases de datos, así como la escritura manual de páginas web empleando HTML y CSS a finales de los años 90.
{.mb-4}

### Nuevas metodologías, mismo paradigma
{.blog-desc-big .m-0 .mb-4}

En resúmen ¿cuál equipo de desarrollo? ¿dónde está el equipo de operaciones? ¿y los equipos de soporte de mesa de ayuda de primer, segundo y tercer nivel? como ves, mi punto es el siguiente: Desde siempre hemos realizado DevOps en latinoamérica, lo que es realmente novedoso es la manera de ejecutarlo y ese es el motivo de escribir este post. Cómo es muy común en nuestra industria, los cambios raramente vienen solos y es bastante frecuente que coincidan dos o más grandes tendencias.
{.mb-4}

Ese fue el caso de las metodologías ágiles, el surgimiento de la nube como natural desarrollo del concepto Software como Servicio y sus posteriores variantes y a una distancia muy corta, el amplio desarrollo y posterior generalización de los contenedores Docker. Solo faltaba un ingrediente para que todo eclosionara en la majestuosa, floreciente  y multitudinaria cantidad de aplicaciones y servicios Web que vemos hoy día.
{.mb-4}

El elemento faltante que permitió unir todo se llama DevOps (ó DevSecOps dependiendo del grado de madurez de la [MIPyME](https://www.certus.edu.pe/blog/que-significa-mipymes/)) y es el tema que ahora nos ocupa.
{.mb-4}

Todos hemos escuchado la historia del desarrollador que mencionó que todo funcionaba en su equipo, pero a la hora de desplegar en producción su obra, todo se derrumbó... (es una canción muy conocida entre la gente de mi edad en México) pero como les comenté al inicio, para los desarrolladores en latinoamérica, ese no es el caso, ya que el mismo desarrollador es el responsable de la entrega en producción. Me recuerda mucho a mis labores de emprendedor, por allá en la década de los noventa. No sólo tenía que desarrollar las soluciones de TI, sino hacer presentaciones de producto, cerrar las ventas que iniciaban los vendedores, hacer la visitas de rigor para dejar funcionando la solución y dar soporte de primer, segundo y tercer nivel a los usuarios (por si alguién tiene dudas de en qué consiste esa cosa llamada emprendedurismo).
{.mb-4}

Con una economía tan castigada como la nuestra y donde aun se ve al elemento tecnológico como una área de soporte más que como un área estratégica, no existe esa división entre la actividades de desarrollo y operaciones tan marcada como lo es en la cultura anglosajona.
{.mb-4}

{{< inner-images >}}

### El demonio está en los detalles
{.blog-desc-big .m-0 .mb-4}

Pues bien, de eso tan conocido para nosotros se trata todo este asunto de DevOps, ¿cómo la ves? nada nuevo ¿verdad? Entonces, por qué tanta alharaca.
{.mb-4}

Como dice el dicho: El demonio está en los detalles y precisamente es lo que mejoró exponencialmente DevOps. Número uno, la velocidad de las entregas, número dos, la calidad de esas entregas, número tres, la operación diaria con el mínimo esfuerzo.
{.mb-4}

La velocidad de los despliegues se mejoró al adoptar la filosofía de las metodologías ágiles. La calidad, al gestionar las entregas llevando un registro histórico exhaustivo, empleando un flujo de trabajo eficiente mediante técnicas ampliamente difundidas de ramificación y versionado semántico, todo facultado por _git_ como gestor de versiones y por último pero no menos importante, una operación diaria garantizada alcanzando acuerdos en los  niveles de servicio (SLA's) antes insospechados (estoy hablando de una disponibilidad del 99.98%), todo ello realizando los despliegues en nube pública, empleando contenedores de Docker y redes de distribución de contenido ó CDN’s.
{.mb-4}

Cada uno de estos temas los iré desarrollando en futuras entregas, así que, nos leemos pronto.
{.mb-4}
