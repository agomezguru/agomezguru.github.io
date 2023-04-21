--- 
author: agomezguru
date: 2023-04-14T16:12:31-06:00 
title: "Despliegue Blue/Green" 
tags: 
  - DevOps
  - Deployment
  - Terraform
description: Despliegue Blue/Green en un entorno de nube.
summary: Despliegue Blue/Green en Google Cloud empleando Terraform. 
--- 


{{< bigcaps E >}}s curioso como ciertas ideas florecen de manera simultánea en varias mentes, no si esto tenga relación con lo que se conoce como conciencia colectiva o sea más afín al bulo Efecto del centésimo mono, pero algo así me ocurrió con los despliegues blue/green.
{.alphabet .mb-4}

Si saberlo ideé una forma de lograr poner en producción un nuevo servidor con un [MTTR](https://blog.infraspeak.com/es/mttr/) o con un [tiempo de inactividad](#descripción-del-problema) de cero, ya que muy frecuentemente el código, algún elemento de la pila o el propio sistema operativo cambian, siempre resulta muy engorroso programar dichos cambios a horas del menor tráfico posible (usualmente en la madrugada o en días de asueto), en especial en los tiempos en que todo se realizaba de forma manual y no existián las técnicas de modernas de despliegue DevOps. En fin, lo que se me ocurrió hace ya 14 meses, en realidad fué inventado desde tiempo atrás y descrito por primera vez, hasta donde tengo conocimiento, en el libro *Continuous Delivery* de la autoría de Jez Humble y David Farley.
{.mb-4}

Una de las mayores ventajas de la nube es que los costos son flexibles, así apoyados con las herramientas de [IaC](https://learn.microsoft.com/es-es/devops/deliver/what-is-infrastructure-as-code) como [Terraform](https://www.terraform.io/) podemos hacer cosas inimaginables hace poco tiempo a un costo irrisorio. Este es el caso de los despliegues Blue/Green.
{.mb-4}

### Descripción del problema
{.blog-desc-big .m-0 .mb-4}

Para entender cabalmente el problema que estamos intentando resolver con el [caso de uso](https://historiadelaempresa.com/caso-de-uso) típico de los despliegues Blue/Green, voy a definir primero *tiempo de inactividad* como un período en el que un sistema no está disponible. Puede aplicarse a cualquier computadora o red, pero se emplea más comúnmente en referencia a servidores. En particular, la confiabilidad del servidor de aplicaciones web a menudo se mide en términos de tiempo de inactividad, donde lo ideal es poco o ningún tiempo de inactividad.
{.mb-4}

Algunas de las razones por las que un servidor puede experimentar tiempo de inactividad son:

{{< list gray>}}

- **Reinicio del servidor:** reiniciar un servidor puede requerir unos minutos de tiempo de inactividad porque el sistema debe apagarse, reiniciarse y luego reiniciar los procesos necesarios para responder a las solicitudes entrantes.
- **Reinicio del software:** reiniciar un proceso, como Apache en un servidor web, puede causar algunos segundos de tiempo de inactividad mientras se reinicia el proceso.
- **Desconexión de la red:** si un servidor se desconecta físicamente de una red, los sistemas de la red no podrán acceder a él.
- **Interrupción de la red:** si alguna parte de una red (incluida Internet) no funciona entre el servidor y el cliente, el cliente no podrá comunicarse con el servidor.
- **Sobrecarga de tráfico:** si un servidor recibe más tráfico del que puede manejar, no podrá responder a todas las solicitudes. Los usuarios pueden experimentar tiempo de inactividad hasta que el tráfico disminuya. Esto puede deberse a un pico en el tráfico o a un ataque DDoS (Denegación de Servicio).
- **Falla de hardware:** si falla un componente de hardware importante, como un HDD o una SSD, es posible que el servidor deje de funcionar.
- **Falla de software:** si un proceso en un servidor, como el servicio httpd (HTTP), deja de ejecutarse, hará que el servidor no responda a las solicitudes hasta que se reinicie el proceso.
- **Corte de energía:** si se corta la energía eléctrica y no hay energía de respaldo disponible (por ejemplo, un generador o UPS), los sistemas afectados estarán fuera de línea hasta que se restablezca la energía.
- **Ataque de piratas informáticos:** si un pirata informático obtiene el control de un servidor, puede impedir el acceso a los servicios necesarios y hacer que el servidor deje de responder.

{{< /list >}}

Si desde siempre la redundancia, como los sistemas de almacenamiento RAID y los generadores de energía de respaldo nos han ayudado a evitar el tiempo de inactividad debido a una falla del hardware, por que no extender el mismo concepto a todo el servidor, la respuesta: *Costos $$$*, hasta ahora.
{.mb-4}

Si nuestra labor como [ingenieros de confiabilidad del sitio](https://www.ibm.com/mx-es/topics/site-reliability-engineering) es mantener los SLA's acordados, ello implicará siempre, de manera intrínseca, minimizar el tiempo de inactividad tanto como sea posible. Aún cuando en promedio durante los últimos dos años los niveles alcanzados de [disponibilidad en mis servicios son 99.98%](https://elviajedelcliente.com/sla/), había ocasiones en que el tiempo de inactividad era inevitable. Por ejemplo, al realizar una migración del servicio a una nueva versión incompatible que requiere de hacer un respaldo a la base de datos, realizar la migración de esos datos a la nueva versión, ejecutar rápidamente un [smoking test](https://es.wikipedia.org/wiki/Pruebas_de_humo), etc. pueden ser necesarios varios minutos o incluso algunas horas de tiempo de inactividad. Este tipo de *tiempo de inactividad planificado* generalmente se programa para las primeras horas de la madrugada o en días no laborables cuando los niveles de tráfico son más bajos.
{.mb-4}

Otro [caso de uso](https://historiadelaempresa.com/caso-de-uso) bastante recurrente es ocasionado, paradójicamente por la propia solución, pues todos los [proveedores (providers)](https://registry.terraform.io/browse/providers) de Terraform están actualizando sus versiones a una velocidad alucinante. En un afán de sacarle media cabeza a su más cercano competidor y en lo pones en práctica lo descrito en este blog, seguramente ya salieron a la luz del sol nuevas características, mejoras, bugfixes, etc. y eso se refleja en las versiones de cada provider.
{.mb-4}

Pues bien, lo último que deseas hacer es _experimentar_ con el ambiente de producción y es aquí donde entra en escena por segunda ocasión los despliegues Blue/Green, por ejemplo: si hace unos tres meses que desplegaste la última versión de tu infraestructura, seguramente todo la pila que sustenta tu software ya sufrió modificaciones, empezando por el sistema operativo, los [artefactos](https://es.abcdef.wiki/wiki/Artifact_(software_development)), ya surgieron nuevas funcionalidades experimentales y un largo etcétera.
{.mb-4}

Por ejemplo, recién leyendo el [sig. artículo](https://www.techrepublic.com/article/nala-apt-package-manager/), se me hizo interesante probar lo expuesto por el autor (muchas veces les pagan comisión por escribirlos y, a la hora de la verdad...), para ello te mostraré los pasos generales a seguir para que logres hacer un despliegue Blue/Green de prueba y si todo funciona como se espera, podría ser tu siguiente gran mejora, que en realidad se compondrá de muchas pequeñas mejoras que sucedieron simultáneamente, pero primero vamos a conceptualizar lo que deseamos lograr.
{.mb-4}

### Despliegue Blue/Green como alternativa de solución
{.blog-desc-big .m-0 .mb-4}

Con todo lo hasta ahora expuesto podemos definir un despliegue Blue/Green, como la implementación de una estrategia de despliegue de servicios y/o aplicaciones Web en producción, que permite actualizarlas de forma segura sin tiempo de inactividad.
{.mb-4}

Este proceso implica la creación de dos instancias idénticas de producción detrás de un balanceador de carga (load balancer). En cualquier momento, una aplicación responde al tráfico de usuarios, mientras que la otra recibe actualizaciones constantes del servidor de integración continua (CI) del equipo de desarrollo.
{.mb-4}

Dependiendo de tu presupuesto puedes mantener la segunda instancia prendida permanentemente, en reposo o como unas cuantas líneas de infraestructura como código con un costo de cero. Este es casi siempre mi caso, ya que poner en operación una nueva instancia con todo lo requerido en la actualidad no me toma más de 5 minutos.
{.mb-4}

Cuando el equipo de trabajo requiere *desplegar* nuevas funciones, cambia la ruta en el balanceador de carga de la versión anterior de su aplicación a la nueva versión. En ese momento, el servidor CI envía actualizaciones a la aplicación que ya no recibe tráfico del enrutador.
{.mb-4}

{{< figure src="blue-green-deployment-model.gif" class="image-fluid mb-3" >}}

Cabe anotar que esta solución aplica solo a servicios desplegados en nube, virtualizados o en contenedores, en el sentido de que su infraestructura debe ser automatizable para que el enfoque tenga sentido.
{.mb-4}

A los despliegues Blue/Green también se les conoce como el Red/Black, ya que representan el mismo concepto. Si bien el primero es el término más común, el segundo parece haberlo popularizado Netflix en sus herramientas (como Spinnaker).
{.mb-4}

### Diseño conceptual de un despliegue Blue/Green
{.blog-desc-big .m-0 .mb-4}

{{< citation >}}
Advertencia: Lo primero que te recomiendo es nunca, nunca experimentar con software que ya tienes puesto en producción, para ello está la nube. En ella, con ayuda de las herramientas adecuadas como las descritas en este blog, podrás crear proyectos completos, con una o decenas de instancias en tiempo récord y un costos extremedamente bajos.
{{< /citation >}}

{{< list >}}

1. El servidor de producción Blue se encuentra activo
1. Los nuevos cambios de código se despliegan en el servidor de producción Green, al que no se puede acceder públicamente a través de la CDN
1. Aprobados los cambios y ya listos para lanzar una nueva versión, pasamos del entorno Blue al entorno Green
1. El ambiente Green está ahora activo y es quién recibe el tráfico externo. El servidor Blue puede quedar prendido por un tiempo razonable como plan de RollBack y después puede ser apagado o incluso destruido para no incurrir en costos adicionales.
1. Cuando el equipo de desarrollo o el de operaciones desee realizar un nuevo cambio o probar una funcionalidad nueva, se enciende o recrea el servidor de producción Blue, según sea el caso y se repite este flujo de trabajo, pero en orden inverso, con el servidor de producción Green activo.
{{< /list >}}

{{< inner-images >}}

#### Prerrequisitos
{.blog-desc-big .m-0 .mb-4}

{{< list >}}

- Tener una cuenta en la nube de tu preferencia
- Herramienta de Infraestructura como Código: [Terraform](https://www.terraform.io/)
- Servidor DNS público administrable como [Terraform Provider](https://registry.terraform.io/browse/providers)
- Dos instancias idénticas de cómputo en nube
- Un dominio web de tu propiedad
{{< /list >}}

Una de las grandes ventajas de [Terraform](https://www.terraform.io/) como herramienta [IaC](https://learn.microsoft.com/es-es/devops/deliver/what-is-infrastructure-as-code) es la modularización de tu infraestructura, yo tengo ya muy estandarizado el proceso con mis propios módulos para la creación de los propios proyectos, otro para el entorno de red y así sucesivamente para las instancias de cómputo, la CDN, etc.
{.mb-4}

Como sé que ya llevas algunos años puliendo tu infraestructura en nube, solo necesitamos alterar tres partes de la misma para lograr nuestro cometido: El módulo de red, el de la CDN y agregar unas cuantas variables de inicialización de tus proyectos.
{.mb-4}

#### El módulo de red
{.blog-desc-big .m-0 .mb-4}

Aquí lo que necesitamos es muy simple, solo dos IP's públicas accesibles desde la CDN. Para lograrlo de una manera simple, agregué una variable en el archivo de configuración del proyecto, el código se ve así:
{.mb-4}

```tf {style=friendly}
  # Set static IPs for the instances
  variable "required_ips" {
    type        = number
    description = "Number of required IPs"
    default     = 1

    validation {
      condition     = var.required_ips > 0 && var.required_ips < 4
      error_message = "Please provide a valid number of required public IPs. Options are: 1, 2 or 3."
    } 
  }
```

Y en main.tf el código queda así:
{.mt-4 .mb-4}

```tf {style=friendly}
  # Generate a random name for external IP address
  resource "random_string" "ext_address" {
    count   = 3
    length  = 4
    upper   = false
    numeric = true
    lower   = true
    special = false
  }

  # Create external IP for world accesss to server's 
  resource "google_compute_address" "external_ip_address" {
    count        = var.required_ips
    project      = var.project_id
    region       = var.region
    name         = "fixed-external-ip-${random_string.ext_address[count.index].result}"
    address_type = "EXTERNAL"
  }
```

Así siempre dispongo de una, dos o tres IP's dependiendo el número de instancias a ejecutar en un momento dado. Dos para los servidores de producción Green/Blue y una extra para el servidor de CI y Staging cuando el proyecto se encuentra en sus fases iniciales.
{.mt-4 .mb-4}

```tf {style=friendly}
  output "external_ip_0" {
    value = google_compute_address.external_ip_address[0].address
  }

  output "external_ip_1" {
    value = var.required_ips >= 2 ? google_compute_address.external_ip_address[1].address: "127.0.0.1"
  }

  output "external_ip_2" {
    value = var.required_ips == 3 ? google_compute_address.external_ip_address[2].address: "127.0.0.1"
  }
```

Las IP's públicas creadas por mi módulo las devuelvo a través del propio mecanismo de salida de Terraform mediante el archivo outputs.tf mostrado.
{.mt-4 .mb-4}

#### El módulo de CDN
{.blog-desc-big .m-0 .mb-4}

La mayoría de mis clientes me solicitan sitios web para la publicación de contenido accesibles por miles de lectores, ello en el pasado me condujo a seleccionar un proveedor de red de distribución de contenidos o CDN. Así, para hacerme la vida más fácil, siempre la pongo al frente de todos mis despliegues en Web.
{.mb-4}

En mi caso muy particular, la conmutación entre los servidores Green/Blue sucede en la CDN y no en el propio balanceador de carga, como lo encontrás detallado en la mayor parte de la literatura al respecto. Lo implementé así por que mi propio balanceador de carga (load balancer) está detrás de la CDN, de forma que el público siempre ve las IP's públicas de la CDN, nunca las de mi infraestructura. Una vez aclarado el punto, lo que yo hago es simplemente alterar los registros DNS para que apunten ya sea a la IP del servidor Blue o el Green, dependiendo de cuál se encuentre en operación en determinado momento.
{.mb-4}

Como tengo por costumbre probar exhaustivamente cualquier idea que se me ocurra, lo cual te recomiendo encarecidamente, hice una copia de todo mi módulo para no afectar los proyectos puestos actualmente en producción, ya que todos sin excepción hacen uso de él. A la nueva copia la nombré siguiendo la conveción estándar de [versionado semántico 2.0.0](https://semver.org/lang/es/).
{.mb-4}

Una vez clonado, lo primero hice fue alterar el código de la CDN para que apunte correctamente al servidor activo. Lo hice pasando una variable de tipo map a Terraform con todos los parámetros necesarios, al final mi código quedó así:
{.mb-4}

```tf {style=friendly}
  variable "prod_enabled" {
    type        = object(
      {
        .
        .
        .
        # Actual IP public address
        ip            = string
        # Backend servers blue/green configuration
        blue_svr      = bool
        blue_svr_ip   = string
        green_svr     = bool
        green_svr_ip  = string
        server_active = string
    })
    description = "Enable access to production environment with blue/green servers"
    default     =  {
      .
      .
      .
      ip            = ""
      blue_svr      = false
      blue_svr_ip   = ""
      green_svr     = false
      green_svr_ip  = ""
      server_active = ""
    }
  }
```

Donde **ip** me permite controlar la IP del servidor que está actualmente activo, tal como se muestra en el siguiente trozo de código:
{.mt-4 .mb-4}

```tf {style=friendly}
  resource "cloudflare_record" "backend_blue_access" {
    count   = var.prod_enabled.blue_svr == true ? 1: 0
    name    = var.cf_cdn.new_zone == true ? "backend-blue-access.${var.cf_cdn.zone_name}": "backend-blue-access.${var.prod_enabled.env_name}.${var.cf_cdn.zone_name}"
    value   = var.prod_enabled.server_active == "blue" ? var.cf_cdn.new_zone == true ? var.cf_cdn.zone_name: "${var.prod_enabled.env_name}.${var.cf_cdn.zone_name}" : var.prod_enabled.blue_svr_ip
    type    = var.prod_enabled.server_active == "blue" ? "CNAME" : "A"
    proxied = false # This value is always false to get access.
  }
```

El mismo código se repite idéntico para el servidor *Green* alterando los valores correspondientes:
{.mt-4 .mb-4}

```tf {style=friendly}
  resource "cloudflare_record" "backend_green_access" {
    count   = var.prod_enabled.green_svr == true ? 1: 0
    name    = var.cf_cdn.new_zone == true ? "backend-green-access.${var.cf_cdn.zone_name}": "backend-green-access.${var.prod_enabled.env_name}.${var.cf_cdn.zone_name}"
    value   = var.prod_enabled.server_active == "green" ? var.cf_cdn.new_zone == true ? var.cf_cdn.zone_name: "${var.prod_enabled.env_name}.${var.cf_cdn.zone_name}" : var.prod_enabled.green_svr_ip
    type    = var.prod_enabled.server_active == "green" ? "CNAME" : "A"
    proxied = false # This value is always false to get access.
  }
```

Si eres perspicaz habrás notado que en la línea 6 altero el tipo del registro DNS y lo cambio de CNAME a tipo A, esto también lo hago en el registro root del proyecto:
{.mt-4 .mb-4}

```tf {style=friendly}
  resource "cloudflare_record" "root" {
    count   = var.prod_enabled.on == true ? var.cf_cdn.new_zone == true ? 1: 0 : 0
    name    = "@"
    value   = var.prod_enabled.ip
    type    = "A"
    proxied = true
  }
```

Con ello garantizo que la CDN siempre apunte a la IP pública del servidor activo.
{.mt-4 .mb-4}

#### Las instancias de cómputo
{.blog-desc-big .m-0 .mb-4}

Ahora solo resta controlar qué instancia está prendida en un determinado momento, ello lo logro con estas tres variables en el archivo de configuración de mi proyecto (defaults.auto.tfvars):
{.mt-4 .mb-4}

```tf {style=friendly}
  blue_enabled  = true
  green_enabled = true
  server_active = "blue" # "green"
```

Ya dentro del archivo de configuración de las instancias las empleo de la siguiente manera:
{.mt-4 .mb-4}

```tf {style=friendly}
  module "blue_server" {
    count   = var.blue_enabled == true ? 1 : 0
    .
    .
    .
    ext_ip  = module.network.external_ip_0
    .
    .
    .
  }
```

Mismo código para el servidor *Green*, solo cambia la variable *var.green_enabled*. El valor de la IP pública que verá la CDN lo obtengo del [módulo de red](#el-módulo-de-red).
{.mt-4 .mb-4}

### Conclusión
{.blog-desc-big .m-0 .mb-4}

Despliegues simples, rollback's rápidos y la fácil recuperación ante desastres son los beneficios clave de la técnica de despliegue Blue/Green, si quieres ahondar aún más en los beneficios y algunos inconvenientes de esta técnica, consulta la siguiente sección.
{.mb-4}

### Referencias
{.blog-desc-big .m-0 .mb-4}

{{< list >}}

- [What is blue green deployment?](https://www.redhat.com/en/topics/devops/what-is-blue-green-deployment)

- [Blue-Green Deployment](https://www.atatus.com/glossary/blue-green-deployment/)

- [Introducción a los despliegues Azul-Verde](https://medium.com/devopslatam/introducción-a-los-despliegues-azul-verde-bb4055811279)
{{< /list >}}

### Créditos
{.blog-desc-big .m-0 .mb-4}

{{< list gray >}}
- Figura de portada. Originalmente publicada en [Blue-Green Deployment](https://www.atatus.com/glossary/blue-green-deployment/)
{.mb-4}

- Animación. Originalmente publicada en [www.redhat.com](https://www.redhat.com/en/topics/devops/what-is-blue-green-deployment)
{.mb-4}

- Figura 1 y 2. Originalmente publicada en [Introducción a los despliegues Azul-Verde](https://medium.com/devopslatam/introducción-a-los-despliegues-azul-verde-bb4055811279)
{{< /list >}}
