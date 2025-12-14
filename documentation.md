# Documentación técnica y funcional

## Descripción general
Este proyecto implementa un flujo experimental para automatizar el procesamiento de correos electrónicos recibidos por una compañía de seguros. Se exploran capacidades de un LLM (Claude 3) para:
- Extraer entidades y atributos relevantes sin fine-tuning mediante ingeniería de prompts inspirada en RAG.
- Analizar el sentimiento de cada correo (neutro/positivo/negativo) y sugerir respuestas automatizadas de atención al cliente.
- Construir estructuras JSON compatibles con una supuesta capa de APIs internas para ejecutar acciones sobre pólizas, clientes y coberturas.

### Alcance
- **Objetivo**: demostrar cómo un LLM puede acelerar la normalización de datos provenientes de texto libre en el dominio de seguros.
- **No incluido**: persistencia real en bases de datos, autenticación contra APIs corporativas ni despliegue productivo; estos pasos se simulan para mantener el ejemplo autocontenido.

## Componentes principales
- **Notebook `TP_MineriaTextos_DiegoTarrio_2024.ipynb`**: concentra el código ejecutable y la explicación original del trabajo práctico. Incluye los bloques de instalación de dependencias, definición de clases y rutinas de ejemplo.
- **Clase `RAGEntityMapper`**: encapsula la integración con el modelo Claude y el mapeo de entidades/atributos del dominio asegurador.
  - `call_claude_api(prompt)`: invoca el endpoint `messages.create` de Claude 3 Opus y devuelve el texto de respuesta.
  - `extract_entities(email_text)`: construye un prompt con el esquema de entidades (Póliza, Cliente y Cobertura), solicita extracción estructurada más análisis de sentimiento y parsea el JSON retornado.
  - `prepare_api_payload(extracted_data)`: filtra valores nulos y arma un payload con atributos presentes para cada entidad detectada.
  - `invoke_apis(payload)`: simula la invocación a APIs del core de la compañía, mostrando en consola las entidades/atributos listos para ser enviados.
- **Funciones utilitarias**:
  - `generar_emails_ejemplo(archivo_salida)`: crea un archivo de correos de prueba separados por el marcador `---SEPARADOR---`.
  - `procesar_emails_desde_archivo(extractor, archivo_entradas)`: lee cada correo, ejecuta extracción, prepara el payload y almacena resultados para su posterior análisis.
  - `main()`: prepara la API key, genera datos de ejemplo, procesa todos los correos y escribe `resultados_procesamiento.json` con las salidas consolidadas.

### Estructura de archivos
- `documentation.md`: este documento.
- `TP_MineriaTextos_DiegoTarrio_2024.ipynb`: notebook con el pipeline completo y celdas de configuración.
- `README.md`: instrucciones del trabajo práctico original y contexto académico.
En ejecución, se generan además `emails_ejemplos.txt` y `resultados_procesamiento.json`.

## Flujo de ejecución recomendado
1. **Preparar dependencias**: instalar `anthropic` y `python-dotenv` (las primeras celdas del notebook incluyen los comandos `pip`).
2. **Configurar credenciales**: crear un archivo `.env` en el directorio raíz con la variable `ANTHROPIC_API_KEY=<clave>` para habilitar el cliente de Claude.
3. **Ejecutar el notebook** (o convertirlo a script) y correr `main()`:
   - Se generan automáticamente correos de prueba en `emails_ejemplos.txt` si no existen.
   - Cada correo se envía al LLM con el prompt estructurado; la respuesta se parsea como JSON y se limpia vía `prepare_api_payload`.
   - El payload final se imprime (simulando la llamada a APIs internas) y se guarda junto al análisis de sentimiento en `resultados_procesamiento.json`.
4. **Revisión de salidas**: los archivos `emails_ejemplos.txt` y `resultados_procesamiento.json` permiten verificar tanto la extracción de datos como las respuestas sugeridas.
5. **Extensión opcional para pruebas**: modificar `generar_emails_ejemplo` con casos límite (faltan datos de póliza, tono agresivo, múltiples coberturas) para validar la robustez del parseo y del análisis de sentimientos.

## Entradas y salidas
- **Entrada principal**: texto libre de correos electrónicos, ya sea generados por `generar_emails_ejemplo` o provistos en `emails_ejemplos.txt`.
- **Salida estructurada**: objeto JSON con:
  - `Entidades`: diccionario por entidad (`Poliza`, `Cliente`, `Cobertura`) con atributos completados o `null`.
  - `Sentimiento`: tono detectado, motivo de reclamo (si aplica) y respuesta sugerida para el cliente.
- **Payload simulado**: diccionario con solo atributos no nulos, base para invocar APIs reales.

## Consideraciones técnicas
- El modelo LLM está fijado a `claude-3-opus-20240229`, pero el diseño facilita encapsular otro proveedor si se reemplaza la inicialización de `RAGEntityMapper`.
- La extracción depende de instrucciones en el prompt; no hay vector DB ni embeddings. Se podría extender a un RAG completo añadiendo recuperación basada en embeddings o fine-tuning especializado.
- El manejo de errores es básico: si el JSON devuelto no se puede parsear, se retorna un objeto vacío y el flujo continúa.
- El script imprime la API key cargada solo para depuración; en entornos reales debe evitarse mostrar credenciales en consola o logs.
- Para reproducibilidad, es recomendable fijar la versión del cliente de Anthropics en `requirements.txt` o `pip install anthropic==<versión probada>` dentro del notebook.

## Requisitos previos
- Python 3.x con acceso a internet para invocar el endpoint de Anthropics.
- Paquetes: `anthropic`, `python-dotenv` y dependencias estándar (`json`, `os`, `pprint`).
- Archivo `.env` con `ANTHROPIC_API_KEY`. Sin esta variable, las llamadas al modelo fallarán y la extracción devolverá JSON vacío.
- Cuenta en Anthropics con límites suficientes para llamadas de prueba (el notebook envía una petición por correo procesado).

## Próximos pasos sugeridos
- Implementar un cliente de APIs reales para persistir `Cliente`, `Poliza` y `Cobertura` en lugar de solo imprimir el payload.
- Incorporar almacenamiento vectorial para normalizar atributos como coberturas y tipos de pólizas mediante búsqueda semántica.
- Generalizar la clase de modelo para soportar proveedores alternativos (por ejemplo, OpenAI GPT) mediante un wrapper configurable.
- Robustecer el manejo de errores y validaciones de esquema antes de enviar datos a servicios externos.
- Añadir pruebas unitarias que mockeen la respuesta del LLM para validar el parseo de JSON y la construcción del payload sin depender de la API.
