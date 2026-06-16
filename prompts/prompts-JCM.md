# Prompts Iniciales y Diario de Aprendizaje - Pipeline CI/CD

Esta vez no he usado simples prompts generativos para que una IA me escupa el código. He mantenido una sesión de programación en pareja interactiva y educativa con mi asistente. Este ha sido el flujo real de aprendizaje y construcción de nuestro pipeline:

### 1. Pruebas y Aislamiento en GitHub Actions
**Mi solicitud:** "Creemos el pipeline paso a paso en `.github/workflows/ci.yml`. Podríamos comenzar con un smoke test de backend."
**Lo que aprendimos:** Al añadir luego el paso de generación de *build*, me di cuenta de que Actions descargaba todo y hacía checkout e instalación de nuevo. Discutimos cómo GitHub Actions aísla cada *Job* en una máquina virtual totalmente limpia para evitar conflictos, lo cual gasta más tiempo pero aporta seguridad y posibilidad de paralelismo. Elegimos mantenerlos separados por limpieza en este ejercicio.

### 2. Buenas Prácticas de Seguridad en AWS (El tirón de orejas)
**Mi solicitud:** "Respecto a AWS, tenemos que ir más atrás, porque sólo me cree la cuenta y sólo tengo un usuario root. Entiendo lo correcto sería crear una IAM user. No te adelantes ni asumas cosas."
**Lo que aprendimos:** Le di un tirón de orejas al asistente por apresurarse. Aseguramos el entorno creando un usuario IAM con políticas de *AdministratorAccess* para no comprometer el usuario Root de AWS. Entendimos la importancia del principio de menor privilegio, incluso en ejercicios de prueba.

### 3. El Misterio de las Claves SSH
**Mi solicitud:** "Tengo una duda, ¿por qué la clave es privada? ¿No se supone que si fuera una ssh key, la llave que se comparte es la pública?"
**Lo que aprendimos:** Clarificamos la criptografía asimétrica. AWS crea la Llave Pública (la cerradura) y la instala en la máquina EC2. Nosotros descargamos la Llave Privada `.pem` (la llave física). Como GitHub Actions actúa en nuestro nombre para conectarse al EC2, necesita que le pasemos la Llave Privada a través de sus Secretos para poder "abrir" la cerradura del servidor.

### 4. Debugging en Vivo (Prisma y Variables de Entorno)
**El problema:** El pipeline dio "verde", la regla de seguridad del puerto 3010 estaba abierta, pero el navegador daba `ERR_CONNECTION_REFUSED`.
**La resolución:** El asistente se conectó a los logs de PM2 y descubrió que la aplicación se estrellaba al instante porque dependía de Prisma y faltaba tanto la carpeta de Prisma como las variables de entorno (`.env`). Modificamos el paso de subida al EC2 para incluir esos archivos e incluimos el comando `npx prisma generate` en el script de despliegue. ¡Por fin vimos "Hola LTI!" en pantalla!

### 5. Pipeline Robusto (Health Check Real)
**Mi solicitud:** "El pipeline debería tener un paso más robusto para revisar porque antes salió todo verde, pero en realidad la app había fallado al correr en PM2."
**La mejora final:** `pm2 start` engañaba al pipeline devolviendo "éxito" aunque la app fallase 1 segundo después. Le exigí al asistente añadir un paso de comprobación al script. Añadimos un pequeño `sleep 5` seguido de un `curl http://localhost:3010`. De esta forma, si el endpoint no responde desde dentro de la propia máquina, forzamos un fallo en el pipeline (`exit 1`) y escupimos los logs de error en la consola de GitHub Actions para evitar despliegues silenciosamente rotos.
