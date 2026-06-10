MCP-SETUP — Instalación y solución de problemas para `@arabold/docs-mcp-server` (Windows)

Resumen

Guía paso a paso para instalar y ejecutar correctamente el Docs MCP Server en Windows (sin Docker). Incluye prerequisitos (Node, Python, Visual Studio Build Tools), comandos útiles de prueba y soluciones para el error típico: falta de Visual Studio Build Tools y problemas de conexión SSE (fetch failed / AggregateError).

1) Requisitos previos

- Windows 10/11 actualizado
- Node.js 22+ (recomendado). Verificar con:

```powershell
node -v
npm -v
```

- Git (opcional pero recomendado):

```powershell
git --version
```

- Python 3.x (necesario para compilación nativa con node-gyp en algunas dependencias). Instálalo desde https://www.python.org/ o Microsoft Store y asegúrate de marcar "Add to PATH".

- Visual Studio Build Tools (C++): necesario para compilar paquetes que usan node-gyp. Puedes instalar la herramienta desde Microsoft:

  - Descarga el instalador: https://aka.ms/vs/17/release/vs_BuildTools.exe
  - Ejecuta el instalador y selecciona la carga de trabajo "Desarrollo para escritorio con C++" (o instala desde línea de comandos):

```powershell
# Ejecutar como administrador (ejemplo de instalación silenciosa)
.\vs_BuildTools.exe --quiet --wait --norestart --add Microsoft.VisualStudio.Workload.VCTools --add Microsoft.VisualStudio.Component.VC.Tools.x86.x64 --add Microsoft.VisualStudio.Component.Windows10SDK.19041
```

Nota: si no quieres la instalación silenciosa, ejecútalo y en la GUI marca "Desktop development with C++" y el Windows SDK.

2) (Opcional) Configurar Python para npm/node-gyp

Si node-gyp requiere una versión específica de Python, configura npm:

```powershell
npm config set python python
```

(Reinicia la terminal después de instalar Visual Studio Build Tools / Python.)

3) Evitar instalaciones globales que requieran compilación (recomendado)

Para evitar errores con módulos nativos al instalar globalmente, usa `npx` para ejecutar el servidor sin instalar globalmente:

```powershell
# Ejecutar sin instalar globalmente
npx @arabold/docs-mcp-server --host 127.0.0.1 --port 6280
```

Si prefieres instalar globalmente (puede fallar sin build tools):

```powershell
npm i -g @arabold/docs-mcp-server
```

Si la instalación falla con errores de `node-gyp` o relacionado con MSBuild, instala Visual Studio Build Tools y Python como se describe arriba.

4) Ejecutar el servidor (arranque básico)

```powershell
# Recomendado (evita problemas IPv6/localhost):
npx @arabold/docs-mcp-server --host 127.0.0.1 --port 6280

# Opcional: activar salida detallada para depurar
npx @arabold/docs-mcp-server --host 127.0.0.1 --port 6280 --verbose
```

Salida esperada (ejemplo):

- Web UI: http://127.0.0.1:6280
- MCP endpoints: http://127.0.0.1:6280/mcp, http://127.0.0.1:6280/sse

5) Abrir puerto en el firewall (si necesitas acceso remoto)

Si Windows Firewall bloquea el puerto, abre una regla:

```powershell
# Ejecutar PowerShell como administrador
New-NetFirewallRule -DisplayName "Grounded Docs MCP 6280" -Direction Inbound -LocalPort 6280 -Protocol TCP -Action Allow
```

6) Probar que el servidor está escuchando y que SSE responde

Comandos útiles (PowerShell):

```powershell
# Comprobar que el puerto está escuchando
Test-NetConnection -ComputerName 127.0.0.1 -Port 6280

# Mostrar procesos que usan el puerto
netstat -ano | findstr :6280

# Probar SSE (usar curl.exe para evitar alias de PowerShell)
curl.exe -i -N http://127.0.0.1:6280/sse
```

Resultados esperados:
- `Test-NetConnection` -> `TcpTestSucceeded: true`
- `netstat` muestra LISTENING en 127.0.0.1:6280
- `curl.exe` devuelve `Content-Type: text/event-stream` y eventos SSE

7) Configurar VS Code MCP (ejemplo de `mcp.json` o ajustes)

Para que VS Code o clientes MCP se conecten, usa la IP explícita `127.0.0.1` (evita ambigüedad IPv6):

```jsonc
{
  "servers": {
    "docs-mcp-server": {
      "type": "sse",
      "url": "http://127.0.0.1:6280/sse",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Si no quieres que VS Code intente conectarse temporalmente, fija `"disabled": true`.

8) Errores comunes y cómo resolverlos

A) Error: "fetch failed: AggregateError" o "Error sending message to http://localhost:6280/sse"
- Causa: el cliente intentó conectarse cuando no había servidor escuchando, o `localhost` resolvió a ::1 y el servidor sólo está escuchando en 127.0.0.1.
- Solución:
  - Asegúrate de arrancar el servidor antes del cliente.
  - Usa `--host 127.0.0.1` al arrancar o actualiza la configuración del cliente para usar `127.0.0.1`.
  - Comprueba firewall/antivirus.

B) Error en `npm install` o `npm i -g @arabold/docs-mcp-server` indicando falta de MSBuild/node-gyp o "gyp ERR!"
- Causa: faltan Visual Studio Build Tools y/o Python en PATH.
- Solución:
  1. Instalar Visual Studio Build Tools con soporte C++ (ver paso 1).
  2. Instalar Python 3.x y asegurarte de que `python` está en PATH.
  3. Reinicia la terminal y reintenta la instalación.

C) Playwright / navegación headless fallando
- Grounded Docs usa Playwright para scrapear páginas que necesitan JS. En el primer arranque puede descargarse Chromium; asegúrate de tener conexión a internet y espacio en disco.
- Si falla la instalación automática de Playwright, puedes preinstalar los navegadores:

```powershell
npx playwright install chromium
```

D) Problemas IPv6 vs IPv4
- `localhost` puede resolverse a `::1` (IPv6). Si el servidor escucha sólo en IPv4, la conexión desde clientes que usan `::1` fallará. Para evitar esto, en la configuración del cliente usa `127.0.0.1` o arranca el servidor con `--host ::` (si quieres IPv6).

9) Comandos de depuración y logs a recopilar

Si necesitas pedir ayuda, recopila y comparte:

- Salida completa del comando de arranque (terminal donde ejecutaste `npx @arabold/docs-mcp-server ...`).
- Salida de error de `npm` si la instalación falla (copia de la consola con errores `gyp ERR!` y `npm-debug.log` si existe).
- Resultado de:

```powershell
node -v
npm -v
python --version
netstat -ano | findstr :6280
Test-NetConnection -ComputerName 127.0.0.1 -Port 6280
```

10) Checklist rápido para otra laptop (hacer en este orden)

- [ ] Instalar Node.js 22+ (y reiniciar la sesión).
- [ ] Instalar Git.
- [ ] Instalar Python 3.x y añadir `python` al PATH.
- [ ] Instalar Visual Studio Build Tools (Desktop C++ workload).
- [ ] Abrir una nueva terminal (recrear PATH).
- [ ] Ejecutar `npx @arabold/docs-mcp-server --host 127.0.0.1 --port 6280`.
- [ ] Probar con `curl.exe -i -N http://127.0.0.1:6280/sse`.

11) Si todo falla: alternativa temporal

- Usa `npx @arabold/docs-mcp-server` en la máquina donde funciona y copia el `mcp.json` con `url` apuntando a esa máquina (requiere abrir el puerto y red adecuada). Esto es sólo un parche; lo correcto es instalar en la máquina objetivo.

---

Si quieres, puedo:
- Añadir pasos exactos para la instalación silenciosa de Visual Studio Build Tools adaptados a tu versión de Windows.
- Generar un script PowerShell que verifique los prerequisitos y haga pruebas básicas (comprobación de versiones y puertos).

Dime si quieres que genere el script de verificación automático y lo añada al repo (PowerShell) y si prefieres que especifique versiones concretas (p. ej. Python 3.11).