# Guía de Implementación Paso a Paso - Sistema de Investigación Académica en n8n

## 📋 Tabla de Contenidos

1. [Requisitos Previos](#requisitos-previos)
2. [Instalación de n8n](#instalación-de-n8n)
3. [Configuración de APIs](#configuración-de-apis)
4. [Setup de Base de Datos](#setup-de-base-de-datos)
5. [Implementación de Workflows](#implementación-de-workflows)
6. [Testing y Validación](#testing-y-validación)
7. [Deployment a Producción](#deployment-a-producción)
8. [Monitoreo y Mantenimiento](#monitoreo-y-mantenimiento)

---

## 1. Requisitos Previos

### Conocimientos Técnicos

- ✅ **Básico**: HTTP/REST APIs, JSON
- ✅ **Intermedio**: JavaScript/Node.js (para custom code nodes)
- ⚠️ **Opcional**: Docker, PostgreSQL, SQL

### Hardware/Infraestructura

**Opción A: n8n Cloud (Recomendado para empezar)**
- Navegador web
- Tarjeta de crédito para subscripción

**Opción B: Self-Hosted**
- Servidor con:
  - CPU: 2+ cores
  - RAM: 4GB+ (8GB recomendado)
  - Disco: 20GB+
  - OS: Linux (Ubuntu 22.04 recomendado)

### Cuentas Necesarias

| Servicio | Requerido | Costo | Propósito |
|----------|-----------|-------|-----------|
| Anthropic (Claude API) | ✅ Sí | $0+ | Procesamiento con IA |
| n8n Cloud | ⚠️ O self-host | $20+/mes | Plataforma de workflows |
| Semantic Scholar | ⚠️ Opcional | Gratis | Búsqueda académica |
| Google Cloud | ⚠️ Opcional | $0+ | Google Docs integration |
| PostgreSQL | ⚠️ Opcional | $0-25/mes | Almacenamiento |

---

## 2. Instalación de n8n

### Opción A: n8n Cloud (Más Fácil)

#### Paso 1: Crear Cuenta

1. Ir a [n8n.io](https://n8n.io)
2. Click en "Start for free"
3. Registrarse con email o GitHub
4. Verificar email

#### Paso 2: Seleccionar Plan

```
Plans disponibles (2025):
- Free: $0/mes - 5,000 executions, workflows básicos
- Starter: $20/mes - 10,000 executions
- Pro: $50/mes - 50,000 executions + features avanzados
```

**Recomendación**: Empezar con Free, upgrade a Starter cuando necesites más.

#### Paso 3: Acceder al Editor

1. Entrar a tu dashboard en `app.n8n.cloud`
2. Click en "Create new workflow"
3. ¡Listo para usar!

---

### Opción B: Self-Hosted con Docker (Control Total)

#### Requisitos:
- Docker instalado
- Docker Compose instalado
- Puerto 5678 disponible

#### Paso 1: Crear docker-compose.yml

```bash
mkdir n8n-research-system
cd n8n-research-system
nano docker-compose.yml
```

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    restart: unless-stopped
    environment:
      - POSTGRES_USER=n8n
      - POSTGRES_PASSWORD=n8n_password_change_me
      - POSTGRES_DB=n8n
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -h localhost -U n8n -d n8n']
      interval: 5s
      timeout: 5s
      retries: 10

  n8n:
    image: n8nio/n8n:latest
    restart: unless-stopped
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=n8n_password_change_me
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_ENV=production
      - WEBHOOK_URL=http://localhost:5678/
      - GENERIC_TIMEZONE=America/Mexico_City
    ports:
      - 5678:5678
    volumes:
      - n8n-data:/home/node/.n8n
    depends_on:
      postgres:
        condition: service_healthy

volumes:
  postgres-data:
  n8n-data:
```

#### Paso 2: Iniciar n8n

```bash
docker-compose up -d
```

#### Paso 3: Acceder a n8n

```bash
# Ver logs para confirmar que inició correctamente
docker-compose logs -f n8n

# Acceder en navegador
# http://localhost:5678
```

#### Paso 4: Setup Inicial

1. Crear cuenta de admin (primera vez)
2. Configurar email y contraseña
3. Entrar al workflow editor

---

### Opción C: Self-Hosted con npm (Desarrollo)

```bash
# Instalar n8n globalmente
npm install n8n -g

# Iniciar n8n
n8n start

# Acceder en http://localhost:5678
```

**Nota**: Esta opción NO es recomendada para producción.

---

## 3. Configuración de APIs

### 3.1 Claude API (Anthropic)

#### Paso 1: Obtener API Key

1. Ir a [console.anthropic.com](https://console.anthropic.com)
2. Crear cuenta o iniciar sesión
3. Navegar a "API Keys"
4. Click en "Create Key"
5. Copiar la key (se muestra solo una vez)

#### Paso 2: Configurar en n8n

1. En n8n, ir a "Credentials" en el menú
2. Click en "Add Credential"
3. Buscar "Anthropic"
4. Pegar API Key
5. Test connection
6. Guardar como "Claude API - Research"

**Seguridad**: Nunca compartas tu API key. n8n la encripta automáticamente.

---

### 3.2 Semantic Scholar API

#### Paso 1: Obtener API Key (Opcional pero Recomendado)

1. Ir a [semanticscholar.org/product/api](https://www.semanticscholar.org/product/api)
2. Llenar formulario de request
3. Recibir API key por email (puede tardar 1-2 días)

**Nota**: Sin API key puedes usar 1000 req/s compartidos. Con API key obtienes rate limit dedicado.

#### Paso 2: Configurar en n8n

**Si tienes API key**:
```
1. Credentials → Add Credential → Header Auth
2. Name: Authorization
3. Value: Bearer YOUR_API_KEY
4. Guardar como "Semantic Scholar API"
```

**Si NO tienes API key**:
- No configurar credential, usar HTTP Request sin auth

---

### 3.3 Google APIs (Para Google Docs integration)

#### Paso 1: Crear Proyecto en Google Cloud

1. Ir a [console.cloud.google.com](https://console.cloud.google.com)
2. Crear nuevo proyecto "n8n-research-system"
3. Habilitar APIs:
   - Google Docs API
   - Google Drive API
   - Google Sheets API (opcional)

#### Paso 2: Crear Credenciales OAuth2

1. APIs & Services → Credentials
2. Create Credentials → OAuth client ID
3. Application type: Web application
4. Authorized redirect URIs:
   - `http://localhost:5678/rest/oauth2-credential/callback` (dev)
   - `https://tu-dominio.com/rest/oauth2-credential/callback` (prod)
5. Copiar Client ID y Client Secret

#### Paso 3: Configurar en n8n

1. Credentials → Add Credential → Google OAuth2 API
2. Pegar Client ID y Client Secret
3. Scopes:
   ```
   https://www.googleapis.com/auth/documents
   https://www.googleapis.com/auth/drive
   ```
4. Click en "Connect my account"
5. Autorizar en Google
6. Guardar como "Google Docs - Research"

---

### 3.4 Otras APIs Académicas (Gratuitas)

#### Crossref (No requiere API key)
```
Endpoint: https://api.crossref.org/works
Credential: None
Rate Limit: Razonable, sin límite específico
```

#### OpenAlex (No requiere API key)
```
Endpoint: https://api.openalex.org/works
Credential: None (pero recomiendan incluir email en User-Agent)
Rate Limit: 100,000 requests/día
```

#### arXiv (No requiere API key)
```
Endpoint: http://export.arxiv.org/api/query
Credential: None
Rate Limit: 1 request/3 seconds
```

---

## 4. Setup de Base de Datos

### 4.1 Opción Simple: Usar PostgreSQL de n8n

Si usaste docker-compose con PostgreSQL, ya tienes una base de datos disponible.

#### Crear Schema para el Proyecto

1. Conectar a PostgreSQL:

```bash
docker exec -it n8n-research-system_postgres_1 psql -U n8n -d n8n
```

2. Crear tablas:

```sql
-- Tabla de búsquedas de literatura
CREATE TABLE research_searches (
    id SERIAL PRIMARY KEY,
    user_id VARCHAR(255),
    query TEXT NOT NULL,
    search_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_results INTEGER,
    analysis JSONB,
    papers JSONB,
    report_url TEXT
);

-- Índices para performance
CREATE INDEX idx_user_searches ON research_searches(user_id, search_date DESC);
CREATE INDEX idx_query ON research_searches USING gin(to_tsvector('english', query));

-- Tabla de papers individuales
CREATE TABLE papers (
    id SERIAL PRIMARY KEY,
    doi VARCHAR(255) UNIQUE,
    title TEXT NOT NULL,
    authors JSONB,
    year INTEGER,
    abstract TEXT,
    citation_count INTEGER,
    url TEXT,
    source VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    metadata JSONB
);

-- Índices
CREATE INDEX idx_doi ON papers(doi);
CREATE INDEX idx_title ON papers USING gin(to_tsvector('english', title));
CREATE INDEX idx_year ON papers(year DESC);

-- Tabla de intereses de usuarios (para monitor automático)
CREATE TABLE user_interests (
    id SERIAL PRIMARY KEY,
    user_id VARCHAR(255) NOT NULL,
    topic TEXT NOT NULL,
    keywords TEXT[],
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_checked TIMESTAMP,
    active BOOLEAN DEFAULT true
);

-- Índice
CREATE INDEX idx_user_interests ON user_interests(user_id, active);

-- Tabla de citaciones generadas
CREATE TABLE citations (
    id SERIAL PRIMARY KEY,
    paper_id INTEGER REFERENCES papers(id),
    format VARCHAR(50) DEFAULT 'APA 7',
    reference TEXT,
    in_text_narrative TEXT,
    in_text_parenthetical TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Vista para búsquedas recientes
CREATE VIEW recent_searches AS
SELECT
    user_id,
    query,
    search_date,
    total_results,
    analysis->>'overview' as summary
FROM research_searches
ORDER BY search_date DESC
LIMIT 100;
```

#### Configurar Credential en n8n

1. Credentials → Add Credential → Postgres
2. Configuración:
   ```
   Host: postgres (si usa docker-compose) o localhost
   Database: n8n
   User: n8n
   Password: [tu password del docker-compose]
   Port: 5432
   ```
3. Test connection
4. Guardar como "PostgreSQL - Research"

---

### 4.2 Opción Cloud: Supabase (Más Fácil)

#### Paso 1: Crear Proyecto en Supabase

1. Ir a [supabase.com](https://supabase.com)
2. Crear cuenta gratuita
3. New Project:
   - Name: research-system
   - Database Password: [generar contraseña segura]
   - Region: Closest to you

#### Paso 2: Crear Tablas

1. En Supabase Dashboard → SQL Editor
2. Copiar y ejecutar el SQL del paso anterior

#### Paso 3: Configurar en n8n

1. Credentials → Add Credential → Postgres
2. Configuración (obtener de Supabase → Settings → Database):
   ```
   Host: db.xxxxx.supabase.co
   Database: postgres
   User: postgres
   Password: [tu password]
   Port: 5432
   SSL: Enabled
   ```

---

### 4.3 Opción Vector Database: Pinecone (Para Búsqueda Semántica)

**Opcional pero Recomendado para Features Avanzados**

#### Paso 1: Crear Cuenta

1. [pinecone.io](https://www.pinecone.io)
2. Crear cuenta gratuita (o Starter $70/mes)

#### Paso 2: Crear Index

1. Dashboard → Create Index
2. Configuración:
   ```
   Name: research-papers
   Dimensions: 1536 (para OpenAI embeddings) o 768 (para otros)
   Metric: cosine
   Pod Type: s1 (starter)
   ```

#### Paso 3: Obtener API Key

1. API Keys → Create API Key
2. Copiar key

#### Paso 4: Configurar en n8n

1. Credentials → Add Credential → Pinecone
2. API Key: [tu key]
3. Environment: [tu environment, ej: us-east-1-aws]

---

## 5. Implementación de Workflows

### 5.1 Workflow 1: Búsqueda Simple (MVP)

#### Paso 1: Crear Nuevo Workflow

1. n8n Dashboard → "New Workflow"
2. Nombrar: "01 - Simple Literature Search"

#### Paso 2: Agregar Nodos

##### Nodo 1: Webhook Trigger

```
1. Search bar → "Webhook"
2. Drag to canvas
3. Configure:
   - HTTP Method: POST
   - Path: search-literature
   - Authentication: None (o agregar después)
   - Response Mode: Last Node
4. Click "Listen for Test Event"
```

##### Nodo 2: HTTP Request - Semantic Scholar

```
1. Add node → "HTTP Request"
2. Connect from Webhook
3. Configure:
   - Method: GET
   - URL: https://api.semanticscholar.org/graph/v1/paper/search
   - Query Parameters:
     * query: {{ $json.query }}
     * limit: {{ $json.limit || 20 }}
     * fields: paperId,title,abstract,authors,year,citationCount,url
4. (Optional) Add Credential: Semantic Scholar API
```

##### Nodo 3: Code Node - Format

```
1. Add node → "Code"
2. Configure:
   - Mode: Run Once for All Items
   - JavaScript code:

const papers = $input.first().json.data || [];

return papers.map(paper => ({
  json: {
    id: paper.paperId,
    title: paper.title,
    authors: paper.authors?.map(a => a.name).join(', ') || 'Unknown',
    year: paper.year || 'N/A',
    citations: paper.citationCount || 0,
    abstract: paper.abstract || 'No abstract',
    url: paper.url || `https://semanticscholar.org/paper/${paper.paperId}`
  }
}));
```

##### Nodo 4: Response

```
1. Add node → "Respond to Webhook"
2. Configure:
   - Response Body: {{ $json }}
```

#### Paso 3: Testing

1. Click "Execute Workflow" en toolbar
2. En otra terminal/Postman:

```bash
curl -X POST http://localhost:5678/webhook/search-literature \
  -H "Content-Type: application/json" \
  -d '{"query": "machine learning", "limit": 5}'
```

3. Verificar respuesta
4. Debug si hay errores (ver execution log)

#### Paso 4: Activar Workflow

1. Toggle "Active" en top-right
2. Webhook ahora está activo 24/7

---

### 5.2 Workflow 2: Con Claude Analysis (Core Feature)

Ver [WORKFLOWS-EJEMPLOS.md](./WORKFLOWS-EJEMPLOS.md) para código completo.

**Resumen de pasos**:

1. Duplicate "01 - Simple Literature Search"
2. Rename to "02 - Advanced Search with Claude"
3. Después del nodo de HTTP Requests, agregar:
   - Merge node (para múltiples APIs)
   - Code node (deduplicación)
   - Anthropic Chat Model node (análisis)
   - PostgreSQL node (storage)
   - Google Docs node (reporte)

**Tiempo de implementación**: 30-60 minutos

---

### 5.3 Workflow 3: Citation Generator

**Pasos**:
1. New Workflow: "03 - APA 7 Citation Generator"
2. Webhook trigger (POST /generate-citation)
3. Switch node (determinar input type: DOI/URL/manual)
4. HTTP Request a Crossref (si DOI)
5. Code node (format APA 7)
6. Response

**Tiempo de implementación**: 15-30 minutos

Ver [WORKFLOWS-EJEMPLOS.md](./WORKFLOWS-EJEMPLOS.md) para código completo.

---

### 5.4 Workflow 4: Text Reviewer

**Complejidad**: Media-Alta

**Pasos**:
1. New Workflow: "04 - Academic Text Reviewer"
2. Webhook trigger
3. Code node (split text into chunks)
4. Loop node (procesar cada chunk)
5. Claude Chat Model (multi-pass review)
6. Code node (compile results)
7. Google Docs (generate report)

**Tiempo de implementación**: 1-2 horas

---

### 5.5 Workflow 5: Automated Monitor

**Requiere**: Cron trigger (disponible en n8n Cloud Starter+)

**Pasos**:
1. New Workflow: "05 - Daily Publication Monitor"
2. Cron trigger (diario 8am)
3. PostgreSQL (load user interests)
4. Loop (foreach interest)
5. HTTP Request (search new papers)
6. Filter (remove seen)
7. Claude (summarize)
8. Email node (send digest)

**Tiempo de implementación**: 1-2 horas

---

## 6. Testing y Validación

### 6.1 Unit Testing de Workflows

Para cada workflow:

```
1. Crear Test Data node con datos de ejemplo
2. Ejecutar workflow step-by-step (click en cada nodo)
3. Verificar output de cada nodo
4. Revisar execution log para errores
5. Testear edge cases:
   - Input vacío
   - Papers sin abstract
   - APIs que fallan
   - Rate limits
```

### 6.2 Integration Testing

```bash
# Script de testing automatizado
# test-workflows.sh

#!/bin/bash

echo "Testing Workflow 1: Simple Search"
curl -X POST http://localhost:5678/webhook/search-literature \
  -H "Content-Type: application/json" \
  -d '{"query": "test query", "limit": 5}'

echo "\nTesting Workflow 3: Citation Generator"
curl -X POST http://localhost:5678/webhook/generate-citation \
  -H "Content-Type: application/json" \
  -d '{"doi": "10.1038/s41586-020-2649-2"}'

# Add more tests...
```

### 6.3 Performance Testing

```javascript
// Dentro de n8n, crear workflow de performance test

// Nodo 1: HTTP Request en loop
for (let i = 0; i < 100; i++) {
  // Medir tiempo de respuesta
  const start = Date.now();

  // Llamar a workflow
  await $http.request({
    method: 'POST',
    url: 'http://localhost:5678/webhook/search-literature',
    body: { query: 'test', limit: 5 }
  });

  const duration = Date.now() - start;
  console.log(`Request ${i}: ${duration}ms`);
}
```

### 6.4 Checklist de Validación

- [ ] Todos los workflows se ejecutan sin errores
- [ ] Tiempos de respuesta < 60 segundos (búsqueda con Claude)
- [ ] Datos se almacenan correctamente en PostgreSQL
- [ ] Reportes de Google Docs se generan correctamente
- [ ] Citaciones APA 7 tienen formato correcto
- [ ] Emails de monitor se envían exitosamente
- [ ] No hay leaks de API keys en logs
- [ ] Rate limits están bajo control

---

## 7. Deployment a Producción

### 7.1 Security Hardening

#### Paso 1: Agregar Autenticación a Webhooks

```
1. En cada Webhook node → Authentication
2. Options:
   - Basic Auth (usuario/contraseña)
   - Header Auth (API token)
   - OAuth2 (más complejo pero más seguro)

3. Distribuir credentials a usuarios autorizados
```

#### Paso 2: HTTPS/SSL

**Si self-hosted**:

```bash
# Instalar Caddy como reverse proxy
sudo apt install caddy

# Configurar Caddyfile
sudo nano /etc/caddy/Caddyfile
```

```
research.tu-dominio.com {
    reverse_proxy localhost:5678
}
```

```bash
# Reiniciar Caddy
sudo systemctl restart caddy
```

**Si n8n Cloud**: SSL ya está configurado.

#### Paso 3: Firewall Rules

```bash
# UFW (Ubuntu)
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw deny 5678/tcp   # Block direct n8n access
sudo ufw enable
```

#### Paso 4: Environment Variables

```bash
# Crear .env file
cat > .env << EOF
CLAUDE_API_KEY=sk-ant-xxx
SEMANTIC_SCHOLAR_KEY=xxx
POSTGRES_PASSWORD=xxx
N8N_ENCRYPTION_KEY=$(openssl rand -hex 32)
EOF

# Proteger archivo
chmod 600 .env
```

---

### 7.2 Configurar Dominio Custom (Self-hosted)

```bash
# 1. Comprar dominio (ej: research-ai.com)

# 2. Configurar DNS A record:
# research-ai.com → tu-server-ip

# 3. Actualizar docker-compose.yml
environment:
  - N8N_HOST=research-ai.com
  - N8N_PROTOCOL=https
  - WEBHOOK_URL=https://research-ai.com/

# 4. Restart
docker-compose down && docker-compose up -d
```

---

### 7.3 Backups Automáticos

#### PostgreSQL Backup Script

```bash
#!/bin/bash
# backup-db.sh

BACKUP_DIR="/backups/n8n"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

docker exec n8n-research-system_postgres_1 \
  pg_dump -U n8n n8n > $BACKUP_DIR/n8n_backup_$DATE.sql

# Mantener solo últimos 30 días
find $BACKUP_DIR -name "n8n_backup_*.sql" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR/n8n_backup_$DATE.sql"
```

#### Automatizar con Cron

```bash
# Editar crontab
crontab -e

# Agregar línea (backup diario a las 2am)
0 2 * * * /home/user/backup-db.sh
```

---

### 7.4 Monitoring y Alertas

#### Configurar Uptime Monitoring

Opciones gratuitas:
- **UptimeRobot**: https://uptimerobot.com
- **StatusCake**: https://www.statuscake.com

Configurar checks para:
- n8n UI: https://research-ai.com
- Webhook endpoints: https://research-ai.com/webhook/search-literature

#### Alertas por Email/Slack

Crear workflow en n8n:

```
Cron Trigger (cada hora)
  → HTTP Request (health check a webhook principal)
  → IF (check si respuesta OK)
      → NO → Send Slack/Email alert
      → YES → No action
```

---

### 7.5 Logging

#### Configurar Log Levels

```yaml
# docker-compose.yml
services:
  n8n:
    environment:
      - N8N_LOG_LEVEL=info  # debug, info, warn, error
      - N8N_LOG_OUTPUT=console,file
      - N8N_LOG_FILE_LOCATION=/home/node/.n8n/logs/
```

#### Log Rotation

```bash
# Instalar logrotate
sudo apt install logrotate

# Configurar
sudo nano /etc/logrotate.d/n8n
```

```
/home/node/.n8n/logs/*.log {
    daily
    rotate 30
    compress
    missingok
    notifempty
}
```

---

## 8. Monitoreo y Mantenimiento

### 8.1 Métricas a Monitorear

#### En n8n Cloud Dashboard

- Workflow executions (por mes)
- Execution times (promedio)
- Error rates
- API usage

#### En PostgreSQL

```sql
-- Query para ver uso por usuario
SELECT
    user_id,
    COUNT(*) as total_searches,
    AVG(total_results) as avg_results,
    MAX(search_date) as last_search
FROM research_searches
WHERE search_date >= NOW() - INTERVAL '30 days'
GROUP BY user_id
ORDER BY total_searches DESC;

-- Papers más citados en DB
SELECT title, authors, citation_count
FROM papers
ORDER BY citation_count DESC
LIMIT 20;
```

#### Costos de Claude API

```javascript
// Workflow para calcular costos mensuales

// Nodo: Code - Calculate Costs
const executions = $input.all(); // Todas las ejecuciones del mes

let totalInputTokens = 0;
let totalOutputTokens = 0;

executions.forEach(exec => {
  totalInputTokens += exec.json.usage?.input_tokens || 0;
  totalOutputTokens += exec.json.usage?.output_tokens || 0;
});

const inputCost = (totalInputTokens / 1_000_000) * 3; // $3 per 1M tokens
const outputCost = (totalOutputTokens / 1_000_000) * 15; // $15 per 1M tokens
const totalCost = inputCost + outputCost;

return {
  period: 'Monthly',
  totalInputTokens,
  totalOutputTokens,
  inputCost,
  outputCost,
  totalCost
};
```

---

### 8.2 Optimizaciones

#### Caching de Resultados

```javascript
// Antes de llamar a API, check cache
const cacheKey = `search_${$json.query}`;
const cached = await redis.get(cacheKey);

if (cached) {
  return JSON.parse(cached);
} else {
  // Hacer búsqueda
  const results = await searchAPI($json.query);

  // Guardar en cache (24 horas)
  await redis.set(cacheKey, JSON.stringify(results), 'EX', 86400);

  return results;
}
```

#### Rate Limiting

```javascript
// Implementar rate limiting en webhook
const userRequests = await redis.incr(`rate_limit:${$json.userId}`);

if (userRequests > 100) { // 100 requests per hour
  throw new Error('Rate limit exceeded. Try again later.');
}

// Expirar contador después de 1 hora
await redis.expire(`rate_limit:${$json.userId}`, 3600);
```

#### Batch Processing

En lugar de procesar papers uno por uno:

```javascript
// BAD: Loop con 50 llamadas a Claude
for (const paper of papers) {
  await claudeAnalyze(paper);
}

// GOOD: Batch de 10 papers por llamada
const batches = chunk(papers, 10);
for (const batch of batches) {
  await claudeAnalyzeBatch(batch); // Procesar 10 juntos
}
```

---

### 8.3 Mantenimiento Regular

#### Checklist Semanal

- [ ] Revisar error logs
- [ ] Verificar backups están funcionando
- [ ] Monitorear costos de APIs
- [ ] Revisar performance (tiempos de respuesta)
- [ ] Actualizar workflows si hay bugs reportados

#### Checklist Mensual

- [ ] Actualizar n8n a última versión
- [ ] Revisar y optimizar queries de PostgreSQL
- [ ] Limpiar datos antiguos (>6 meses)
- [ ] Analizar métricas de uso
- [ ] Planear nuevas features basado en feedback

#### Actualizar n8n

```bash
# Self-hosted con Docker
cd n8n-research-system
docker-compose pull
docker-compose down
docker-compose up -d

# Verificar versión
docker exec n8n-research-system_n8n_1 n8n --version
```

#### Limpiar Datos Antiguos

```sql
-- Eliminar búsquedas >6 meses
DELETE FROM research_searches
WHERE search_date < NOW() - INTERVAL '6 months';

-- Vacuum para recuperar espacio
VACUUM FULL research_searches;
```

---

## 9. Troubleshooting Común

### Problema: Workflow no se ejecuta

**Solución**:
1. Verificar que workflow está "Active"
2. Check webhook URL es accesible
3. Ver execution log para errores
4. Verificar credenciales de APIs

### Problema: Claude API timeout

**Solución**:
1. Aumentar timeout en HTTP Request node
2. Reducir tamaño de input (menos tokens)
3. Usar Claude Haiku (más rápido) en vez de Sonnet

### Problema: PostgreSQL connection refused

**Solución**:
```bash
# Verificar PostgreSQL está running
docker ps | grep postgres

# Ver logs
docker logs n8n-research-system_postgres_1

# Restart
docker-compose restart postgres
```

### Problema: Costos de Claude muy altos

**Solución**:
1. Implementar caching agresivo
2. Reducir max_tokens en prompts
3. Usar Claude Haiku para tareas simples
4. Implementar rate limiting por usuario
5. Agregar confirmación antes de ejecutar análisis costosos

---

## 10. Recursos Adicionales

### Documentación Oficial

- [n8n Docs](https://docs.n8n.io/)
- [Claude API Docs](https://docs.anthropic.com/)
- [Semantic Scholar API](https://api.semanticscholar.org/)
- [Crossref API](https://github.com/CrossRef/rest-api-doc)

### Comunidades

- [n8n Community Forum](https://community.n8n.io/)
- [n8n Discord](https://discord.gg/n8n)
- Reddit: r/n8n

### Tutoriales

- [n8n YouTube Channel](https://www.youtube.com/@n8n-io)
- [n8n Workflow Templates](https://n8n.io/workflows/)

---

## 🎉 ¡Felicidades!

Si llegaste hasta aquí y completaste todos los pasos, ahora tienes un **sistema completo de investigación académica automatizado** corriendo en n8n.

### Próximos Pasos Sugeridos

1. **Personalizar workflows** según tus necesidades específicas
2. **Agregar más APIs** (PubMed, IEEE Xplore, etc.)
3. **Crear interfaz web** personalizada (con Next.js + n8n webhooks)
4. **Integrar con Zotero/Mendeley** para gestión de referencias
5. **Implementar features avanzados** (recomendaciones de papers, network analysis)

---

**Tiempo total de implementación estimado**: 8-16 horas (dependiendo de experiencia)

**Documento creado**: 2025-11-05

**Versión**: 1.0

¿Preguntas? Consulta el documento de [FACTIBILIDAD-TECNICA.md](./FACTIBILIDAD-TECNICA.md) para más detalles.
