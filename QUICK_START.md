# 🚀 Guía de Instalación Rápida: CLIProxyAPI para Gavazia-Brain

## 📋 Resumen

CLIProxyAPI te permite usar modelos de IA premium (Gemini, Claude, GPT-5) **gratis o a bajo costo** a través de sus interfaces CLI/OAuth, convirtiéndolas en APIs compatibles con OpenAI.

---

## ⚡ Opción 1: Instalación Rápida (Recomendada)

### Paso 1: Instalar Go

#### Windows (usando Chocolatey)

```powershell
# Si no tienes Chocolatey, instalarlo primero:
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Instalar Go
choco install golang -y

# Verificar instalación
go version
```

#### Windows (Instalador Manual)

1. Descargar Go desde: <https://go.dev/dl/>
2. Ejecutar el instalador `.msi`
3. Reiniciar PowerShell
4. Verificar: `go version`

---

### Paso 2: Compilar CLIProxyAPI

```powershell
# Navegar al directorio
cd "c:\Users\Andy Gavaz\Documents\Antigravity\Gavazia-Brain\CLIProxyAPI"

# Compilar
go build -o cliproxyapi.exe ./cmd/cli-proxy-api

# Verificar que se creó el ejecutable
ls cliproxyapi.exe
```

---

### Paso 3: Configuración Inicial

```powershell
# Copiar archivos de configuración
Copy-Item config.example.yaml config.yaml
Copy-Item .env.example .env

# Editar config.yaml con tu editor favorito
notepad config.yaml
```

#### Configuración Mínima en `config.yaml`

```yaml
# Solo escuchar en localhost por seguridad
host: "127.0.0.1"
port: 8317

# Tu API key para autenticación
api-keys:
  - "gavazia-brain-master-key-2025"

# Directorio de autenticación
auth-dir: "~/.cli-proxy-api"

# Habilitar debug
debug: true
```

---

### Paso 4: Obtener Gemini API Key (Gratis)

1. **Ir a Google AI Studio**: <https://aistudio.google.com/apikey>
2. **Crear nueva API key**
3. **Copiar la key** (empieza con `AIzaSy...`)
4. **Agregar a `config.yaml`**:

```yaml
# Descomentar y agregar tu key
gemini-api-key:
  - api-key: "AIzaSy...TU_KEY_AQUI"
    prefix: "gemini"
```

---

### Paso 5: Ejecutar el Proxy

```powershell
# Ejecutar CLIProxyAPI
.\cliproxyapi.exe

# Deberías ver:
# [INFO] Starting CLI Proxy API server on 127.0.0.1:8317
```

---

### Paso 6: Probar la Instalación

#### En otra terminal PowerShell

```powershell
# Listar modelos disponibles
curl http://localhost:8317/v1/models `
  -H "Authorization: Bearer gavazia-brain-master-key-2025"

# Test de chat completion
curl http://localhost:8317/v1/chat/completions `
  -H "Authorization: Bearer gavazia-brain-master-key-2025" `
  -H "Content-Type: application/json" `
  -d '{
    "model": "gemini/gemini-2.5-pro",
    "messages": [{"role": "user", "content": "Hola, ¿cómo estás?"}]
  }'
```

Si ves una respuesta JSON con el mensaje del modelo, **¡está funcionando!** 🎉

---

## 🔧 Opción 2: Docker (Alternativa)

Si prefieres no instalar Go:

```powershell
# Construir imagen Docker
docker build -t cliproxyapi .

# Ejecutar contenedor
docker run -p 8317:8317 `
  -v "${PWD}/config.yaml:/app/config.yaml" `
  -v "${PWD}/auths:/app/auths" `
  cliproxyapi
```

---

## 📊 Configuración Avanzada (Opcional)

### Multi-Cuenta Load Balancing

```yaml
# Múltiples API keys de Gemini para rotar
gemini-api-key:
  - api-key: "AIzaSy...key1"
    prefix: "gemini-1"
  - api-key: "AIzaSy...key2"
    prefix: "gemini-2"
  - api-key: "AIzaSy...key3"
    prefix: "gemini-3"

routing:
  strategy: "round-robin"  # Rotar entre las 3 keys
```

### Agregar Claude Code (Si tienes suscripción)

```powershell
# Autenticar Claude via OAuth
curl http://localhost:8317/v0/management/oauth/claude/login

# Seguir el link que aparece para autorizar
```

### Agregar OpenAI Codex (Si tienes ChatGPT Plus/Pro)

```powershell
# Autenticar OpenAI via OAuth
curl http://localhost:8317/v0/management/oauth/codex/login

# Seguir el link para autorizar
```

---

## 🔌 Integración con Gavazia-Brain

### Crear Cliente Python

```powershell
# Crear directorio para integración
New-Item -ItemType Directory -Path "c:\Users\Andy Gavaz\Documents\Antigravity\Gavazia-Brain\06_AI_Integration" -Force

# El código del cliente ya está en GAVAZIA_INTEGRATION_PLAN.md
```

### Ejemplo de Uso con Agente

```python
from cliproxy_client import CLIProxyClient

# Inicializar cliente
client = CLIProxyClient(
    base_url="http://localhost:8317",
    api_key="gavazia-brain-master-key-2025"
)

# Usar con agente Nexxus
response = client.chat_completion(
    model="gemini/gemini-2.5-pro",
    messages=[
        {"role": "system", "content": "Eres Nexxus, ingeniero full-stack de Gavazia-Brain"},
        {"role": "user", "content": "Crea un plan para un MVP de e-commerce"}
    ],
    temperature=0.7
)

print(response['choices'][0]['message']['content'])
```

---

## 🎯 Checklist de Instalación

- [ ] Instalar Go (o usar Docker)
- [ ] Compilar CLIProxyAPI
- [ ] Copiar archivos de configuración
- [ ] Obtener Gemini API key
- [ ] Configurar `config.yaml`
- [ ] Ejecutar el proxy
- [ ] Probar con curl
- [ ] Crear cliente Python
- [ ] Integrar con primer agente (12_Orchestrator)
- [ ] Expandir a todos los agentes

---

## 🆘 Troubleshooting

### Error: "go: command not found"

**Solución**: Reiniciar PowerShell después de instalar Go

### Error: "connection refused"

**Solución**: Verificar que el proxy está corriendo en `localhost:8317`

### Error: "invalid API key"

**Solución**: Verificar que la Gemini API key es correcta en `config.yaml`

### Error: "model not found"

**Solución**: Listar modelos disponibles con `curl http://localhost:8317/v1/models`

---

## 💰 Costos Estimados

| Configuración | Costo/Mes | Modelos Disponibles |
|--------------|-----------|---------------------|
| **Solo Gemini** | $0 | Gemini 2.5 Pro, 2.0 Flash |
| **Gemini + Claude Pro** | $20 | + Claude 4.5 Sonnet |
| **Gemini + Claude + ChatGPT** | $40 | + GPT-5, o1 |

**Ahorro vs APIs directas**: $200-800/mes 💰

---

## 📚 Recursos

- **Plan de Integración Completo**: `GAVAZIA_INTEGRATION_PLAN.md`
- **Documentación Oficial**: <https://help.router-for.me/>
- **GitHub**: <https://github.com/Gavazia/CLIProxyAPI>

---

## 🚀 Próximo Paso

Una vez instalado y funcionando, continúa con:

1. Crear el cliente Python (`cliproxy_client.py`)
2. Integrar con `12_Orchestrator` primero
3. Testear con casos de uso reales
4. Expandir a los 12 agentes

**¿Listo para empezar?** Ejecuta el Paso 1 para instalar Go. 🎯
