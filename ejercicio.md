# Práctica con Servicios AWS: CloudShell, S3, CloudFront, AWS SDK, SNS, Lambda

## Objetivo General
Desarrollar una aplicación en React que permita subir archivos a AWS S3, y notificar mediante SNS y Lambda, usando un entorno local en Visual Studio Code. Se aprenderán conceptos básicos de cada servicio y cómo interactúan entre sí.

---

## Paso 1: Configuración del Entorno de Desarrollo


### 1.2. Crear una Carpeta de Proyecto
1. Abre Visual Studio Code.
2. Crea una nueva carpeta en tu disco (por ejemplo, `aws-practica`).
3. Abre la carpeta en VS Code usando _File > Open Folder_.

---

## Paso 2: Crear y Configurar la Aplicación React

### 2.1. Inicializar la Aplicación
1. Abre una terminal integrada en VS Code (_View > Terminal_).
2. Ejecuta el siguiente comando para crear una nueva aplicación React:
   ```sh
   npx create-react-app mi-aplicacion
   ```
   - Este comando descargará y configurará la estructura básica de una aplicación React.
   - La carpeta `mi-aplicacion` se creará dentro de tu carpeta de proyecto.

3. Cambia al directorio de la aplicación:
   ```sh
   cd mi-aplicacion
   ```

### 2.2. Ejecutar la Aplicación en Modo Desarrollo
1. En la terminal, ejecuta:
   ```sh
   npm start
   ```
   - Esto inicia el servidor de desarrollo.
   - Se abrirá una ventana del navegador con la aplicación React corriendo (usualmente en [http://localhost:3000](http://localhost:3000)).
   - Si realizas cambios en el código, el navegador se actualizará automáticamente.

### 2.3. Instalación de Dependencias Adicionales
1. Con la terminal abierta en el directorio del proyecto, instala el AWS SDK para S3:
   ```sh
   npm install @aws-sdk/client-s3
   ```
2. Instala la librería `web-vitals` (usada para medir el rendimiento, aunque es opcional):
   ```sh
   npm install --save-dev web-vitals
   ```

### 2.4. Modificar el Archivo `App.js`
1. En VS Code, navega a la carpeta `mi-aplicacion/src/`.
2. Abre el archivo `App.js` y reemplázalo con el siguiente código. Este código permite seleccionar un archivo desde el navegador y subirlo a un bucket de S3 utilizando el AWS SDK:

```
import { useState } from "react";
import "./App.css";
import { Upload } from "@aws-sdk/lib-storage";
import { S3Client, S3 } from "@aws-sdk/client-s3";

function App() {
  const [selectFile, setSelectFile] = useState(null);
  const [uploading, setUploading] = useState(false);
  const [uploadMessage, setUploadMessage] = useState("");

  const handleFileInput = (e) => {
    // Extract the first file from the FileList
    const file = e.target.files[0];
    console.log("Nombre del archivo:", file.name);
    setSelectFile(file);
  };

  const handleUpload = async () => {
    if (!selectFile) {
      setUploadMessage("Please select a file.");
      return;
    }
    

    setUploading(true);
    setUploadMessage("Uploading...");

    try {
   
      const fileStream = selectFile.stream ? selectFile.stream() : selectFile;
      const params = {
        Bucket: "osmefru",
        Key: selectFile.name,
        Body: fileStream,
      };
      const parallelUploads3 = new Upload({
        client:  new S3Client({
          region: "us-east-1",
          credentials: {
            accessKeyId: "XXXXXXXXXXXX",
            secretAccessKey: "XXXXXXXXXXXXXX",
            sessionToken: "XXXXXXXXXXX"
          },
        }),
        params: params,
      });
  
      parallelUploads3.on("httpUploadProgress", (progress) => {
        console.log(progress);
      });
  
      await parallelUploads3.done();



      setUploadMessage("Upload successful!");
    } catch (error) {
      console.error("Error uploading file:", error);
      setUploadMessage("Error uploading file. Please try again.");
    } finally {
      setUploading(false);
    }
  };

  return (
    <div className="app-container">
      <h1>File Uploader</h1>
      <input type="file" onChange={handleFileInput} multiple></input>
      <button onClick={handleUpload} disabled={uploading}>
        {uploading ? "Uploading..." : "Upload To S3"}
      </button>
      {uploadMessage && <p className={uploading ? "uploading-message" : "upload-message"}>{uploadMessage}</p>}
    </div>
  );
}

export default App;
```


 - Las credenciales y valores en estas lineas 
            accessKeyId: "XXXXXXXXX",
             secretAccessKey: "XXXXXXXXX",
             sessionToken: "XXXXXXXXX",

    Debeis obtenerlas del laboratorio en AWS.
 
### 2.5. Modificar el Archivo `App.css`
1. En la misma carpeta `src/`, abre el archivo `App.css`.
2. Reemplaza el contenido con el siguiente código para estilizar la aplicación:

   ```css
   .app-container {
     text-align: center;
     margin: 50px;
   }

   input {
     margin-bottom: 20px;
   }

   button {
     padding: 10px;
     cursor: pointer;
     background-color: #4caf50;
     color: white;
     border: none;
   }

   button:disabled {
     background-color: #cccccc;
     cursor: not-allowed;
   }

   .upload-message {
     color: #4caf50;
   }

   .uploading-message {
     color: #007bff;
   }
   ```

> **Nota:** Las credenciales se definen en el código del SDK de JavaScript. En un entorno de producción, es recomendable gestionarlas de forma segura (por ejemplo, utilizando variables de entorno o servicios de gestión de secretos).

---

## Paso 3: Configuración de AWS SNS

### 3.1. Crear un Tema en SNS
1. Inicia sesión en la consola de AWS.
2. Ve al servicio **Simple Notification Service (SNS)**.
3. Haz clic en **Create topic** (Crear tema).
4. Selecciona el tipo de tema (por ejemplo, Standard).
5. Asigna un nombre al tema y finaliza la creación.

### 3.2. Suscribir un Correo Electrónico
1. Una vez creado el tema, abre sus detalles.
2. Selecciona la opción **Create subscription** (Crear suscripción).
3. En el campo de protocolo, elige **Email**.
4. Introduce tu dirección de correo (o la que se indique, por ejemplo, la de Tajamar).
5. Haz clic en **Create subscription**.
6. Revisa tu correo y confirma la suscripción haciendo clic en el enlace que recibas.

---

## Paso 4: Configuración de AWS Lambda

### 4.1. Crear una Función Lambda
1. En la consola de AWS, ve a **Lambda**.
2. Haz clic en **Create function**.
3. Selecciona **Author from scratch**.
4. Asigna un nombre a la función (por ejemplo, `ProcesarSubidaS3`).
5. Selecciona **Python 3.12** como runtime.
6. En la sección de permisos, selecciona un rol existente (por ejemplo, `LabRol`) o crea uno nuevo, pero asegúrate de que tenga permisos para publicar en SNS.

### 4.2. Agregar el Código a la Función
1. En el editor de la consola Lambda, reemplaza el código en el archivo principal (usualmente `lambda_function.py`) con el siguiente:

   ```python
   import json
   import boto3

   def lambda_handler(event, context):
       # Obtener la información del evento de S3
       s3_event = event['Records'][0]['s3']
       bucket_name = s3_event['bucket']['name']
       object_key = s3_event['object']['key']
       object_size = s3_event['object']['size']

       # Crear un cliente de SNS
       sns_client = boto3.client('sns')
       # Reemplaza 'arn:aws:sns:us-east-1:XXXXXXXX:tu-tema' con el ARN real de tu tema SNS
       topic_arn = 'arn:aws:sns:us-east-1:XXXXXXXX:tu-tema'

       # Crear el mensaje que se enviará por correo
       sns_message = f'Se insertó un nuevo archivo en S3: {object_key}, Tamaño: {object_size} bytes.'

       # Publicar el mensaje en el tema SNS
       sns_client.publish(TopicArn=topic_arn, Message=sns_message)

       return {
           'statusCode': 200,
           'body': json.dumps('Mensaje enviado correctamente.')
       }
   ```

2. Ajusta el ARN del tema SNS en el código por el que corresponda en tu cuenta.

### 4.3. Configurar el Tiempo de Ejecución
1. En la configuración de la función, ajusta el **Timeout** a 1 minuto (60 segundos) para dar tiempo suficiente a la ejecución de la Lambda.

---

## Paso 5: Configuración de AWS S3

### 5.1. Crear un Bucket S3
1. Accede a la consola de AWS y ve al servicio **S3**.
2. Haz clic en **Create bucket**.
3. Asigna un nombre único al bucket (por ejemplo, `mi-bucket-archivo`).
4. Selecciona la región `us-east-1` (o la que corresponda) y continúa.
5. En la sección de **Block Public Access**, desactiva la opción que impida el acceso público (si se requiere que los archivos sean accesibles públicamente).  
   > **Atención:** Activar acceso público tiene implicaciones de seguridad. Hazlo solo en entornos de prueba.

### 5.2. Configurar Permisos del Bucket
1. Una vez creado el bucket, ve a la pestaña **Permissions**.
2. En **Bucket policy**, añade la siguiente política (reemplaza `YOUR_BUCKET_NAME` por el nombre real del bucket):

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": ["arn:aws:s3:::YOUR_BUCKET_NAME/*", "arn:aws:s3:::YOUR_BUCKET_NAME"]
       }
     ]
   }
   ```

### 5.3. Configurar CORS (Cross-Origin Resource Sharing)
1. En la misma pestaña de **Permissions**, busca la sección **CORS configuration**.
2. Añade la siguiente configuración para permitir distintos métodos y orígenes:

   ```json
   [
     {
       "AllowedHeaders": ["*"],
       "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
       "AllowedOrigins": ["*"],
       "ExposeHeaders": ["ETag"]
     }
   ]
   ```

### 5.4. Configurar Notificaciones de Eventos para Lambda
1. Ve a la pestaña **Properties** del bucket S3.
2. Busca la sección **Event notifications**.
3. Configura una nueva notificación que invoque la función Lambda cuando se suba un nuevo objeto:
   - Selecciona el evento `PUT` o `All object create events`.
   - Elige la función Lambda creada en el Paso 4.
   - Guarda la configuración.

---

## Paso 6: Pruebas Finales y Verificación

### 6.1. Ejecutar la Aplicación
1. En VS Code, abre la terminal en la carpeta del proyecto `mi-aplicacion`.
2. Ejecuta:
   ```sh
   npm start
   ```
3. La aplicación se abrirá en el navegador en [http://localhost:3000](http://localhost:3000).

### 6.2. Subir un Archivo y Verificar
1. En la interfaz de la aplicación, utiliza el botón para seleccionar un archivo.
2. Haz clic en el botón **Subir a S3**.
3. Verifica lo siguiente:
   - **En S3:** Ingresa a la consola de S3 y revisa que el archivo se haya subido correctamente al bucket.
   - **En SNS:** Revisa el correo electrónico para confirmar que recibiste la notificación.
   - **En AWS Lambda:** Si necesitas depurar, revisa los logs en AWS CloudWatch para ver la ejecución de la función Lambda.

### 6.3. Depuración y Solución de Problemas
- **Error en la consola del navegador:**  
  Revisa los mensajes de error en la consola (F12) para identificar problemas de conexión o errores en el código.
- **Problemas con las credenciales en el SDK:**  
  Asegúrate de que los valores de `accessKeyId`, `secretAccessKey` y `sessionToken` en `App.js` sean correctos. Recuerda que en un entorno real se deben manejar de forma segura.
- **Lambda sin ejecución:**  
  Verifica en AWS CloudWatch los logs de Lambda para detectar errores o configuraciones incorrectas.
