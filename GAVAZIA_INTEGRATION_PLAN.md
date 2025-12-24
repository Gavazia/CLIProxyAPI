# 🔌 CLIProxyAPI Integration Plan for Gavazia-Brain

## 📋 Executive Summary

Este documento describe cómo integrar **CLIProxyAPI** con **Gavazia-Brain** para dar a los 12 Agentes Especializados acceso a múltiples modelos de IA premium sin costos de API.

---

## 🎯 Objetivos de la Integración

1. **Reducir Costos**: Usar modelos premium (Gemini 2.5 Pro, GPT-5, Claude 4.5) sin pagar por API calls
2. **Redundancia**: Si un modelo falla, automáticamente usar otro
3. **Flexibilidad**: Cambiar de modelo sin cambiar código de los agentes
4. **Multi-Cuenta**: Rotar entre múltiples cuentas para evitar rate limits

---

## 🏗️ Arquitectura Propuesta

```
┌─────────────────────────────────────────────────────────┐
│                  Gavazia-Brain Agents                   │
│  (01_Nexxus, 02_Vibe, 03_Oracle, ..., 12_Orchestrator) │
└────────────────────┬────────────────────────────────────┘
                     │
                     ↓
          ┌──────────────────────┐
          │   CLIProxyAPI Server │
          │   (localhost:8317)   │
          └──────────┬───────────┘
                     │
        ┌────────────┼────────────┬────────────┐
        ↓            ↓            ↓            ↓
   Gemini CLI   Claude OAuth  GPT-5 OAuth  Amp CLI
   (gratis)     (suscripción) (suscripción) (opcional)
```

---

## 📦 Componentes del Sistema

### 1. CLIProxyAPI Server

- **Puerto**: 8317 (configurable)
- **Protocolo**: HTTP/HTTPS
- **Endpoints**: OpenAI-compatible (`/v1/chat/completions`, `/v1/models`, etc.)
- **Autenticación**: API keys configurables

### 2. Proveedores Soportados

- **Gemini CLI**: Gratis, requiere autenticación CLI
- **Claude Code**: OAuth, requiere suscripción Claude Pro
- **OpenAI Codex**: OAuth, requiere suscripción ChatGPT Plus/Pro
- **Amp CLI**: Requiere suscripción Amp
- **Qwen Code**: OAuth, requiere cuenta Qwen
- **iFlow**: OAuth, requiere cuenta iFlow

### 3. Características Clave

- ✅ **Load Balancing**: Round-robin entre múltiples cuentas
- ✅ **Model Mapping**: Mapear modelos no disponibles a alternativas
- ✅ **Streaming**: Respuestas en tiempo real
- ✅ **Function Calling**: Soporte para herramientas
- ✅ **Multimodal**: Texto + imágenes

---

## 🚀 Plan de Implementación

### Fase 1: Instalación y Configuración Básica

#### 1.1. Compilar CLIProxyAPI

```powershell
cd "c:\Users\Andy Gavaz\Documents\Antigravity\Gavazia-Brain\CLIProxyAPI"
go build -o cliproxyapi.exe ./cmd/cli-proxy-api
```

#### 1.2. Crear Configuración Inicial

```powershell
# Copiar archivos de ejemplo
Copy-Item config.example.yaml config.yaml
Copy-Item .env.example .env
```

#### 1.3. Configurar `config.yaml`

```yaml
# Configuración básica
host: "127.0.0.1"  # Solo localhost por seguridad
port: 8317

# API keys para autenticación
api-keys:
  - "gavazia-brain-master-key-2025"

# Directorio de autenticación
auth-dir: "~/.cli-proxy-api"

# Habilitar debug durante setup
debug: true

# Routing strategy
routing:
  strategy: "round-robin"
```

---

### Fase 2: Configurar Proveedores

#### 2.1. Gemini CLI (Gratis - Prioridad Alta)

```yaml
# En config.yaml, descomentar y configurar:
gemini-api-key:
  - api-key: "AIzaSy...01"  # Tu API key de Google AI Studio
    prefix: "gemini"
```

**Cómo obtener la API key**:

1. Ir a <https://aistudio.google.com/apikey>
2. Crear nueva API key
3. Copiar y pegar en config.yaml

#### 2.2. Claude Code (OAuth - Si tienes suscripción)

```yaml
# Requiere autenticación OAuth
# El proxy te guiará por el proceso de login
```

**Proceso de autenticación**:

```powershell
# Ejecutar el proxy
.\cliproxyapi.exe

# En otra terminal, autenticar Claude
curl http://localhost:8317/v0/management/oauth/claude/login
# Seguir el link de OAuth que aparece
```

#### 2.3. OpenAI Codex (OAuth - Si tienes ChatGPT Plus/Pro)

```yaml
# Similar a Claude, requiere OAuth
```

**Proceso de autenticación**:

```powershell
curl http://localhost:8317/v0/management/oauth/codex/login
# Seguir el link de OAuth
```

---

### Fase 3: Integración con Gavazia-Brain

#### 3.1. Crear Cliente de API para Agentes

```python
# Archivo: Gavazia-Brain/06_AI_Integration/cliproxy_client.py

import requests
import json

class CLIProxyClient:
    def __init__(self, base_url="http://localhost:8317", api_key="gavazia-brain-master-key-2025"):
        self.base_url = base_url
        self.api_key = api_key
        self.headers = {
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        }
    
    def chat_completion(self, model, messages, stream=False, **kwargs):
        """
        Enviar request de chat completion
        
        Args:
            model: Nombre del modelo (ej: "gemini/gemini-2.5-pro", "claude-sonnet-4")
            messages: Lista de mensajes [{"role": "user", "content": "..."}]
            stream: Si True, retorna streaming response
            **kwargs: Parámetros adicionales (temperature, max_tokens, etc.)
        """
        payload = {
            "model": model,
            "messages": messages,
            "stream": stream,
            **kwargs
        }
        
        response = requests.post(
            f"{self.base_url}/v1/chat/completions",
            headers=self.headers,
            json=payload,
            stream=stream
        )
        
        if stream:
            return self._handle_stream(response)
        else:
            return response.json()
    
    def _handle_stream(self, response):
        """Manejar streaming response"""
        for line in response.iter_lines():
            if line:
                line = line.decode('utf-8')
                if line.startswith('data: '):
                    data = line[6:]
                    if data != '[DONE]':
                        yield json.loads(data)
    
    def list_models(self):
        """Listar modelos disponibles"""
        response = requests.get(
            f"{self.base_url}/v1/models",
            headers=self.headers
        )
        return response.json()

# Ejemplo de uso
if __name__ == "__main__":
    client = CLIProxyClient()
    
    # Listar modelos disponibles
    models = client.list_models()
    print("Modelos disponibles:", models)
    
    # Chat completion simple
    response = client.chat_completion(
        model="gemini/gemini-2.5-pro",
        messages=[
            {"role": "user", "content": "Hola, ¿cómo estás?"}
        ],
        temperature=0.7
    )
    print("Respuesta:", response['choices'][0]['message']['content'])
```

#### 3.2. Integrar con Agentes

```python
# Archivo: Gavazia-Brain/05_Agent_Identities/agent_base.py

from cliproxy_client import CLIProxyClient

class GavaziaAgent:
    def __init__(self, agent_name, default_model="gemini/gemini-2.5-pro"):
        self.agent_name = agent_name
        self.default_model = default_model
        self.client = CLIProxyClient()
    
    def ask(self, prompt, model=None, **kwargs):
        """
        Enviar pregunta al modelo de IA
        
        Args:
            prompt: Pregunta o instrucción
            model: Modelo a usar (si None, usa default_model)
            **kwargs: Parámetros adicionales
        """
        model = model or self.default_model
        
        messages = [
            {"role": "system", "content": f"Eres {self.agent_name}, un agente especializado de Gavazia-Brain."},
            {"role": "user", "content": prompt}
        ]
        
        response = self.client.chat_completion(
            model=model,
            messages=messages,
            **kwargs
        )
        
        return response['choices'][0]['message']['content']

# Ejemplo: Agente Nexxus
class NexxusAgent(GavaziaAgent):
    def __init__(self):
        super().__init__(
            agent_name="01_Nexxus - Backend & Apps Engineer",
            default_model="gemini/gemini-2.5-pro"
        )
    
    def build_mvp(self, requirements):
        """Construir MVP según requisitos"""
        prompt = f"""
        Como ingeniero full-stack, analiza estos requisitos y crea un plan de MVP:
        
        {requirements}
        
        Sigue el protocolo Emergent E1:
        1. Frontend con mock data
        2. Contracts.md
        3. Backend implementation
        4. Testing
        """
        return self.ask(prompt)
```

---

### Fase 4: Configuración Avanzada

#### 4.1. Model Mapping (Para Amp CLI)

```yaml
# En config.yaml
ampcode:
  upstream-url: "https://ampcode.com"
  force-model-mappings: true
  model-mappings:
    - from: "claude-opus-4.5"      # Modelo que Amp pide
      to: "claude-sonnet-4"        # Modelo que tienes disponible
    - from: "gpt-5"
      to: "gemini/gemini-2.5-pro"  # Usar Gemini en lugar de GPT-5
```

#### 4.2. Multi-Cuenta Load Balancing

```yaml
# Configurar múltiples cuentas de Gemini
gemini-api-key:
  - api-key: "AIzaSy...01"
    prefix: "gemini-1"
  - api-key: "AIzaSy...02"
    prefix: "gemini-2"
  - api-key: "AIzaSy...03"
    prefix: "gemini-3"

# El proxy rotará automáticamente entre ellas
routing:
  strategy: "round-robin"
```

---

## 📊 Configuración Recomendada para Gavazia-Brain

### Mapeo de Agentes a Modelos

| Agente | Modelo Primario | Modelo Fallback | Justificación |
|--------|----------------|-----------------|---------------|
| **01_Nexxus** | `gemini/gemini-2.5-pro` | `claude-sonnet-4` | Coding + reasoning |
| **02_Vibe** | `claude-sonnet-4` | `gemini/gemini-2.5-pro` | Creatividad visual |
| **03_Oracle** | `gemini/gemini-2.5-pro` | `claude-sonnet-4` | Análisis de datos |
| **04_DevOps** | `gemini/gemini-2.5-pro` | `gpt-5-codex` | Infraestructura |
| **05_UX_Designer** | `claude-sonnet-4` | `gemini/gemini-2.5-pro` | Diseño UX |
| **06_Business** | `gpt-5-codex` | `gemini/gemini-2.5-pro` | Análisis financiero |
| **07_Apex_Mkt** | `claude-sonnet-4` | `gemini/gemini-2.5-pro` | Copywriting |
| **08_IA_Expert** | `gemini/gemini-2.5-pro` | `claude-sonnet-4` | Automatización IA |
| **09_Arquitecto** | `claude-sonnet-4` | `gemini/gemini-2.5-pro` | Arquitectura |
| **10_Synapse** | `gemini/gemini-2.5-pro` | `claude-sonnet-4` | Búsqueda |
| **11_Cipher** | `gpt-5-codex` | `claude-sonnet-4` | Seguridad |
| **12_Orchestrator** | `gemini/gemini-2.5-pro` | `claude-sonnet-4` | Coordinación |

---

## 🔒 Seguridad

### Recomendaciones

1. **Solo Localhost**: Mantener `host: "127.0.0.1"` para evitar acceso externo
2. **API Keys Fuertes**: Usar claves aleatorias largas
3. **No Compartir Credenciales**: Mantener `.env` y `config.yaml` en `.gitignore`
4. **HTTPS**: Si necesitas acceso remoto, habilitar TLS

### Archivo `.gitignore`

```gitignore
# CLIProxyAPI
CLIProxyAPI/config.yaml
CLIProxyAPI/.env
CLIProxyAPI/auths/
CLIProxyAPI/logs/
CLIProxyAPI/cliproxyapi.exe
```

---

## 📈 Monitoreo y Testing

### 1. Verificar que el Proxy está Corriendo

```powershell
curl http://localhost:8317/v1/models
```

### 2. Test de Chat Completion

```powershell
curl http://localhost:8317/v1/chat/completions `
  -H "Authorization: Bearer gavazia-brain-master-key-2025" `
  -H "Content-Type: application/json" `
  -d '{
    "model": "gemini/gemini-2.5-pro",
    "messages": [{"role": "user", "content": "Hola"}]
  }'
```

### 3. Logs

```powershell
# Habilitar logging en config.yaml
debug: true
logging-to-file: true

# Ver logs
Get-Content CLIProxyAPI/logs/latest.log -Tail 50 -Wait
```

---

## 🎯 Próximos Pasos

1. ✅ **Clonar repositorio** (Completado)
2. ⏳ **Compilar CLIProxyAPI**
3. ⏳ **Configurar Gemini API key** (gratis, alta prioridad)
4. ⏳ **Crear cliente Python para agentes**
5. ⏳ **Integrar con 12_Orchestrator primero** (testing)
6. ⏳ **Expandir a todos los agentes**
7. ⏳ **Configurar OAuth para Claude/GPT** (opcional, si tienes suscripciones)

---

## 💰 Estimación de Costos

### Opción 1: Solo Gemini CLI (Gratis)

- **Costo**: $0/mes
- **Límites**: Rate limits de Google AI Studio
- **Modelos**: Gemini 2.5 Pro, Gemini 2.0 Flash

### Opción 2: Gemini + Claude Pro

- **Costo**: $20/mes (Claude Pro)
- **Límites**: Más generosos
- **Modelos**: Gemini 2.5 Pro + Claude 4.5 Sonnet

### Opción 3: Gemini + Claude + ChatGPT Plus

- **Costo**: $40/mes ($20 Claude + $20 ChatGPT)
- **Límites**: Muy generosos
- **Modelos**: Gemini 2.5 Pro + Claude 4.5 + GPT-5

**Comparación con APIs directas**:

- OpenAI API: ~$100-500/mes para uso intensivo
- Claude API: ~$150-400/mes
- **Ahorro estimado**: $200-800/mes 💰

---

## 📚 Recursos Adicionales

- **Documentación Oficial**: <https://help.router-for.me/>
- **Guía de Amp CLI**: <https://help.router-for.me/agent-client/amp-cli.html>
- **Management API**: <https://help.router-for.me/management/api>
- **GitHub Issues**: <https://github.com/Gavazia/CLIProxyAPI/issues>

---

> **Nota**: Este plan está diseñado para ser implementado paso a paso. Empezaremos con Gemini CLI (gratis) y luego expandiremos según necesidades.

**Estado**: 📋 Plan Creado - Listo para Implementación
**Próximo Paso**: Compilar CLIProxyAPI y configurar Gemini API key
