// 1. Estructura para datos del payload entre tarea y malla
struct TaskPayload {
  colorMask: vec4,
  visible: bool,
};

// 2. Estructura para salida de vértices (al rasterizador)
struct MeshVertexOutput {
  @builtin(position) position: vec4,
  @location(0) color: vec4, // Location para pasar al fragment shader
};

// 3. Estructura para datos de cada primitiva
struct MeshPrimitiveOutput {
  @builtin(primitive_indices) indices: vec3,
  @builtin(primitive_cull) cull: bool,
  @location(1) colorMask: vec4, // Per-primitive data al fragment shader
};

// 4. Variable de grupo de trabajo
var workgroupData: f32;

// 5. Buffers de entrada (ajustados a tu proyecto si es necesario)
@group(0) @binding(0) var positions: array<vec4, 3>;
@group(0) @binding(1) var colors: array<vec4, 3>;

// ------------------------------
// Shader de Tarea
// ------------------------------
@task @payload(taskPayload: TaskPayload) // Especificar tipo del payload
@workgroup_size(1)
fn ts_main() -> @builtin(mesh_task_size) vec3 {
  workgroupData = 1.0;

  // Configurar datos para el mesh shader
  taskPayload.colorMask = vec4(1.0, 1.0, 0.0, 1.0);
  taskPayload.visible = true;

  // Dispatch: 1x1x1 workgroups para el mesh shader
  return vec3(1u, 1u, 1u);
}

// ------------------------------
// Shader de Malla
// ------------------------------
@mesh(
  @builtin(mesh_vertices) vertices: array<MeshVertexOutput, 3>, // Máx 3 vértices
  @builtin(mesh_primitives) primitives: array<MeshPrimitiveOutput, 1> // Máx 1 primitiva
)
@payload(taskPayload: TaskPayload) // Recibir payload de la tarea
@workgroup_size(1)
fn ms_main() {
  workgroupData = 2.0;

  // Configurar vértices
  vertices[0].position = positions[0];
  vertices[0].color = colors[0] * taskPayload.colorMask;

  vertices[1].position = positions[1];
  vertices[1].color = colors[1] * taskPayload.colorMask;

  vertices[2].position = positions[2];
  vertices[2].color = colors[2] * taskPayload.colorMask;

  // Configurar primitiva (triángulo)
  primitives[0].indices = vec3<u32>(0u, 1u, 2u);
  primitives[0].cull = !taskPayload.visible;
  primitives[0].colorMask = vec4<f32>(1.0, 0.0, 1.0, 1.0);
}
<iframe width="560" height="315" src="https://www.youtube.com/embed/ID_DEL_VIDEO" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<iframe width="560" height="315" src="https://www.youtube.com/embed/ID_DEL_VIDEO" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
// 1. Estructura para datos del payload entre tarea y malla
struct TaskPayload {
  colorMask: vec4,
  visible: bool,
};

// 2. Estructura para salida de vértices (al rasterizador)
struct MeshVertexOutput {
  @builtin(position) position: vec4,
  @location(0) color: vec4, // Location para pasar al fragment shader
};

// 3. Estructura para datos de cada primitiva
struct MeshPrimitiveOutput {
  @builtin(primitive_indices) indices: vec3,
  @builtin(primitive_cull) cull: bool,
  @location(1) colorMask: vec4, // Per-primitive data al fragment shader
};

// 4. Variable de grupo de trabajo
var workgroupData: f32;

// 5. Buffers de entrada (ajustados a tu proyecto si es necesario)
@group(0) @binding(0) var positions: array<vec4, 3>;
@group(0) @binding(1) var colors: array<vec4, 3>;

// ------------------------------
// Shader de Tarea
// ------------------------------
@task @payload(taskPayload: TaskPayload) // Especificar tipo del payload
@workgroup_size(1)
fn ts_main() -> @builtin(mesh_task_size) vec3 {
  workgroupData = 1.0;

  // Configurar datos para el mesh shader
  taskPayload.colorMask = vec4(1.0, 1.0, 0.0, 1.0);
  taskPayload.visible = true;

  // Dispatch: 1x1x1 workgroups para el mesh shader
  return vec3(1u, 1u, 1u);
}

// ------------------------------
// Shader de Malla
// ------------------------------
@mesh(
  @builtin(mesh_vertices) vertices: array<MeshVertexOutput, 3>, // Máx 3 vértices
  @builtin(mesh_primitives) primitives: array<MeshPrimitiveOutput, 1> // Máx 1 primitiva
)
@payload(taskPayload: TaskPayload) // Recibir payload de la tarea
@workgroup_size(1)
fn ms_main() {
  workgroupData = 2.0;

  // Configurar vértices
  vertices[0].position = positions[0];
  vertices[0].color = colors[0] * taskPayload.colorMask;

  vertices[1].position = positions[1];
  vertices[1].color = colors[1] * taskPayload.colorMask;

  vertices[2].position = positions[2];
  vertices[2].color = colors[2] * taskPayload.colorMask;

  // Configurar primitiva (triángulo)
  primitives[0].indices = vec3<u32>(0u, 1u, 2u);
  primitives[0].cull = !taskPayload.visible;
  primitives[0].colorMask = vec4<f32>(1.0, 0.0, 1.0, 1.0);
}
Ajustes adicionales importantes
 
- Se especifica el tipo del payload en los decoradores  @payload(...)  para que el compilador lo reconozca.

- Los datos per-primitive ( colorMask ) tienen un  @location  para que el fragment shader los pueda leer.

- Se usan sufijos  u  en todos los valores enteros sin signo para evitar ambigüedades de tipo.

- La estructura de salida de vértices incluye  @location  para el color, lo cual es necesario para pasar información al fragment shader.
 
Para que funcione en tu proyecto GitHub ("truk")
 
- Asegúrate de que los bindings de  positions  y  colors  coincidan con cómo creas los bind groups en tu código Rust/wgpu.Descripción para Radame (proyecto "truk" - Shader de Malla)
 
Este código implementa un sistema de shaders de tarea y malla (mesh/task shaders) compatible con wgpu, diseñado para renderizar geometría de forma eficiente mediante el uso de "meshlets" (pequeños grupos de triángulos). Forma parte del proyecto "truk" y está optimizado para generar y configurar primitivas de renderizado de manera programable, reemplazando los pipelines tradicionales de shaders de vértice.
 
Funcionalidad principal
 
- Shader de Tarea ( ts_main ): Configura datos compartidos para todos los shaders de malla del grupo de trabajo (como una máscara de color y un indicador de visibilidad), y define cuántos grupos de trabajo de malla se despacharán (en este caso, 1x1x1).
​
- Shader de Malla ( ms_main ): Genera un triángulo completo, asignando posiciones y colores a sus vértices a partir de buffers de entrada. También configura propiedades de la primitiva (índices de vértice, activación de culling y datos per-primitive para el shader de fragmento).
 
Componentes clave
 
-  TaskPayload : Estructura que transmite datos entre el shader de tarea y el de malla, permitiendo configurar propiedades globales para cada grupo de geometría.
​
-  MeshVertexOutput : Define la información de cada vértice (posición en espacio clip y color) que se envía al rasterizador.
​
-  MeshPrimitiveOutput : Controla propiedades de la primitiva, como los índices de los vértices que la forman, si debe ser ocultada (culling) y datos no interpolados para el shader de fragmento.
​
- Buffers de entrada: Se leen posiciones y colores de vértices desde memoria uniforme, adaptables a las necesidades específicas de los modelos 3D de "truk".
 
Ventajas en el proyecto
 
- Mayor flexibilidad en la generación de geometría en comparación con pipelines tradicionales.
​
- Optimización para renderizado de meshlets, reduciendo el tráfico de memoria y mejorando el culling.
​
- Compatibilidad con los estándares de WebGPU y los backends principales de wgpu (Vulkan, Metal, DX12).
 
 
 
¿Quieres que la descripción sea más técnica (para documentación de código) o más general (para un README del repositorio)? ¡Avísame y la ajusto! 😊

- Si tus posiciones son  vec3<f32> , conviértelas a  vec4<f32>  en el shader (ej:  vec4(positions[0], 1.0) ).

- Verifica que en tu pipeline de wgpu hayas habilitado la característica de mesh shaders y configurado correctamente los layouts de pipeline.
 
¿Necesitas que te ayude a adaptar el código Rust correspondiente para usar este shader en tu proyecto?// 1. Estructura para datos del payload entre tarea y malla
struct TaskPayload {
    colorMask: vec4<f32>,
    visible: bool,
};

// 2. Estructura para salida de vértices (al rasterizador)
struct MeshVertexOutput {
    @builtin(position) position: vec4<f32>,
    @location(0) color: vec4<f32>, // Location para pasar al fragment shader
};

// 3. Estructura para datos de cada primitiva
struct MeshPrimitiveOutput {
    @builtin(primitive_indices) indices: vec3<u32>,
    @builtin(primitive_cull) cull: bool,
    @location(1) colorMask: vec4<f32>, // Per-primitive data al fragment shader
};

// 4. Variable de grupo de trabajo
var<workgroup> workgroupData: f32;

// 5. Buffers de entrada (ajustados a tu proyecto si es necesario)
@group(0) @binding(0)
var<uniform> positions: array<vec4<f32>, 3>;

@group(0) @binding(1)
var<uniform> colors: array<vec4<f32>, 3>;


// ------------------------------
// Shader de Tarea
// ------------------------------
@task
@payload(taskPayload: TaskPayload) // Especificar tipo del payload
@workgroup_size(1)
fn ts_main() -> @builtin(mesh_task_size) vec3<u32> {
    workgroupData = 1.0;
    
    // Configurar datos para el mesh shader
    taskPayload.colorMask = vec4(1.0, 1.0, 0.0, 1.0);
    taskPayload.visible = true;
    
    // Dispatch: 1x1x1 workgroups para el mesh shader
    return vec3(1u, 1u, 1u);
}


// ------------------------------
// Shader de Malla
// ------------------------------
@mesh(
    @builtin(mesh_vertices) vertices: array<MeshVertexOutput, 3>, // Máx 3 vértices
    @builtin(mesh_primitives) primitives: array<MeshPrimitiveOutput, 1> // Máx 1 primitiva
)
@payload(taskPayload: TaskPayload) // Recibir payload de la tarea
@workgroup_size(1)
fn ms_main() {
    workgroupData = 2.0;

    // Configurar vértices
    vertices[0].position = positions[0];
    vertices[0].color = colors[0] * taskPayload.colorMask;

    vertices[1].position = positions[1];
    vertices[1].color = colors[1] * taskPayload.colorMask;

    vertices[2].position = positions[2];
    vertices[2].color = colors[2] * taskPayload.colorMask;
    
    // Configurar primitiva (triángulo)
    primitives[0].indices = vec3<u32>(0u, 1u, 2u);
    primitives[0].cull = !taskPayload.visible;
    primitives[0].colorMask = vec4<f32>(1.0, 0.0, 1.0, 1.0);
}
name: "Build and Test For PR"

on: [push, pull_request, workflow_dispatch]

permissions:
  contents: read

env:
  FORK_COUNT: 2
  FAIL_FAST: 0
  SHOW_ERROR_DETAIL: 1
  VERSIONS_LIMIT: 4
  JACOCO_ENABLE: true
  CANDIDATE_VERSIONS: '
    spring.version:5.3.24,6.1.5;
    spring-boot.version:2.7.6,3.2.3;
    '
  MAVEN_OPTS: >-
    -XX:+UseG1GC
    -XX:InitiatingHeapOccupancyPercent=45
    -XX:+UseStringDeduplication
    -XX:-TieredCompilation
    -XX:TieredStopAtLevel=1
    -Dmaven.javadoc.skip=true
    -Dmaven.wagon.http.retryHandler.count=5
    -Dmaven.wagon.httpconnectionManager.ttlSeconds=120
  MAVEN_ARGS: >-
    -e
    --batch-mode
    --no-snapshot-updates
    --no-transfer-progress
    --fail-fast

jobs:
  # 1. Comprobación de formato de código
  check-format:
    name: "Check if code needs formatting"
    runs-on: ubuntu-22.04
    steps:
      - name: "Checkout"
        uses: actions/checkout@v4
      - name: "Setup maven"
        uses: actions/setup-java@v4
        with:
          java-version: 21
          distribution: zulu
      - name: "Check if code aligns with code style"
        id: check
        run: mvn --log-file mvn.log spotless:check
        continue-on-error: true
      - name: "Upload checkstyle result"
        uses: actions/upload-artifact@v4
        with:
          name: checkstyle-result
          path: mvn.log
      - name: "Generate Summary for successful run"
        if: ${{ steps.check.outcome == 'success' }}
        run: |
          echo ":ballot_box_with_check: Kudos! No formatting issues found!" >> $GITHUB_STEP_SUMMARY
      - name: "Generate Summary for failed run"
        if: ${{ steps.check.outcome == 'failure' }}
        run: |
          echo "## :negative_squared_cross_mark: Formatting issues found!" >> $GITHUB_STEP_SUMMARY
          echo "\`\`\`" >> $GITHUB_STEP_SUMMARY
          cat mvn.log | grep "ERROR" | sed 's/Check if code needs formatting    Check if code aligns with code style   [0-9A-Z:.-]\+//' | sed 's/\[ERROR] //' | head -n -11 >> $GITHUB_STEP_SUMMARY
          echo "\`\`\`" >> $GITHUB_STEP_SUMMARY
          echo "Please run \`mvn spotless:apply\` to fix the formatting issues." >> $GITHUB_STEP_SUMMARY
      - name: "Fail if code needs formatting"
        if: ${{ steps.check.outcome == 'failure' }}
        uses: actions/github-script@v7.0.1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            core.setFailed("Formatting issues found! \nRun \`mvn spotless:apply\` to fix.")

  # 2. Comprobación de licencias
  license:
    name: "Check License"
    needs: check-format
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v4
      - name: "Check License"
        uses: apache/skywalking-eyes@e1a02359b239bd28de3f6d35fdc870250fa513d5
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      - name: "Set up JDK 21"
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: 21
      - name: "Compile Dubbo (Linux)"
        run: |
          ./mvnw ${{ env.MAVEN_ARGS }} -T 2C clean install -Pskip-spotless -Dmaven.test.skip=true -Dcheckstyle.skip=true -Dcheckstyle_unix.skip=true -Drat.skip=true
      - name: "Check Dependencies' License"
        uses: apache/skywalking-eyes/dependency@e1a02359b239bd28de3f6d35fdc870250fa513d5
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          config: .licenserc.yaml
          mode: check

  # 3. Construcción de fuente
  build-source:
    name: "Build Dubbo"
    needs: check-format
    runs-on: ubuntu-22.04
    outputs:
      version: ${{ steps.dubbo-version.outputs.version }}
    steps:
      - name: "Checkout code"
        uses: actions/checkout@v4
      - name: "Set up JDK"
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: 21
      - name: "Set current date as env variable"
        run: echo "TODAY=$(date +'%Y%m%d')" >> $GITHUB_ENV
      - name: "Restore local maven repository cache"
        uses: actions/cache/restore@v4
        id: cache-maven-repository
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}-${{ env.TODAY }}
      - name: "Restore common local maven repository cache"
        uses: actions/cache/restore@v4
        if: steps.cache-maven-repository.outputs.cache-hit != 'true'
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
          restore-keys: |
            ${{ runner.os }}-maven-
      - name: "Clean dubbo cache"
        run: rm -rf ~/.m2/repository/org/apache/dubbo
      - name: "Build Dubbo with maven"
        run: |
          ./mvnw ${{ env.MAVEN_ARGS }} clean install -Psources,skip-spotless,checkstyle -Dmaven.test.skip=true -DembeddedZookeeperPath=${{ github.workspace }}/.tmp/zookeeper
      - name: "Save dubbo cache"
        uses: actions/cache/save@v4
        with:
          path: ~/.m2/repository/org/apache/dubbo
          key: ${{ runner.os }}-dubbo-snapshot-${{ github.sha }}-${{ github.run_id }}
      - name: "Clean dubbo cache"
        run: rm -rf ~/.m2/repository/org/apache/dubbo
      - name: "Save local maven repository cache"
        uses: actions/cache/save@v4
        if: steps.cache-maven-repository.outputs.cache-hit != 'true'
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}-${{ env.TODAY }}
      - name: "Pack class result"
        run: |
          shopt -s globstar
          zip ${{ github.workspace }}/class.zip **/target/classes/* -r
      - name: "Upload class result"
        uses: actions/upload-artifact@v4
        with:
          name: "class-file"
          path: ${{ github.workspace }}/class.zip
      - name: "Pack checkstyle file if failure"
        if: failure()
        run: zip ${{ github.workspace }}/checkstyle.zip *checkstyle* -r
      - name: "Upload checkstyle file if failure"
        uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: "checkstyle-file"
          path: ${{ github.workspace }}/checkstyle.zip
      - name: "Calculate Dubbo Version"
        id: dubbo-version
        run: |
          REVISION=`awk '/<revision>[^<]+<\/revision>/{gsub(/<revision>|<\/revision>/,"",$1);print $1;exit;}' ./pom.xml`
          echo "version=$REVISION" >> $GITHUB_OUTPUT
          echo "dubbo version: $REVISION"

  # 4. Preparación para pruebas unitarias
  unit-test-prepare:
    name: "Preparation for Unit Test"
    needs: check-format
    runs-on: ubuntu-22.04
    env:
      ZOOKEEPER_VERSION: 3.7.2
    steps:
      - name: "Cache zookeeper binary archive"
        uses: actions/cache/restore@v4
        id: "cache-zookeeper"
        with:
          path: ${{ github.workspace }}/.tmp/zookeeper
          key: zookeeper-${{ runner.os }}-${{ env.ZOOKEEPER_VERSION }}
          restore-keys: |
            zookeeper-${{ runner.os }}-${{ env.ZOOKEEPER_VERSION }}
      - name: "Set up msys2 if necessary"
        uses: msys2/setup-msys2@v2
        if: ${{ startsWith( runner.os, 'Windows') && steps.cache-zookeeper.outputs.cache-hit != 'true' }}
        with:
          release: false
      - name: "Download zookeeper binary archive"
        if: steps.cache-zookeeper.outputs.cache-hit != 'true'
        run: |
          mkdir -p ${{ github.workspace }}/.tmp/zookeeper
          wget -t 1 -T 120 -c https://archive.apache.org/dist/zookeeper/zookeeper-${{ env.ZOOKEEPER_VERSION }}/apache-zookeeper-${{ env.ZOOKEEPER_VERSION }}-bin.tar.gz -O ${{ github.workspace }}/.tmp/zookeeper/apache-zookeeper-bin.tar.gz ||
          wget -t 1 -T 120 -c https://apache.website-solution.net/zookeeper/zookeeper-${{ env.ZOOKEEPER_VERSION }}/apache-zookeeper-${{ env.ZOOKEEPER_VERSION }}-bin.tar.gz -O ${{ github.workspace }}/.tmp/zookeeper/apache-zookeeper-bin.tar.gz ||
          wget -t 1 -T 120 -c http://ftp.jaist.ac.jp/pub/apache/zookeeper/zookeeper-${{ env.ZOOKEEPER_VERSION }}/apache-zookeeper-${{ env.ZOOKEEPER_VERSION }}-bin.tar.gz -O ${{ github.workspace }}/.tmp/zookeeper/apache-zookeeper-bin.tar.gz ||
          wget -t 1 -T 120 -c http://apache.mirror.cdnetworks.com/zookeeper/zookeeper-${{ env.ZOOKEEPER_VERSION }}/apache-zookeeper-${{ env.ZOOKEEPER_VERSION }}-bin.tar.gz -O ${{ github.workspace }}/.tmp/zookeeper/apache-zookeeper-bin.tar.gz ||
          wget -t 1 -T 120 -c http://mirror.apache-kr.org/apache/zookeeper/zookeeper-${{ env.ZOOKEEPER_VERSION }}/apache-zookeeper-${{ env.ZOOKEEPER_VERSION }}-bin.tar.gz -O ${{ github.workspace }}/.tmp/zookeeper/apache-zookeeper-bin.tar.gz
          ls -al ${{ github.workspace }}/.tmp/zookeeper/apache-zookeeper-bin.tar.gz
      - name: "Save zookeeper cache"
        uses: actions/cache/save@v4
        if: steps.cache-zookeeper.outputs.cache-hit != 'true'
        with:
          path: ${{ github.workspace }}/.tmp/zookeeper
          key: zookeeper-${{ runner.os }}-${{ env.ZOOKEEPER_VERSION }}

  # 5. Pruebas unitarias
  unit-test:
    needs: [check-format, unit-test-prepare]
    name: "Unit Test On ubuntu-22.04 Java: ${{ matrix.java }}"
    runs-on: ubuntu-22.04
    strategy:
      fail-fast: false
      matrix:
        java: [ 8, 11, 17, 21, 25 ]
    env:
      DISABLE_FILE_SYSTEM_TEST: true
      ZOOKEEPER_VERSION: 3.7.2
    steps:
      - name: "Set MAVEN_OPTS for JDK 24+"
        if: ${{ matrix.java >= 24 }}
        run: echo "MAVEN_OPTS=--sun-misc-unsafe-memory-access=allow" >> $GITHUB_ENV
      - name: "Checkout code"
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: "Set up JDK ${{ matrix.java }}"
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: ${{ matrix.java }}
      - name: "Set current date as env variable"
        run: echo "TODAY=$(date +'%Y%m%d')" >> $GITHUB_ENV
      - name: "Cache local maven repository"
        uses: actions/cache/restore@v4
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}-${{ env.TODAY }}
          restore-keys: |
            ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
            ${{ runner.os }}-maven-
      - name: "Restore zookeeper binary archive"
        uses: actions/cache/restore@v4
        id: "cache-zookeeper"
        with:
          path: ${{ github.workspace }}/.tmp/zookeeper
          key: zookeeper-${{ runner.os }}-${{ env.ZOOKEEPER_VERSION }}
      - name: "Test with maven on Java: 8"
        timeout-minutes: 90
        if: ${{ matrix.java == '8' }}
        run: |
          set -o pipefail
          ./mvnw ${{ env.MAVEN_ARGS }} clean test verify -Pjacoco,'!jdk15ge-add-open',skip-spotless -DtrimStackTrace=false -Dmaven.test.skip=false -Dcheckstyle.skip=false -Dcheckstyle_unix.skip=false -Drat.skip=false -DembeddedZookeeperPath=${{ github.workspace }}/.tmp/zookeeper 2>&1 |
## Vídeo relacionado
[![Título del vídeo](https://img.youtube.com/vi/ID_DEL_VIDEO/0.jpg)](https://www.youtube.com/watch?v=ID_DEL_VIDEO)

O inserta el reproductor directamente:
<iframe width="560" height="315" src="https://www.youtube.com/embed/ID_DEL_VIDEO" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<img align="right" width="25%" src="logo.png">

# wgpu

[![Matrix Space](https://img.shields.io/static/v1?label=Space&message=%23Wgpu&color=blue&logo=matrix)](https://matrix.to/#/#Wgpu:matrix.org)
[![Dev Matrix  ](https://img.shields.io/static/v1?label=devs&message=%23wgpu&color=blueviolet&logo=matrix)](https://matrix.to/#/#wgpu:matrix.org)
[![User Matrix ](https://img.shields.io/static/v1?label=users&message=%23wgpu-users&color=blueviolet&logo=matrix)](https://matrix.to/#/#wgpu-users:matrix.org)
[![Build Status](https://github.com/gfx-rs/wgpu/workflows/CI/badge.svg)](https://github.com/gfx-rs/wgpu/actions)
[![codecov.io](https://codecov.io/gh/gfx-rs/wgpu/branch/trunk/graph/badge.svg?token=84qJTesmeS)](https://codecov.io/gh/gfx-rs/wgpu)

`wgpu` is a cross-platform, safe, pure-rust graphics API. It runs natively on Vulkan, Metal, D3D12, and OpenGL; and on top of WebGL2 and WebGPU on wasm.

The API is based on the [WebGPU standard](https://gpuweb.github.io/gpuweb/). It serves as the core of the WebGPU integration in Firefox, Servo, and Deno.

## Quick Links

| Docs                                 | Examples                                                             | Changelog                                                                    |
|:------------------------------------:|:--------------------------------------------------------------------:|:----------------------------------------------------------------------------:|
| [v24](https://docs.rs/wgpu/)         | [v24](https://github.com/gfx-rs/wgpu/tree/v24/examples#readme)       | [v24](https://github.com/gfx-rs/wgpu/releases)                               |
| [`trunk`](https://wgpu.rs/doc/wgpu/) | [`trunk`](https://github.com/gfx-rs/wgpu/tree/trunk/examples#readme) | [`trunk`](https://github.com/gfx-rs/wgpu/blob/trunk/CHANGELOG.md#unreleased) |

## Repo Overview

The repository hosts the following libraries:

- [![Crates.io](https://img.shields.io/crates/v/wgpu.svg?label=wgpu)](https://crates.io/crates/wgpu) [![docs.rs](https://docs.rs/wgpu/badge.svg)](https://docs.rs/wgpu/) - User facing Rust API.
- [![Crates.io](https://img.shields.io/crates/v/wgpu-core.svg?label=wgpu-core)](https://crates.io/crates/wgpu-core) [![docs.rs](https://docs.rs/wgpu-core/badge.svg)](https://docs.rs/wgpu-core/) - Internal safe implementation.
- [![Crates.io](https://img.shields.io/crates/v/wgpu-hal.svg?label=wgpu-hal)](https://crates.io/crates/wgpu-hal) [![docs.rs](https://docs.rs/wgpu-hal/badge.svg)](https://docs.rs/wgpu-hal/) - Internal unsafe GPU API abstraction layer.
- [![Crates.io](https://img.shields.io/crates/v/wgpu-types.svg?label=wgpu-types)](https://crates.io/crates/wgpu-types) [![docs.rs](https://docs.rs/wgpu-types/badge.svg)](https://docs.rs/wgpu-types/) - Rust types shared between all crates.
- [![Crates.io](https://img.shields.io/crates/v/naga.svg?label=naga)](https://crates.io/crates/naga) [![docs.rs](https://docs.rs/naga/badge.svg)](https://docs.rs/naga/) - Stand-alone shader translation library.
- [![Crates.io](https://img.shields.io/crates/v/deno_webgpu.svg?label=deno_webgpu)](https://crates.io/crates/deno_webgpu) - WebGPU implementation for the Deno JavaScript/TypeScript runtime

The following binaries:

- [![Crates.io](https://img.shields.io/crates/v/naga-cli.svg?label=naga-cli)](https://crates.io/crates/naga-cli) - Tool for translating shaders between different languages using `naga`.
- [![Crates.io](https://img.shields.io/crates/v/wgpu-info.svg?label=wgpu-info)](https://crates.io/crates/wgpu-info) - Tool for getting information on GPUs in the system.
- `cts_runner` - WebGPU Conformance Test Suite runner using `deno_webgpu`.
- `player` - standalone application for replaying the API traces.

For an overview of all the components in the gfx-rs ecosystem, see [the big picture](./etc/big-picture.png).

## Getting Started

### Play with our Examples

Go to [https://wgpu.rs/examples/] to play with our examples in your browser. Requires a browser supporting WebGPU for the WebGPU examples.

### Rust

Rust examples can be found at [wgpu/examples](examples). You can run the examples on native with `cargo run --bin wgpu-examples <example>`. See the [list of examples](examples).

If you are new to wgpu and graphics programming, we recommend starting with https://sotrh.github.io/learn-wgpu/.

To run the examples in a browser, run `cargo xtask run-wasm`.
Then open `http://localhost:8000` in your browser, and you can choose an example to run.
Naturally, in order to display any of the WebGPU based examples, you need to make sure your browser supports it.

### C/C++

To use wgpu in C/C++, you need [wgpu-native](https://github.com/gfx-rs/wgpu-native).

If you are looking for a wgpu C++ tutorial, look at the following:

- https://eliemichel.github.io/LearnWebGPU/

### Others

If you want to use wgpu in other languages, there are many bindings to wgpu-native from languages such as Python, D, Julia, Kotlin, and more. See [the list](https://github.com/gfx-rs/wgpu-native#bindings).

## Community

We have the Matrix space [![Matrix Space](https://img.shields.io/static/v1?label=Space&message=%23Wgpu&color=blue&logo=matrix)](https://matrix.to/#/#Wgpu:matrix.org) with a few different rooms that form the wgpu community:

- [![Wgpu Matrix](https://img.shields.io/static/v1?label=wgpu-devs&message=%23wgpu&color=blueviolet&logo=matrix)](https://matrix.to/#/#wgpu:matrix.org) - discussion of the wgpu's development.
- [![Naga Matrix](https://img.shields.io/static/v1?label=naga-devs&message=%23naga&color=blueviolet&logo=matrix)](https://matrix.to/#/#naga:matrix.org) - discussion of the naga's development.
- [![User Matrix](https://img.shields.io/static/v1?label=wgpu-users&message=%23wgpu-users&color=blueviolet&logo=matrix)](https://matrix.to/#/#wgpu-users:matrix.org) - discussion of using the library and the surrounding ecosystem.
- [![Random Matrix](https://img.shields.io/static/v1?label=random&message=%23wgpu-random&color=blueviolet&logo=matrix)](https://matrix.to/#/#wgpu-random:matrix.org) - discussion of everything else.

## Wiki

We have a [wiki](https://github.com/gfx-rs/wgpu/wiki) that serves as a knowledge base.

## Extension Specifications

While the core of wgpu is based on the WebGPU standard, we also support extensions that allow for features that the standard does not have yet.
For high-level documentation on how to use these extensions, see the individual specifications:

🧪EXPERIMENTAL🧪 APIs are subject to change and may allow undefined behavior if used incorrectly.

- 🧪EXPERIMENTAL🧪 [Ray Tracing](./etc/specs/ray_tracing.md).

## Supported Platforms

| API    | Windows            | Linux/Android      | macOS/iOS          | Web (wasm)         |
| ------ | ------------------ | ------------------ | ------------------ | ------------------ |
| Vulkan |         ✅         |         ✅         |         🌋         |                    |
| Metal  |                    |                    |         ✅         |                    |
| DX12   |         ✅         |                    |                    |                    |
| OpenGL |    🆗 (GL 3.3+)    |  🆗 (GL ES 3.0+)   |         📐         |    🆗 (WebGL2)     |
| WebGPU |                    |                    |                    |         ✅         |

✅ = First Class Support  
🆗 = Downlevel/Best Effort Support  
📐 = Requires the [ANGLE](#angle) translation layer (GL ES 3.0 only)  
🌋 = Requires the [MoltenVK](https://vulkan.lunarg.com/sdk/home#mac) translation layer  
🛠️ = Unsupported, though open to contributions

### Shader Support

wgpu supports shaders in [WGSL](https://gpuweb.github.io/gpuweb/wgsl/), SPIR-V, and GLSL.
Both [HLSL](https://github.com/Microsoft/DirectXShaderCompiler) and [GLSL](https://github.com/KhronosGroup/glslang)
have compilers to target SPIR-V. All of these shader languages can be used with any backend as we handle all of the conversions. Additionally, support for these shader inputs is not going away.

While WebGPU does not support any shading language other than WGSL, we will automatically convert your
non-WGSL shaders if you're running on WebGPU.

WGSL is always supported by default, but GLSL and SPIR-V need features enabled to compile in support.

Note that the WGSL specification is still under development,
so the [draft specification][wgsl spec] does not exactly describe what `wgpu` supports.
See [below](#tracking-the-webgpu-and-wgsl-draft-specifications) for details.

To enable SPIR-V shaders, enable the `spirv` feature of wgpu.
To enable GLSL shaders, enable the `glsl` feature of wgpu.

### Angle

[Angle](http://angleproject.org) is a translation layer from GLES to other backends developed by Google.
We support running our GLES3 backend over it in order to reach platforms DX11 support, which aren't accessible otherwise.
In order to run with Angle, the "angle" feature has to be enabled, and Angle libraries placed in a location visible to the application.
These binaries can be downloaded from [gfbuild-angle](https://github.com/DileSoft/gfbuild-angle) artifacts, [manual compilation](https://github.com/google/angle/blob/main/doc/DevSetup.md) may be required on Macs with Apple silicon.

On Windows, you generally need to copy them into the working directory, in the same directory as the executable, or somewhere in your path.
On Linux, you can point to them using `LD_LIBRARY_PATH` environment.

### MSRV policy

Due to complex dependants, we have two MSRV policies:

- `naga`, `wgpu-core`, `wgpu-hal`, and `wgpu-types`'s MSRV is **1.76**, but may be lower than the rest of the workspace in the future.
- The rest of the workspace has an MSRV of **1.83** as well right now, but may be higher than above listed crates.

It is enforced on CI (in "/.github/workflows/ci.yml") with the `CORE_MSRV` and `REPO_MSRV` variables.
This version can only be upgraded in breaking releases, though we release a breaking version every three months.

The `naga`, `wgpu-core`, `wgpu-hal`, and `wgpu-types` crates should never
require an MSRV ahead of Firefox's MSRV for nightly builds, as
determined by the value of `MINIMUM_RUST_VERSION` in
[`python/mozboot/mozboot/util.py`][util].

[util]: https://searchfox.org/mozilla-central/source/python/mozboot/mozboot/util.py

## Environment Variables

All testing and example infrastructure share the same set of environment variables that determine which Backend/GPU it will run on.

- `WGPU_ADAPTER_NAME` with a substring of the name of the adapter you want to use (ex. `1080` will match `NVIDIA GeForce 1080ti`).
- `WGPU_BACKEND` with a comma-separated list of the backends you want to use (`vulkan`, `metal`, `dx12`, or `gl`).
- `WGPU_POWER_PREF` with the power preference to choose when a specific adapter name isn't specified (`high`, `low` or `none`)
- `WGPU_DX12_COMPILER` with the DX12 shader compiler you wish to use (`dxc`, `static-dxc`, or `fxc`). Note that `dxc` requires `dxil.dll` and `dxcompiler.dll` to be in the working directory, and `static-dxc` requires the `static-dxc` crate feature to be enabled. Otherwise, it will fall back to `fxc`.
- `WGPU_GLES_MINOR_VERSION` with the minor OpenGL ES 3 version number to request (`0`, `1`, `2` or `automatic`).
- `WGPU_ALLOW_UNDERLYING_NONCOMPLIANT_ADAPTER` with a boolean whether non-compliant drivers are enumerated (`0` for false, `1` for true).

When running the CTS, use the variables `DENO_WEBGPU_ADAPTER_NAME`, `DENO_WEBGPU_BACKEND`, `DENO_WEBGPU_POWER_PREFERENCE`.

## Testing

We have multiple methods of testing, each of which tests different qualities about wgpu. We automatically run our tests on CI. The current state of CI testing:

| Platform/Backend | Tests              | Notes                 |
| ---------------- | ------------------ | --------------------- |
| Windows/DX12     | :heavy_check_mark: | using WARP            |
| Windows/OpenGL   | :heavy_check_mark: | using llvmpipe        |
| MacOS/Metal      | :heavy_check_mark: | using hardware runner |
| Linux/Vulkan     | :heavy_check_mark: | using lavapipe        |
| Linux/OpenGL ES  | :heavy_check_mark: | using llvmpipe        |
| Chrome/WebGL     | :heavy_check_mark: | using swiftshader     |
| Chrome/WebGPU    | :x:                | not set up            |

### Core Test Infrastructure

We use a tool called [`cargo nextest`](https://github.com/nextest-rs/nextest) to run our tests.
To install it, run `cargo install cargo-nextest`.

To run the test suite:

```
cargo xtask test
```

To run the test suite on WebGL (currently incomplete):

```
cd wgpu
wasm-pack test --headless --chrome --no-default-features --features webgl --workspace
```

This will automatically run the tests using a packaged browser. Remove `--headless` to run the tests with whatever browser you wish at `http://localhost:8000`.

If you are a user and want a way to help contribute to wgpu, we always need more help writing test cases.

### WebGPU Conformance Test Suite

WebGPU includes a Conformance Test Suite to validate that implementations are working correctly. We can run this CTS against wgpu.

To run the CTS, first, you need to check it out:

```
git clone https://github.com/gpuweb/cts.git
cd cts
# works in bash and powershell
git checkout $(cat ../cts_runner/revision.txt)
```

To run a given set of tests:

```
# Must be inside the `cts` folder we just checked out, else this will fail
cargo run --manifest-path ../Cargo.toml -p cts_runner --bin cts_runner -- ./tools/run_deno --verbose "<test string>"
```

To find the full list of tests, go to the [online cts viewer](https://gpuweb.github.io/cts/standalone/?runnow=0&worker=0&debug=0&q=webgpu:*).

The list of currently enabled CTS tests can be found [here](./cts_runner/test.lst).

## Tracking the WebGPU and WGSL draft specifications

The `wgpu` crate is meant to be an idiomatic Rust translation of the [WebGPU API][webgpu spec].
That specification, along with its shading language, [WGSL][wgsl spec],
are both still in the "Working Draft" phase,
and while the general outlines are stable,
details change frequently.
Until the specification is stabilized, the `wgpu` crate and the version of WGSL it implements
will likely differ from what is specified,
as the implementation catches up.

Exactly which WGSL features `wgpu` supports depends on how you are using it:

- When running as native code, `wgpu` uses the [Naga][naga] crate
  to translate WGSL code into the shading language of your platform's native GPU API.
  Naga has [a milestone][naga wgsl milestone]
  for catching up to the WGSL specification,
  but in general, there is no up-to-date summary
  of the differences between Naga and the WGSL spec.

- When running in a web browser (by compilation to WebAssembly)
  without the `"webgl"` feature enabled,
  `wgpu` relies on the browser's own WebGPU implementation.
  WGSL shaders are simply passed through to the browser,
  so that determines which WGSL features you can use.

- When running in a web browser with `wgpu`'s `"webgl"` feature enabled,
  `wgpu` uses Naga to translate WGSL programs into GLSL.
  This uses the same version of Naga as if you were running `wgpu` as native code.

[webgpu spec]: https://www.w3.org/TR/webgpu/
[wgsl spec]: https://gpuweb.github.io/gpuweb/wgsl/
[naga]: https://github.com/gfx-rs/naga/
[naga wgsl milestone]: https://github.com/gfx-rs/naga/milestone/4

## Coordinate Systems

wgpu uses the coordinate systems of D3D and Metal:

| Render                                              | Texture                                               |
| --------------------------------------------------- | ----------------------------------------------------- |
| ![render_coordinates](./etc/render_coordinates.png) | ![texture_coordinates](./etc/texture_coordinates.png) |
