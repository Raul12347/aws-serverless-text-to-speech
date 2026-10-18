# Conversor e Integración Serverless de Texto/Audio con AWS

##  Descripción del Proyecto
Este proyecto implementa una arquitectura completamente serverless impulsada por eventos para la conversión automática de archivos utilizando servicios de IA de AWS. Al cargar un archivo fuente en un bucket de entrada de Amazon S3, el sistema desencadena automáticamente una función AWS Lambda que procesa la información (por ejemplo, síntesis de texto a voz mediante Amazon Polly), distribuye notificaciones o eventos con Amazon SNS / EventBridge y almacena el resultado procesado en un bucket S3 de destino.

---

## Arquitectura y Tecnologías
- **Proveedor Cloud:** Amazon Web Services (AWS)
- **Servicios AWS:** Amazon S3, AWS Lambda, Amazon Polly, Amazon EventBridge, Amazon SNS, AWS IAM.

### Diagrama de Arquitectura
![Diagrama de Arquitectura](./docs/Arquitectura.drawio.png)


## Diseño de la Solución
Explicación detallada del flujo de datos y cómo interactúan los componentes entre sí:
1. Carga del archivo (Evento trigger):El usuario o sistema sube o craga un archivo a un bucket de entrada en **Amazon S3**.
2. Invocacion del Serverless: La carga del objeto genere un evento de S3, que desencadena una funcion en **AWS Lambda**.
3. Sintesis de audio: La funcion Lambda lee el contenido del archivo del texto y realiza la llamada de **Amazon Polly** para convertir el texto de un archivo de audio. 
4. Almacenamiento del resultado:Lambda guarda el archivo del audio y en el bucket de **Amazon S3**. 
5. **Notificación del evento:** Se emite una notificación mediante **Amazon SNS / EventBridge** informando sobre la finalización exitosa del procesamiento.



## Pasos para Replicar el Despliegue

### Requisitos Previos
- Cuenta activa de AWS (Free Tier).
- Permisos IAM necesarios para la creación de recursos.
# Step 1: Crear los Buckets de Amazon S3
1. Ve a la consola de Amazon S3 y haz clic en Crear bucket.
2. Bucket de Entrada:
- Nombre: polly-input-text-bucket (debe ser un nombre único a nivel mundial).
- Región: Selecciona tu región preferida (ej. us-east-1).
- Configuración: Mantén las opciones por defecto (Bloqueo de acceso público activado).
3. Bucket de Salida:
- Nombre: polly-output-audio-bucket.
- Mantén las opciones por defecto y crea el bucket.
# Step 2: Crear el Rol de IAM para Lambda
1. Ve al servicio de IAM > Roles y haz clic en Crear rol.
2. Selecciona Servicio de AWS como tipo de entidad de confianza y elige Lambda.
3. Asigna las siguientes políticas de permisos:
- AWSLambdaBasicExecutionRole (para guardar logs en CloudWatch).
- AmazonPollyFullAccess (o crear una política en línea con la acción polly:SynthesizeSpeech).
- Política personalizada para S3: Agrega permisos para s3:GetObject en el bucket de entrada y s3:PutObject en el bucket de salida.
4. Nombra el rol como LambdaPollyS3ExecutionRole y haz clic en Crear rol.

# Step 3: Crear y Configurar la Función AWS Lambda
1. Dirígete a la consola de AWS Lambda y selecciona Crear una función.
2. Selecciona Autor desde cero:
- Nombre de la función: TextToAudioPollyFunction.
- Runtime: Python 3.12 (o superior).
- Permisos: Elige Usar un rol existente y selecciona LambdaPollyS3ExecutionRole.
3. Haz clic en Crear función.
4. En el editor de código de Lambda, copia y pega el código Python ubicado en src/lambda_function.py.
5. Asegúrate de actualizar en el código las variables de entorno o nombres exactos de tu bucket de salida.
6. Haz clic en Deploy.

# Step 4: Configurar el Trigger de Amazon S3
1. Dentro de tu función Lambda TextToAudioPollyFunction, ve a la pestaña Configuración > Desencadenadores (Triggers).
2. Haz clic en Agregar desencadenador.
3. Selecciona S3 de la lista:
- Bucket: Selecciona polly-input-text-bucket.
- Tipo de evento: Todos los eventos de creación de objetos (s3:ObjectCreated:*).
- Sufijo (Opcional): .txt (para evitar procesar otros formatos).
4. Confirma la casilla de verificación y haz clic en Agregar.

# Step 5: (Opcional) Configurar Notificaciones con EventBridge / SNS
1. Ve a Amazon SNS y crea un Tema (Topic) de tipo Estándar llamado AudioProcessingNotification.

Crea una Suscripción en el tema seleccionando el protocolo Correo electrónico e ingresa tu e-mail para confirmar la suscripción.

En la función Lambda, agrega la llamada a Boto3 de SNS (sns_client.publish()) al finalizar la generación exitosa del archivo de audio en S3.

# Step 6: Prueba de Funcionamiento
Abre el bucket de entrada polly-input-text-bucket en Amazon S3.

Sube un archivo de texto de prueba llamado prueba.txt que contenga un mensaje corto.

Ve al bucket de salida polly-output-audio-bucket y verifica que se haya generado el archivo .mp3.

(Opcional) Revisa Amazon CloudWatch Logs en el grupo de logs de la función Lambda para auditoría o depuración.

### Configuración e Instalación
1. Clonar este repositorio:
   ```bash
   git clone https://github.com/Raul12347/aws-serverless-text-to-speech.git