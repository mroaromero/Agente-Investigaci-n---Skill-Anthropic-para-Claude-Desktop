# Análisis de Factibilidad Técnica: Sistema de Investigación Académica en n8n

## 📋 Resumen Ejecutivo

**Proyecto**: Sistema de Investigación Académica Automatizado con IA

**Plataforma**: n8n (Workflow Automation Platform)

**Objetivo**: Migrar y expandir las capacidades de la Custom Skill de investigación académica a un sistema automatizado en n8n con máximas capacidades.

**Veredicto**: ✅ **ALTAMENTE FACTIBLE** con arquitectura híbrida y algunas limitaciones conocidas.

**Complejidad**: Media-Alta

**Tiempo estimado de implementación**: 4-6 semanas

**Inversión estimada**: $50-200/mes (dependiendo de uso y plan)

---

## 🎯 Capacidades Objetivo

### 1. Búsqueda y Análisis de Literatura
- Búsqueda automatizada en múltiples bases de datos académicas
- Extracción y análisis de metadatos
- Ranking de relevancia de papers
- Detección de gaps en investigación
- Generación de resúmenes críticos

### 2. Desarrollo de Hipótesis
- Análisis de literatura para identificar variables
- Sugerencias de metodologías basadas en precedentes
- Evaluación de viabilidad estadística
- Generación de marcos teóricos

### 3. Citación y Referencias (APA 7)
- Conversión automática de metadatos a formato APA 7
- Validación de formato de citas
- Gestión de bibliografía
- Detección de duplicados

### 4. Escritura Académica Asistida
- Revisión de estilo y gramática académica
- Sugerencias de conectores y transiciones
- Paráfrasis para evitar plagio
- Mejora de claridad y precisión

### 5. Automatización de Workflows
- Alertas de nuevas publicaciones en áreas de interés
- Generación automatizada de reportes de literatura
- Extracción de datos de múltiples papers
- Integración con gestores de referencias

---

## 🏗️ Arquitectura Propuesta

```
┌─────────────────────────────────────────────────────────────────┐
│                         n8n WORKFLOWS                            │
└─────────────────────────────────────────────────────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│  ACADEMIC     │     │   AI/LLM      │     │  STORAGE &    │
│  DATA SOURCES │     │   PROCESSING  │     │  OUTPUT       │
└───────────────┘     └───────────────┘     └───────────────┘
        │                       │                       │
  ┌─────┴─────┐         ┌───────┴───────┐       ┌─────┴─────┐
  │           │         │               │       │           │
  ▼           ▼         ▼               ▼       ▼           ▼
Semantic   Crossref  Claude API    OpenAI   PostgreSQL   Google
Scholar              (Anthropic)   GPT-4                  Docs

OpenAlex   PubMed    Prompt        Embeddings  Airtable  Notion
                     Engineering

Custom     arXiv     Function      Vector      JSON      Email
Scrapers            Calling        DB          Files
```

### Componentes Principales

#### 1. **Capa de Ingesta de Datos Académicos**

**APIs Disponibles en n8n:**

- ✅ **Semantic Scholar API** (Recomendado)
  - 200M papers indexados
  - Metadatos ricos (abstract, citations, PDFs)
  - Rate limit: 1000 req/s (sin auth), ilimitado (con API key)
  - **Costo**: GRATUITO

- ✅ **Crossref API**
  - Metadatos bibliográficos de alta calidad
  - DOIs, funding data, ORCID
  - Rate limit: Razonable (no especificado)
  - **Costo**: GRATUITO

- ✅ **OpenAlex API**
  - Agregador de Microsoft Academic, Crossref, ORCID
  - 200M+ works indexados
  - Rate limit: 100,000 llamadas/día
  - **Costo**: GRATUITO

- ⚠️ **Google Scholar** (Limitado)
  - NO tiene API oficial
  - Requiere web scraping (contra TOS)
  - Alternativa: Usar SerpAPI o ScraperAPI
  - **Costo**: $50-150/mes (APIs terceros)

- ✅ **PubMed/PMC API**
  - Especializado en ciencias médicas/biológicas
  - E-utilities gratuitas
  - **Costo**: GRATUITO

- ✅ **arXiv API**
  - Preprints en física, matemáticas, CS
  - API REST completa
  - **Costo**: GRATUITO

#### 2. **Capa de Procesamiento con IA/LLM**

**Integración Claude (Anthropic) en n8n:**

- ✅ **Nodo nativo de Anthropic** disponible en n8n
- ✅ **AI Agent node** con soporte para Claude Sonnet 4 y Opus 4
- ✅ **Function calling** y **tool use** soportados
- ✅ **Batch processing** de prompts disponible

**Capacidades:**
- Análisis de abstracts y papers completos
- Generación de resúmenes críticos
- Identificación de variables y metodologías
- Revisión de escritura académica
- Paráfrasis y mejora de estilo
- Generación de hipótesis basadas en literatura

**Alternativas/Complementos:**
- OpenAI GPT-4 (nodo nativo en n8n)
- Embeddings para búsqueda semántica
- LangChain nodes para RAG (Retrieval-Augmented Generation)

#### 3. **Capa de Almacenamiento y Gestión de Datos**

**Opciones de Base de Datos:**

- ✅ **PostgreSQL** (nodo nativo)
  - Almacenamiento relacional de papers y metadatos
  - Búsquedas complejas y relaciones

- ✅ **Vector Database** (Pinecone, Weaviate)
  - Búsqueda semántica de papers
  - Embeddings para similarity search

- ✅ **Airtable** (nodo nativo)
  - Base de datos colaborativa
  - Interfaz visual para gestión de literatura

- ✅ **Google Sheets** (nodo nativo)
  - Almacenamiento simple y colaborativo
  - Fácil integración con otros tools

#### 4. **Capa de Outputs e Integraciones**

**Destinos de Output:**

- ✅ **Google Docs** - Generación de reportes automáticos
- ✅ **Notion** - Base de conocimiento de investigación
- ✅ **Zotero** (vía API) - Gestor de referencias
- ✅ **Mendeley** (vía API) - Gestor de referencias
- ✅ **Email** - Alertas y notificaciones
- ✅ **Slack/Discord** - Notificaciones de equipo
- ✅ **PDF Generation** - Exportación de documentos
- ✅ **LaTeX** - Generación de papers académicos

---

## 🔄 Workflows Principales Propuestos

### Workflow 1: Búsqueda Exhaustiva de Literatura

```
TRIGGER (Manual/Webhook)
    │
    ├─ Input: Tema de investigación
    │
    ▼
PARALLEL SEARCH (Split in Batches)
    │
    ├─► Semantic Scholar API
    ├─► Crossref API
    ├─► OpenAlex API
    ├─► arXiv API
    └─► PubMed API
    │
    ▼
MERGE & DEDUPLICATE
    │
    ├─ Normalizar metadatos
    ├─ Eliminar duplicados por DOI
    └─ Rankear por relevancia
    │
    ▼
CLAUDE ANALYSIS (AI Agent)
    │
    ├─ Analizar abstracts
    ├─ Identificar papers clave
    ├─ Detectar gaps de investigación
    └─ Generar resumen crítico
    │
    ▼
STORE RESULTS
    │
    ├─► PostgreSQL (metadatos)
    ├─► Vector DB (embeddings)
    └─► Airtable (vista colaborativa)
    │
    ▼
OUTPUT
    │
    ├─► Google Docs (reporte formateado)
    ├─► Email (notificación)
    └─► Notion (actualizar base de conocimiento)
```

**Tiempo estimado**: 2-5 minutos (50-100 papers)

**Tokens Claude**: ~50,000-100,000 tokens por búsqueda

**Costo por ejecución**: ~$0.50-1.50

---

### Workflow 2: Desarrollo de Hipótesis Basada en Evidencia

```
TRIGGER (Manual)
    │
    ├─ Input: Área de investigación, contexto
    │
    ▼
RETRIEVE RELEVANT LITERATURE
    │
    ├─ Query Vector DB (semantic search)
    └─ Recuperar top 20 papers relevantes
    │
    ▼
CLAUDE ANALYSIS - Stage 1: Variables
    │
    ├─ Identificar variables dependientes/independientes
    ├─ Analizar relaciones causales en literatura
    └─ Identificar variables de control
    │
    ▼
CLAUDE ANALYSIS - Stage 2: Metodologías
    │
    ├─ Revisar diseños metodológicos previos
    ├─ Evaluar fortalezas y limitaciones
    └─ Sugerir metodología apropiada
    │
    ▼
CLAUDE GENERATION - Stage 3: Hipótesis
    │
    ├─ Formular hipótesis H1, H2, H3...
    ├─ Justificar con base en literatura
    └─ Evaluar viabilidad
    │
    ▼
FORMAT & OUTPUT
    │
    ├─► Google Docs (documento estructurado)
    └─► Save to DB
```

**Tiempo estimado**: 3-7 minutos

**Tokens Claude**: ~100,000-200,000 tokens

**Costo por ejecución**: ~$1-3

---

### Workflow 3: Generación de Citaciones APA 7

```
TRIGGER (Webhook/Manual)
    │
    ├─ Input: DOI, URL, o metadatos parciales
    │
    ▼
FETCH METADATA
    │
    ├─ IF DOI → Crossref API
    ├─ IF Title → Semantic Scholar Search
    └─ ELSE → Manual input
    │
    ▼
NORMALIZE METADATA
    │
    ├─ Extract: Author, Year, Title, Journal, etc.
    └─ Validate completeness
    │
    ▼
CLAUDE FORMATTING (Optional)
    │
    ├─ Format según APA 7
    ├─ Handle edge cases (múltiples autores, sin fecha, etc.)
    └─ Generate in-text citation variants
    │
    OR
    │
TEMPLATE FORMATTING (Faster)
    │
    └─ Apply APA 7 template con JavaScript
    │
    ▼
OUTPUT
    │
    ├─► Return formatted citation
    └─► Add to Zotero/Mendeley (optional)
```

**Tiempo estimado**: 5-15 segundos

**Tokens Claude**: ~500-1,000 (si se usa)

**Costo por ejecución**: ~$0.001-0.01

---

### Workflow 4: Revisión de Texto Académico

```
TRIGGER (Manual/API)
    │
    ├─ Input: Texto a revisar
    │
    ▼
PREPROCESSING
    │
    ├─ Split por párrafos/secciones
    └─ Identificar contexto (intro, métodos, etc.)
    │
    ▼
CLAUDE ANALYSIS - Multi-pass
    │
    ├─ Pass 1: Gramática y claridad
    ├─ Pass 2: Estilo académico y tono
    ├─ Pass 3: Conectores y transiciones
    ├─ Pass 4: Argumentación y evidencia
    └─ Pass 5: Detección de potencial plagio
    │
    ▼
ACADEMIC PHRASEBANK (Optional)
    │
    ├─ Query vectorDB con frases del Academic Phrasebank
    └─ Sugerir alternativas académicas
    │
    ▼
GENERATE REPORT
    │
    ├─ Texto original vs mejorado (side-by-side)
    ├─ Explicación de cambios
    ├─ Sugerencias adicionales
    └─ Score de calidad académica
    │
    ▼
OUTPUT
    │
    ├─► Google Docs (documento con comentarios)
    └─► JSON (para integración con editores)
```

**Tiempo estimado**: 1-3 minutos (1000 palabras)

**Tokens Claude**: ~20,000-50,000 tokens

**Costo por ejecución**: ~$0.30-0.75

---

### Workflow 5: Monitor de Nuevas Publicaciones (Automatizado)

```
TRIGGER (Cron: Diario 8am)
    │
    ├─ Load user research interests from DB
    │
    ▼
FOREACH Interest Topic
    │
    ▼
SEARCH NEW PAPERS (Last 24h)
    │
    ├─► Semantic Scholar (date filter)
    ├─► arXiv (recent submissions)
    └─► Crossref (recent publications)
    │
    ▼
FILTER & RANK
    │
    ├─ Eliminar ya vistos (check DB)
    ├─ Calcular relevance score
    └─ Top 5 papers por topic
    │
    ▼
CLAUDE SUMMARIZATION
    │
    ├─ Generate brief summary (2-3 sentences)
    └─ Extract key findings
    │
    ▼
COMPILE DIGEST
    │
    ├─ Group by topic
    └─ Format email template
    │
    ▼
SEND NOTIFICATION
    │
    ├─► Email (daily digest)
    ├─► Slack (si hay papers muy relevantes)
    └─► Update Notion database
```

**Frecuencia**: Diaria

**Tiempo de ejecución**: 5-10 minutos

**Tokens Claude**: ~10,000-20,000 tokens/día

**Costo mensual**: ~$15-30

---

## 💰 Análisis de Costos

### Costos de n8n

**Opción 1: Self-hosted (Open Source)**
- n8n: GRATUITO (self-hosted)
- Servidor: $5-20/mes (VPS - DigitalOcean, Hetzner)
- **Total**: $5-20/mes

**Opción 2: n8n Cloud**
- Starter: $20/mes (5,000 workflow executions)
- Pro: $50/mes (10,000 workflow executions)
- **Total**: $20-50/mes

### Costos de APIs Académicas

- Semantic Scholar: GRATUITO
- Crossref: GRATUITO
- OpenAlex: GRATUITO
- arXiv: GRATUITO
- PubMed: GRATUITO
- Google Scholar (via SerpAPI): $50-150/mes (opcional)

**Total APIs académicas**: $0-150/mes

### Costos de IA/LLM

**Claude API (Anthropic)**
- Entrada: $3 / 1M tokens
- Salida: $15 / 1M tokens

**Estimación de uso mensual (investigador activo):**
- 100 búsquedas de literatura/mes: ~10M tokens → $30-45
- 20 desarrollos de hipótesis/mes: ~4M tokens → $20-30
- 100 revisiones de texto/mes: ~4M tokens → $20-30
- 1 monitor diario (30 ejecuciones): ~600K tokens → $3-5

**Total Claude**: ~$75-110/mes

**Alternativa con OpenAI GPT-4:**
- Similar o ligeramente más caro

### Costos de Almacenamiento

**Opciones:**

1. **PostgreSQL self-hosted**: $0 (incluido en VPS)
2. **Supabase** (PostgreSQL managed): $25/mes
3. **Airtable**: $20/mes (Plus plan)
4. **Pinecone** (Vector DB): $70/mes (starter)

**Total Storage**: $0-70/mes

### 📊 Resumen de Costos Totales

| Configuración | n8n | APIs | Claude | Storage | **Total/mes** |
|---------------|-----|------|--------|---------|---------------|
| **Mínima** (self-hosted, sin Google Scholar, storage básico) | $10 | $0 | $75 | $0 | **$85** |
| **Recomendada** (n8n Cloud, APIs gratuitas, storage managed) | $50 | $0 | $100 | $25 | **$175** |
| **Premium** (todo managed, Google Scholar, vector DB) | $50 | $150 | $110 | $70 | **$380** |

**Para un usuario individual investigador activo**: $85-175/mes

**Para un equipo pequeño (3-5 personas)**: $200-400/mes

---

## ⚡ Performance y Escalabilidad

### Capacidad de n8n

- **Ejecuciones simultáneas**: 220 workflows/segundo (single instance)
- **Timeout por workflow**: Configurable (default 2 minutos)
- **Memoria**: ~512MB-1GB por instancia

### Tiempos de Respuesta Estimados

| Tarea | Tiempo | Notas |
|-------|--------|-------|
| Búsqueda simple (1 API) | 2-5 seg | Sin análisis IA |
| Búsqueda exhaustiva (5 APIs) | 10-30 seg | Paralelo |
| Análisis con Claude | 20-60 seg | Depende de tokens |
| Desarrollo de hipótesis completo | 3-7 min | Multi-stage |
| Revisión de texto (1000 palabras) | 1-3 min | Incluye sugerencias |
| Generación de citación | 5-15 seg | Con fetch de metadata |

### Optimizaciones

1. **Caching**: Almacenar resultados de búsquedas frecuentes
2. **Batch Processing**: Procesar múltiples papers en un solo prompt
3. **Streaming**: Respuestas en tiempo real con Claude streaming API
4. **Queue Management**: Cola de tareas para evitar rate limits

---

## 🔒 Seguridad y Privacidad

### Consideraciones

- **Datos académicos**: Mayormente públicos (sin problemas)
- **Textos del usuario**: Sensibles (pueden contener investigación no publicada)
- **APIs de terceros**: Revisar términos de uso

### Recomendaciones

1. **Self-host n8n** para control total de datos
2. **Encriptar credenciales** de APIs (n8n lo hace por defecto)
3. **No enviar papers completos** a APIs sin permisos (copyright)
4. **Usar Claude on-prem** si hay datos muy sensibles (Enterprise)
5. **Configurar backups** regulares de databases

---

## 🚧 Limitaciones y Desafíos

### Limitaciones Técnicas

1. **Google Scholar**: No tiene API oficial
   - Solución: Usar APIs de terceros ($) o enfocarse en Semantic Scholar

2. **Acceso a PDFs completos**: Muchos papers están tras paywall
   - Solución: Usar Unpaywall API, Sci-Hub (legal gris), o limitar a abstracts

3. **Rate Limits**: APIs gratuitas tienen límites
   - Solución: Implementar rate limiting y colas en n8n

4. **Calidad de metadata**: Varía entre fuentes
   - Solución: Cross-reference múltiples APIs

5. **Context window de LLMs**: Claude tiene límite de ~200K tokens
   - Solución: Chunking y summarización progresiva

### Limitaciones de IA

1. **Alucinaciones**: Claude puede inventar citas o datos
   - Solución: Siempre verificar referencias, usar retrieval

2. **Sesgo**: LLMs pueden tener sesgos disciplinarios
   - Solución: Prompt engineering cuidadoso, validación humana

3. **Detección de plagio**: No es 100% confiable
   - Solución: Complementar con Turnitin o herramientas especializadas

### Desafíos de Implementación

1. **Curva de aprendizaje**: n8n requiere familiarización
   - Tiempo estimado: 1-2 semanas para dominar lo básico

2. **Mantenimiento**: APIs y n8n requieren actualizaciones
   - Tiempo estimado: 2-4 horas/mes

3. **Costos variables**: Dependen mucho del uso
   - Solución: Monitorear y optimizar continuamente

---

## 📈 Roadmap de Implementación

### Fase 1: MVP (Semanas 1-2)

**Objetivo**: Validar concepto con funcionalidades básicas

- [ ] Setup de n8n (self-hosted o cloud)
- [ ] Integración con Semantic Scholar API
- [ ] Integración con Claude API (Anthropic node)
- [ ] Workflow 1: Búsqueda básica de literatura
- [ ] Workflow 3: Generación de citaciones APA 7
- [ ] Storage en Google Sheets (simple)
- [ ] Testing con casos reales

**Entregables**:
- 2 workflows funcionales
- Documentación básica
- Demo con 10-20 búsquedas de prueba

### Fase 2: Core Features (Semanas 3-4)

**Objetivo**: Implementar funcionalidades principales

- [ ] Integración multi-API (Crossref, OpenAlex, arXiv)
- [ ] Workflow 2: Desarrollo de hipótesis
- [ ] Workflow 4: Revisión de texto académico
- [ ] PostgreSQL para almacenamiento estructurado
- [ ] Vector database para búsqueda semántica (Pinecone/Weaviate)
- [ ] Output a Google Docs con formato
- [ ] Deduplicación y ranking de resultados

**Entregables**:
- 4 workflows completos
- Base de datos funcional
- API endpoints (webhooks)

### Fase 3: Automatización (Semanas 5-6)

**Objetivo**: Workflows automáticos y avanzados

- [ ] Workflow 5: Monitor de nuevas publicaciones
- [ ] Sistema de alertas (Email/Slack)
- [ ] Integración con gestores de referencias (Zotero)
- [ ] Dashboard en Notion/Airtable
- [ ] Academic Phrasebank como vector DB
- [ ] Batch processing de papers
- [ ] Optimizaciones de performance

**Entregables**:
- Sistema completo automatizado
- Dashboard de visualización
- Notificaciones configurables

### Fase 4: Refinamiento (Semana 7+)

**Objetivo**: Optimización y features adicionales

- [ ] Optimización de costos (caching, batch)
- [ ] Interfaz web custom (opcional)
- [ ] Export a LaTeX/Overleaf
- [ ] Análisis de tendencias de investigación
- [ ] Recomendaciones de colaboradores potenciales
- [ ] Integración con ORCID
- [ ] Multilingual support

---

## 🎯 Ventajas vs Custom Skill de Claude Desktop

| Aspecto | Custom Skill (Claude Desktop) | Sistema n8n |
|---------|-------------------------------|-------------|
| **Setup** | Simple (copiar archivo) | Medio (requiere configuración) |
| **Costo** | Solo Claude subscription ($20/mes) | $85-175/mes |
| **Automatización** | ❌ Manual, requiere interacción | ✅ Workflows automáticos |
| **Integraciones** | ⚠️ Limitadas a lo que Claude puede hacer | ✅ 400+ integraciones nativas |
| **Almacenamiento** | ❌ No persiste datos | ✅ Databases, archivos, cloud |
| **APIs académicas** | ⚠️ Claude debe buscar (menos confiable) | ✅ Acceso directo a APIs |
| **Batch processing** | ❌ Uno por uno | ✅ Procesar 100s de papers |
| **Colaboración** | ⚠️ Individual | ✅ Multi-usuario, compartido |
| **Alertas automáticas** | ❌ No | ✅ Cron jobs, monitoring |
| **Customización** | ⚠️ Limitada a prompts | ✅ Total (código, lógica) |
| **Escalabilidad** | ⚠️ Manual scaling | ✅ Auto-scaling |

---

## ✅ Recomendaciones Finales

### Para Usuario Individual (Estudiante/Investigador)

**Recomiendo**: Empezar con Custom Skill y evaluar n8n si necesitas:
- Automatización de búsquedas recurrentes
- Procesar grandes volúmenes de papers
- Integración con gestores de referencias
- Alertas automáticas

**Configuración sugerida**:
- n8n self-hosted ($10/mes)
- APIs gratuitas (Semantic Scholar, Crossref)
- Claude API ($75-100/mes)
- PostgreSQL self-hosted ($0)
- **Total: ~$85-110/mes**

### Para Equipo de Investigación (3-10 personas)

**Recomiendo**: Sistema completo en n8n

**Configuración sugerida**:
- n8n Cloud Pro ($50/mes)
- APIs gratuitas + SerpAPI ($50/mes)
- Claude API ($100-150/mes según uso compartido)
- Supabase + Pinecone ($95/mes)
- **Total: ~$295/mes** → $30-60/persona/mes

### Para Institución Académica

**Recomiendo**: Implementación enterprise con self-hosting

**Configuración sugerida**:
- n8n Enterprise (self-hosted) - Licencia anual
- Servidores propios
- Claude Enterprise API
- Infraestructura propia
- **Análisis de costos personalizado**

---

## 📚 Recursos y Próximos Pasos

### Documentación

- [n8n Documentation](https://docs.n8n.io/)
- [Anthropic API Docs](https://docs.anthropic.com/)
- [Semantic Scholar API](https://www.semanticscholar.org/product/api)
- [Crossref API](https://www.crossref.org/documentation/retrieve-metadata/)

### Templates de n8n

- [n8n Workflow Templates](https://n8n.io/workflows/)
- [Claude AI Workflows](https://n8n.io/integrations/claude/)

### Siguientes Acciones

1. **Decidir configuración** (individual vs equipo)
2. **Setup de n8n** (cloud o self-hosted)
3. **Obtener API keys** (Anthropic, Semantic Scholar)
4. **Implementar MVP** (Fase 1 del roadmap)
5. **Testing con casos reales**
6. **Iterar y expandir**

---

## 🤔 Preguntas Clave Para Decidir

1. **¿Cuántas búsquedas de literatura realizas al mes?**
   - <10: Custom Skill suficiente
   - 10-50: n8n justificado
   - 50+: n8n altamente recomendado

2. **¿Necesitas automatización?**
   - No: Custom Skill
   - Sí: n8n

3. **¿Trabajas en equipo?**
   - Solo: Custom Skill o n8n personal
   - Equipo: n8n definitivamente

4. **¿Cuál es tu presupuesto mensual?**
   - <$50: Custom Skill
   - $85-200: n8n configuración básica/media
   - $200+: n8n configuración completa

5. **¿Tienes conocimientos técnicos?**
   - Básicos: Custom Skill o n8n Cloud (fácil)
   - Intermedios: n8n self-hosted
   - Avanzados: n8n enterprise, custom nodes

---

## 🎬 Conclusión

La migración del proyecto de Custom Skill a n8n es **altamente factible** y ofrece ventajas significativas en términos de:

✅ **Automatización**
✅ **Escalabilidad**
✅ **Integraciones**
✅ **Colaboración**
✅ **Almacenamiento persistente**

Sin embargo, viene con:

⚠️ **Mayor complejidad técnica**
⚠️ **Costos más altos** ($85-175/mes vs $20/mes)
⚠️ **Tiempo de implementación** (4-6 semanas)

**Veredicto**: Si eres un investigador activo que realiza búsquedas frecuentes, trabaja en equipo, o necesita automatización, **vale totalmente la pena**.

Si solo necesitas ayuda ocasional con investigación, la Custom Skill de Claude Desktop es suficiente.

**Mi recomendación**: Empieza con Custom Skill, y si te encuentras usándola intensivamente (>10 veces/semana), considera la migración a n8n.

---

**Documento generado**: 2025-11-05

**Versión**: 1.0

**Autor**: Análisis técnico para proyecto de investigación académica
