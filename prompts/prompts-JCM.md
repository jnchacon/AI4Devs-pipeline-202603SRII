# Prompts Iniciales - Pipeline CI/CD

## Paso 1: Tests de backend
**Prompt utilizado:**
> "Creemos el pipeline paso a paso en `.github/workflows/ci.yml`. Configura el workflow de GitHub Actions para que se dispare con un push a una rama con un Pull Request abierto. Comenzaremos con un job para ejecutar los tests de backend (un smoke test) usando Node.js 20, instalando las dependencias y ejecutando `npm run test` en la carpeta `backend`."
