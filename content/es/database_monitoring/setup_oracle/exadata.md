---
description: Instala y configura la Monitorización de base de datos para Oracle Exadata
further_reading:
- link: /integrations/oracle/
  tag: Documentación
  text: Integración básica de Oracle
title: Configuración de la Monitorización de base de datos para Oracle Exadata
---

{{% dbm-oracle-definition %}}

El Agent recopila telemetría directamente de la base de datos iniciando sesión como usuario de sólo lectura.

## Antes de empezar

{{% dbm-supported-oracle-versions %}}

{{% dbm-supported-oracle-agent-version %}}

Impacto del rendimiento
: el valor predeterminado de configuración del Agent para la Monitorización de base de datos es conservador, pero puedes ajustar parámetros como el intervalo de recopilación y la frecuencia de muestreo de consultas para que se adapten mejor a tus necesidades. Para la mayoría de las cargas de trabajo, el Agent representa menos del uno por ciento del tiempo de ejecución de consultas en la base de datos y menos del uno por ciento de la CPU. <br/><br/>
La Monitorización de base de datos se ejecuta como integración sobre el Agent base ([ver valores de referencia][6]).

Proxies, equilibradores de carga y agrupadores de conexiones
: El Agent debe conectarse directamente al host que está siendo monitorizado. El Agent no debe conectarse a la base de datos a través de un proxy, equilibrador de carga o agrupador de conexiones. Cada Agent debe tener conocimiento del nombre de host subyacente y debe ceñirse a un único host durante toda su vida, incluso en casos de conmutación por error. Si el Datadog Agent se conecta a diferentes hosts mientras se está ejecutando, los valores de las métricas serán incorrectos.

Consideraciones sobre la seguridad de los datos
: para saber qué datos recopila el Agent de tus bases de datos y cómo garantizar su seguridad, consulta [Información confidencial][7].

## Configuración

Sigue los siguientes pasos para habilitar la Monitorización de base de datos con tu base de datos Oracle:

1. [Crea el usuario Datadog](#create-the-datadog-user).
1. [Instala el Agent](#install-the-agent).
1. [Configura el Agent](#configure-the-agent).
1. [Instala o verifica la integración de Oracle](#install-or-verify-the-oracle-integration).
1. [Confirma la configuración](#validate-the-setup).

### Crear el usuario de Datadog

{{% dbm-create-oracle-user %}}

### Instalación del Agent

Para determinar dónde instalar el Agent, consulta la documentación [Arquitecturas de configuración de DBM][12]. El Agent no requiere ningún cliente de Oracle externo.

Para conocer los pasos de instalación, consulta las [instrucciones de instalación del Agent][9].

### Configurar el Agent

#### Exadata multinodo

Configura el Agent para cada nodo siguiendo las instrucciones para [Oracle RAC][4].

#### Exadata de nodo único

Configura el Agent siguiendo las instrucciones para [bases de datos de Oracle autoalojadas][3].

### Instalar o verificar la integración de Oracle

#### Primeras instalaciones

En la página de integraciones de Datadog, instala la [integración de Oracle][10] para tu organización. Se instala un [dashboard de Oracle][11] en tu cuenta que puedes utilizar para monitorizar el rendimiento de tus bases de datos Oracle.

#### Instalaciones existentes

{{% dbm-existing-oracle-integration-setup %}}

### Validar la configuración

[Ejecuta el subcomando de estado del Agent][1] y busca `oracle` en la sección **Checks**. Visita la página [Dashboard][11] y [Bases de datos][2] de Datadog para empezar.

## Consultas personalizadas

La Monitorización de base de datos admite consultas personalizadas para bases de datos Oracle. Para obtener más información sobre las opciones de configuración disponibles, consulta [conf.yaml.example][5].

<div class="alert alert-warning">La ejecución de consultas personalizadas puede dar lugar a costes o tasas adicionales evaluados por Oracle.</div>

[1]: /es/agent/configuration/agent-commands/#agent-status-and-information
[2]: https://app.datadoghq.com/databases
[3]: /es/database_monitoring/setup_oracle/selfhosted
[4]: /es/database_monitoring/setup_oracle/rac
[5]: https://github.com/DataDog/datadog-agent/blob/main/cmd/agent/dist/conf.d/oracle.d/conf.yaml.example
[6]: /es/database_monitoring/agent_integration_overhead/?tab=oracle
[7]: /es/database_monitoring/data_collected/#sensitive-information
[8]: https://app.datadoghq.com/dash/integration/30990/dbm-oracle-database-overview
[9]: https://app.datadoghq.com/account/settings/agent/latest
[10]: https://app.datadoghq.com/integrations/oracle
[11]: https://app.datadoghq.com/dash/integration/30990/dbm-oracle-database-overview
[12]: /es/database_monitoring/architecture/

## Referencias adicionales

{{< partial name="whats-next/whats-next.html" >}}