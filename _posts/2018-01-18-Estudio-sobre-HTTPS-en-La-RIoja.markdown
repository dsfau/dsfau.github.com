---
layout: post
title:  "HTTPS research in La Rioja"
date:   2018-01-18 11:18:29 +0200
categories: blog-ES
---
# HTTPS research in La Rioja

Hace ya algunos meses (Junio del 2017) publiqué los resultados de un pequeño estudio realizado a las empresas de La Rioja para determinar el grado de implementación de HTTPS en las empresas de la comunidad.
Los resultados no fueron muy buenos puesto que el grado de implementación para nada fue lo esperado.
Para realizar el pequeño estudio se cogió como muestra las 40 empresas de La Rioja con un nivel de facturación mas alto. Este dato lo obtuve de un ranking que publica el diario "el economista. "(http://ranking-empresas.eleconomista.es/empresas-RIOJA.html)"
En caso de que queráis obtener vosotros los datos os dejo unos simples comandos para extraer las paginas web de las empresas.

```bash
for i in $(curl http://ranking-empresas.eleconomista.es/empresas-RIOJA.html|\
grep '<td class="col_responsive1"'|sed 's/"col_responsive1"/\n/g'|\
grep "^>class"|sed 's/href=/\n/g'|sed "s/>Ver/\n/g"|grep '^"h'|sed 's/"//g');\
do 
  curl $i |grep '<td class="tal website">'|sed 's/tal website/\n/g'|sed 's/rel="nofollow"/\n/g'|\
  grep '^"><a'|sed 's/href=/\n/g'|grep '^"h' >> webs ;\
 done
```
De este modo obtendremos un fichero llamado "webs" con las urls de las empresas del ranking, ya solo queda acudir a Qualys SSl labs y analizar las webs, este proceso lo podéis automatizar con la ayuda de la api de SSL Labs que devuelve los resultados en JSON y son muy fáciles de analizar.

## Resultados del 2017
Como ya os avanzaba anteriormente el año pasado los resultados no fueron muy buenos. Del total de 40 empresas que analizamos unicamente un 37,5% hacia uso de HTTPS frente a un 62,5% que no usaba esta tecnología.

![Gráfico con los datos mencionados anteriormente](https://github.com/dsfau/dsfau.github.com/blob/master/_posts/_img/https_implementation_2017.jpeg?raw=true)

Este resultado ya es llamativo puesto que el sujeto analizado son empresas con una facturación considerablemente alta y el implementar esta tecnología no supone económicamente un alto gasto.

Visto este resultado indagué algo más en ese 37,5% de empresas que si hacia uso de HTTPS para saber si se hacía un uso correcto o no.

Los resultados no son para nada buenos, unicamente un 26,6% obtiene una calificación entre "A" y "B" a los cuales les podríamos llamar "aprobados", frente a un 73,4% que "suspende" en esta calificación.

![Gráfico con los datos mencionados anteriormente](https://github.com/dsfau/dsfau.github.com/blob/master/_posts/_img/https_quality_2017.jpeg?raw=true)

Lo que nos da una escalofriante cifra SOLO UN 10% (9,98%), de las 40 empresas objeto de estudio, aprueba en una implementación correcta de el protocolo HTTPS.

## Resultados en 2018
A comienzos de este año he decidido repetir el mismo análisis, con las mismas 40 empresas del análisis anterior, con el fin de saber si se ha mejorado en algo o no.

Los resultados han sido mejores que hace unos meses.

![Gráfico con los datos mencionados anteriormente](https://github.com/dsfau/dsfau.github.com/blob/master/_posts/_img/https_implementation_2018.png?raw=true)

El grado de implementación de HTTPS ha subido aunque esta cifra es algo engañosa y ahora explico el porque.

Entendemos como implementación de HTTPS que exista una forma posible de acceder a la web vía protocolo seguro, sin entrar en detalles técnicos de si el certificado esta o no expedido para el dominio al cual tu estas accediendo. Esto lleva a confusión puesto que la mayor parte de las webs están alojadas en plataformas de hosting compartido y en dicho hosting existen clientes que si hacen uso de HTTPS lo cual permite, en algunos casos, que tú accedas a un dominio hospedado en la misma plataforma, via un certificado en el cual el Common Name del certificado no coincida con el dominio al cual accedes.

El grado de implementación, tal cual queda explicado en el párrafo anterior como la posibilidad técnica de acceder a la web por HTTPS, aumenta considerablemente.
De un 37,5% en 2017 a un 75% en Enero de 2018, frente a un 25% que se resiste a usar esta tecnología.

Pasemos ahora a analizar la correcta o no implementación del protocolo. Es aquí donde los datos cambian rotundamente. De ese 75% que si hace uso de HTTPS obtenemos que solo un 23,33% aprobaría en una correcta implementación. O lo que es igual de las 40 empresas estudiadas un 17,5% aprueba frete a un 82% que no.

![Gráfico con los datos mencionados anteriormente](https://github.com/dsfau/dsfau.github.com/blob/master/_posts/_img/https_quality_2018.png?raw=true)

Otro dato interesante, un 30,3% del 75% que hace uso de HTTPS tiene alguna vulnerabilidad relacionada con el protocolo. (Freack, Logjam, Drown, Poodle-TLS, Poodle-SSL, DH Inseguro, SSLv2)

![Gráfico con los datos mencionados anteriormente](https://github.com/dsfau/dsfau.github.com/blob/master/_posts/_img/some_vulnerability_2018.png?raw=true)

Relacionado con lo anterior y sorprendido con abultado porcentaje (30,3%), me decidí a mirar que versiones de TLS se estaban permitiendo para la negociación. Aquí respiré aliviado al ver que todos ofrecían TLS1.2, el 100% del 75% que usa HTTPS (la ultima versión del protocolo), empece a estar más inquieto al ver que también todos ofrecían la TLS1.1, y ya me entraron sudores fríos al ver que solo uno se había molestado en deshabilitar la versión TLS1.0 que como todos sabréis tiene vulnerabilidades conocidas.

![Gráfico con los datos mencionados anteriormente](https://github.com/dsfau/dsfau.github.com/blob/master/_posts/_img/tls_versions_2018.png?raw=true)

## Conclusiones
No querría añadir ninguna valoración personal, son datos objetivos que dan poco lugar a ningún tipo de valoración. Solo se puede decir algo obvio y es que hay mucho camino que recorrer para mejorar esta situación pese a que algo ha mejorado con respecto al año pasado.

Si alguien quisiera algún dato más en concreto me lo puede pedir por e-mail.

Saludos
