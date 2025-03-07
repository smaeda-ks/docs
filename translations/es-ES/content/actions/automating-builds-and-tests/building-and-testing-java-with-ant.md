---
title: Construir y probar Java con Ant
intro: Puedes crear un flujo de trabajo de integración continua (CI) en Acciones de GitHub para construir y probar tu proyecto Java con Ant.
redirect_from:
  - /actions/language-and-framework-guides/building-and-testing-java-with-ant
  - /actions/guides/building-and-testing-java-with-ant
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
type: tutorial
topics:
  - CI
  - Java
  - Ant
shortTitle: Crear & probar con Java & Ant
---

{% data reusables.actions.enterprise-beta %}
{% data reusables.actions.enterprise-github-hosted-runners %}

## Introducción

Esta guía te muestra cómo crear un flujo de trabajo que realiza integración continua (CI) para tu proyecto de Java por medio del sistema de construcción Ant. El flujo de trabajo que creas te permitirá ver cuándo las confirmaciones de una solicitud de extracción causan la construcción o las fallas de prueba en tu rama por defecto; este enfoque puede ayudar a garantizar que tu código siempre sea correcto. Puedes ampliar tu flujo de trabajo de CI para cargar artefactos desde una ejecución de flujo de trabajo.

{% ifversion ghae %}
{% data reusables.actions.self-hosted-runners-software %}
{% else %}
Los ejecutores alojados en {% data variables.product.prodname_dotcom %} tienen un caché de herramientas con software preinstalado, que incluye Java Development Kits (JDK) y Ant. Para encontrar una lista de software y de las versiones pre-instaladas de JDK y de Ant, consulta la sección "[Especificaciones para los ejecutores hospedados en {% data variables.product.prodname_dotcom %}](/actions/reference/specifications-for-github-hosted-runners/#supported-software)".
{% endif %}

## Prerrequisitos

Deberías estar familiarizado con YAML y la sintaxis para las {% data variables.product.prodname_actions %}. Para obtener más información, consulta:
- "[Sintaxis de flujo de trabajo para las {% data variables.product.prodname_actions %}](/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions)"
- "[Aprende sobre las {% data variables.product.prodname_actions %}](/actions/learn-github-actions)"

Recomendamos que tengas un conocimiento básico de Java y de la estructura de Ant. Para obtener más información, consulta el [Manual de Apache Ant](https://ant.apache.org/manual/).

{% data reusables.actions.enterprise-setup-prereq %}

## Utilizar el flujo de trabajo inicial de Ant

{% data variables.product.prodname_dotcom %} proporciona un flujo de trabajo inicial de Ant que funcionará para la mayoría de los proyectos de Java basados en Ant. Para obtener más información, consulta el [Flujo de trabajo inicial de Ant](https://github.com/actions/starter-workflows/blob/main/ci/ant.yml).

Para comenzar rápidamente, puedes elegir el flujo de trabajo inicial de Ant preconfigurado cuando crees un nuevo flujo de trabajo. Para obtener más información, consulta la "[guía rápida de {% data variables.product.prodname_actions %}](/actions/quickstart)".

También puedes agregar este flujo de trabajo de forma manual al crear un archivo nuevo en el directorio de tu repositorio `.github/workflows`.

{% raw %}
```yaml{:copy}
name: Java CI

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          java-version: '11'
          distribution: 'adopt'
      - name: Build with Ant
        run: ant -noinput -buildfile build.xml
```
{% endraw %}

Este flujo de trabajo realiza los siguientes pasos:

1. El paso `checkout (comprobación)` descarga una copia de tu repositorio en el ejecutor.
2. El paso `setup-java` configura el JDK de Java 11 por Adoptium.
3. El paso "Build with Ant" (Construir con Ant) ejecuta el objetivo predeterminado en tu `build.xml` en el modo no interactivo.

Los flujos de trabajo iniciales predeterminados son un punto de partida excelente para crear tu flujo de trabajo de prueba y de compilación y puedes personalizarlos de acuerdo con las necesidades de tu proyecto.

{% data reusables.github-actions.example-github-runner %}

{% data reusables.github-actions.java-jvm-architecture %}

## Construir y probar tu código

Puedes usar los mismos comandos que usas de forma local para construir y probar tu código.

El flujo de trabajo de inicio ejecutará el destino predeterminado especificado en tu archivo _build.xml_.  Normalmente, tu objetivo predeterminado se configurará para crear clases, ejecutar pruebas y empaquetar clases en su formato distribuible, por ejemplo, un archivo JAR.

Si usas diferentes comandos para construir tu proyecto, o si deseas ejecutar un objetivo diferente, puedes especificarlos. Por ejemplo, es posible que desees ejecutar el destino `jar` que está configurado en tu archivo `-ci.xml_`.

{% raw %}
```yaml{:copy}
steps:
  - uses: actions/checkout@v2
  - uses: actions/setup-java@v2
    with:
      java-version: '11'
      distribution: 'adopt'
  - name: Run the Ant jar target
    run: ant -noinput -buildfile build-ci.xml jar
```
{% endraw %}

## Empaquetar datos de flujo de trabajo como artefactos

Una vez que tu compilación haya tenido éxito y tus pruebas hayan pasado, es posible que desees cargar los paquetes Java resultantes como un artefacto de construcción. Esto almacenará los paquetes construidos como parte de la ejecución del flujo de trabajo y te permitirá descargarlos. Los artefactos pueden ayudarte a probar y depurar solicitudes de extracción en tu entorno local antes de que se fusionen. Para obtener más información, consulta "[Conservar datos de flujo de trabajo mediante artefactos](/actions/automating-your-workflow-with-github-actions/persisting-workflow-data-using-artifacts)."

Por lo general, Ant crea archivos de salida como JAR, EAR o WAR en el directorio `build/jar`. Puedes cargar los contenidos de ese directorio utilizando la acción `upload-Artifact`.

{% raw %}
```yaml{:copy}
steps:
  - uses: actions/checkout@v2
  - uses: actions/setup-java@v2
    with:
      java-version: '11'
      distribution: 'adopt'

  - run: ant -noinput -buildfile build.xml
  - uses: actions/upload-artifact@v2
    with:
      name: Package
      path: build/jar
```
{% endraw %}
