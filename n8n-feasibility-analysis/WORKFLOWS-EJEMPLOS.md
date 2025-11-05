# Workflows de Ejemplo para n8n - Sistema de Investigación Académica

Este documento contiene ejemplos detallados de workflows de n8n listos para implementar.

---

## 📋 Índice de Workflows

1. [Búsqueda Simple en Semantic Scholar](#workflow-1-búsqueda-simple-en-semantic-scholar)
2. [Búsqueda Multi-API con Análisis Claude](#workflow-2-búsqueda-multi-api-con-análisis-claude)
3. [Generador de Citaciones APA 7](#workflow-3-generador-de-citaciones-apa-7)
4. [Revisor de Texto Académico](#workflow-4-revisor-de-texto-académico)
5. [Monitor Automático de Nuevas Publicaciones](#workflow-5-monitor-automático-de-nuevas-publicaciones)

---

## Workflow 1: Búsqueda Simple en Semantic Scholar

### Descripción
Workflow básico que busca papers en Semantic Scholar y retorna resultados formateados.

### Configuración

**Trigger**: Webhook (POST)

**Input esperado**:
```json
{
  "query": "machine learning in education",
  "limit": 20,
  "fields": "paperId,title,abstract,authors,year,citationCount,url"
}
```

### Nodos del Workflow

#### 1. Webhook (Trigger)
```
- Method: POST
- Path: /search-literature
- Response Mode: Last Node
```

#### 2. HTTP Request - Semantic Scholar
```
- Method: GET
- URL: https://api.semanticscholar.org/graph/v1/paper/search
- Parameters:
  * query: {{ $json.query }}
  * limit: {{ $json.limit }}
  * fields: {{ $json.fields }}
- Authentication: None (o API Key si tienes)
```

#### 3. Code Node - Format Results
```javascript
// Formatear resultados de Semantic Scholar
const papers = $input.all()[0].json.data;

const formattedPapers = papers.map(paper => ({
  id: paper.paperId,
  title: paper.title,
  authors: paper.authors.map(a => a.name).join(', '),
  year: paper.year,
  citations: paper.citationCount,
  abstract: paper.abstract || 'No abstract available',
  url: paper.url || `https://www.semanticscholar.org/paper/${paper.paperId}`,
  relevanceScore: paper.citationCount / (2025 - paper.year + 1) // Simple relevance metric
}));

// Ordenar por relevancia
formattedPapers.sort((a, b) => b.relevanceScore - a.relevanceScore);

return formattedPapers.map(paper => ({ json: paper }));
```

#### 4. Response (Webhook Response)
```
- Response Body: {{ $json }}
```

### Ejemplo de Uso

**Request**:
```bash
curl -X POST https://tu-n8n-instance.com/webhook/search-literature \
  -H "Content-Type: application/json" \
  -d '{
    "query": "deep learning natural language processing",
    "limit": 10,
    "fields": "paperId,title,abstract,authors,year,citationCount,url"
  }'
```

**Response**:
```json
[
  {
    "id": "abc123",
    "title": "BERT: Pre-training of Deep Bidirectional Transformers",
    "authors": "Jacob Devlin, Ming-Wei Chang, Kenton Lee",
    "year": 2019,
    "citations": 15420,
    "abstract": "We introduce BERT, a new language representation model...",
    "url": "https://www.semanticscholar.org/paper/abc123",
    "relevanceScore": 2570
  }
]
```

---

## Workflow 2: Búsqueda Multi-API con Análisis Claude

### Descripción
Workflow avanzado que busca en múltiples APIs académicas en paralelo, deduplica resultados, y usa Claude para análisis crítico.

### Diagrama de Flujo

```
Webhook Trigger
    │
    ▼
Split Into Batches (3 APIs en paralelo)
    │
    ├─► HTTP Request: Semantic Scholar
    ├─► HTTP Request: Crossref
    └─► HTTP Request: OpenAlex
    │
    ▼
Merge Results
    │
    ▼
Code: Deduplicate by DOI/Title
    │
    ▼
Code: Rank by Relevance
    │
    ▼
Take Top 20 Papers
    │
    ▼
Claude AI Agent: Analyze & Summarize
    │
    ▼
PostgreSQL: Store Results
    │
    ▼
Google Docs: Generate Report
    │
    ▼
Webhook Response
```

### Configuración Detallada

#### Nodo 1: Webhook Trigger
```json
{
  "method": "POST",
  "path": "research-search-advanced",
  "responseMode": "lastNode"
}
```

#### Nodo 2: Set Variables
```javascript
// Extraer y preparar parámetros de búsqueda
const query = $json.query;
const searchDate = new Date().toISOString();

return {
  query: query,
  semanticScholarQuery: query,
  crossrefQuery: query.replace(/\s+/g, '+'),
  openAlexQuery: encodeURIComponent(query),
  searchDate: searchDate,
  userId: $json.userId || 'anonymous'
};
```

#### Nodo 3: Split In Batches (for parallel execution)
```
- Batch Size: 1
- Options: Reset
```

#### Nodos 4-6: HTTP Requests (En paralelo)

**4a. Semantic Scholar API**
```
- URL: https://api.semanticscholar.org/graph/v1/paper/search
- Parameters:
  * query: {{ $json.semanticScholarQuery }}
  * limit: 50
  * fields: paperId,externalIds,title,abstract,authors,year,citationCount,publicationDate,journal,url
```

**4b. Crossref API**
```
- URL: https://api.crossref.org/works
- Parameters:
  * query: {{ $json.crossrefQuery }}
  * rows: 50
  * select: DOI,title,author,published,abstract,container-title,URL
```

**4c. OpenAlex API**
```
- URL: https://api.openalex.org/works
- Parameters:
  * search: {{ $json.openAlexQuery }}
  * per-page: 50
  * select: id,doi,title,display_name,publication_year,cited_by_count,abstract_inverted_index,authorships
```

#### Nodo 7: Merge (Combine results from 3 APIs)
```
- Mode: Merge By Index
```

#### Nodo 8: Code - Deduplicate & Normalize
```javascript
// Obtener todos los resultados de las 3 APIs
const semanticResults = $input.first().json.data || [];
const crossrefResults = $input.all()[1]?.json?.message?.items || [];
const openAlexResults = $input.all()[2]?.json?.results || [];

// Función para normalizar autores
function normalizeAuthors(paper, source) {
  if (source === 'semantic') {
    return paper.authors?.map(a => a.name) || [];
  } else if (source === 'crossref') {
    return paper.author?.map(a => `${a.given} ${a.family}`) || [];
  } else if (source === 'openalex') {
    return paper.authorships?.map(a => a.author.display_name) || [];
  }
  return [];
}

// Función para normalizar abstract
function normalizeAbstract(paper, source) {
  if (source === 'semantic') {
    return paper.abstract || '';
  } else if (source === 'crossref') {
    return paper.abstract || '';
  } else if (source === 'openalex') {
    // OpenAlex usa inverted index - convertir a texto
    const inverted = paper.abstract_inverted_index || {};
    const words = [];
    Object.keys(inverted).forEach(word => {
      inverted[word].forEach(pos => {
        words[pos] = word;
      });
    });
    return words.join(' ');
  }
  return '';
}

// Normalizar todos los papers
const allPapers = [];

// Semantic Scholar
semanticResults.forEach(paper => {
  allPapers.push({
    source: 'SemanticScholar',
    doi: paper.externalIds?.DOI || null,
    title: paper.title,
    authors: normalizeAuthors(paper, 'semantic'),
    year: paper.year,
    abstract: normalizeAbstract(paper, 'semantic'),
    citationCount: paper.citationCount || 0,
    url: paper.url || `https://www.semanticscholar.org/paper/${paper.paperId}`,
    journal: paper.journal?.name || null,
    publicationDate: paper.publicationDate || null
  });
});

// Crossref
crossrefResults.forEach(paper => {
  allPapers.push({
    source: 'Crossref',
    doi: paper.DOI || null,
    title: paper.title?.[0] || '',
    authors: normalizeAuthors(paper, 'crossref'),
    year: paper.published?.['date-parts']?.[0]?.[0] || null,
    abstract: normalizeAbstract(paper, 'crossref'),
    citationCount: 0, // Crossref no provee citation count
    url: paper.URL || `https://doi.org/${paper.DOI}`,
    journal: paper['container-title']?.[0] || null,
    publicationDate: null
  });
});

// OpenAlex
openAlexResults.forEach(paper => {
  allPapers.push({
    source: 'OpenAlex',
    doi: paper.doi?.replace('https://doi.org/', '') || null,
    title: paper.display_name || paper.title,
    authors: normalizeAuthors(paper, 'openalex'),
    year: paper.publication_year,
    abstract: normalizeAbstract(paper, 'openalex'),
    citationCount: paper.cited_by_count || 0,
    url: paper.doi || paper.id,
    journal: null,
    publicationDate: null
  });
});

// Deduplicar por DOI y título
const uniquePapers = [];
const seenDOIs = new Set();
const seenTitles = new Set();

allPapers.forEach(paper => {
  // Normalizar título para comparación
  const normalizedTitle = paper.title.toLowerCase().replace(/[^\w\s]/g, '').trim();

  // Chequear duplicados
  const isDuplicateDOI = paper.doi && seenDOIs.has(paper.doi);
  const isDuplicateTitle = seenTitles.has(normalizedTitle);

  if (!isDuplicateDOI && !isDuplicateTitle) {
    if (paper.doi) seenDOIs.add(paper.doi);
    seenTitles.add(normalizedTitle);

    // Calcular relevance score
    const currentYear = 2025;
    const age = currentYear - (paper.year || currentYear);
    const citationScore = paper.citationCount || 0;
    const recencyBonus = Math.max(0, 5 - age); // Papers recientes bonus

    paper.relevanceScore = (citationScore * 0.7) + (recencyBonus * 10);

    uniquePapers.push(paper);
  }
});

// Ordenar por relevancia
uniquePapers.sort((a, b) => b.relevanceScore - a.relevanceScore);

// Retornar top 20
return uniquePapers.slice(0, 20).map(paper => ({ json: paper }));
```

#### Nodo 9: Claude AI Agent - Análisis Crítico

**Configuración**:
```
- Model: Claude 3.5 Sonnet
- Temperature: 0.3 (más determinista)
- Max Tokens: 4000
```

**System Prompt**:
```
Eres un investigador académico experto. Tu tarea es analizar un conjunto de papers científicos y proporcionar un análisis crítico estructurado.

Para cada paper, debes:
1. Identificar la contribución principal
2. Evaluar la relevancia para el tema de búsqueda
3. Identificar metodologías utilizadas
4. Detectar limitaciones o gaps

Al final, proporciona:
- Resumen de tendencias principales
- Gaps de investigación identificados
- Papers más relevantes (top 5)
- Recomendaciones de líneas de investigación

Formato de salida: JSON estructurado
```

**User Prompt**:
```javascript
`Analiza estos ${$input.all().length} papers sobre el tema: "${$('Webhook').item.json.query}"

Papers:
${$input.all().map((item, i) => `
${i+1}. ${item.json.title}
   Autores: ${item.json.authors.slice(0, 3).join(', ')}${item.json.authors.length > 3 ? ' et al.' : ''}
   Año: ${item.json.year}
   Citas: ${item.json.citationCount}
   Abstract: ${item.json.abstract?.substring(0, 300)}...
   URL: ${item.json.url}
`).join('\n')}

Proporciona tu análisis en formato JSON con esta estructura:
{
  "overview": "Resumen general de las tendencias",
  "topPapers": [
    {
      "rank": 1,
      "title": "...",
      "reason": "Por qué es relevante"
    }
  ],
  "gaps": ["gap 1", "gap 2"],
  "recommendations": ["recomendación 1", "recomendación 2"],
  "methodologies": ["método 1", "método 2"]
}
`
```

#### Nodo 10: Code - Parse Claude Response
```javascript
// Claude retorna texto, necesitamos extraer el JSON
const claudeResponse = $json.content || $json.text || $json.response;

// Intentar parsear JSON
let analysis;
try {
  // Buscar JSON en la respuesta (puede estar envuelto en ```json```)
  const jsonMatch = claudeResponse.match(/\{[\s\S]*\}/);
  if (jsonMatch) {
    analysis = JSON.parse(jsonMatch[0]);
  } else {
    analysis = JSON.parse(claudeResponse);
  }
} catch (e) {
  // Si falla, crear estructura básica
  analysis = {
    overview: claudeResponse,
    topPapers: [],
    gaps: [],
    recommendations: [],
    methodologies: []
  };
}

// Combinar análisis con papers originales
const papers = $input.all().slice(0, -1); // Todos excepto el último (Claude response)

return {
  searchQuery: $('Webhook').item.json.query,
  searchDate: new Date().toISOString(),
  totalPapersFound: papers.length,
  analysis: analysis,
  papers: papers.map(p => p.json)
};
```

#### Nodo 11: PostgreSQL - Store Results
```sql
INSERT INTO research_searches (
  user_id,
  query,
  search_date,
  total_results,
  analysis,
  papers
) VALUES (
  '{{ $('Set Variables').item.json.userId }}',
  '{{ $json.searchQuery }}',
  '{{ $json.searchDate }}',
  {{ $json.totalPapersFound }},
  '{{ JSON.stringify($json.analysis) }}',
  '{{ JSON.stringify($json.papers) }}'
) RETURNING id;
```

#### Nodo 12: Google Docs - Generate Report

**Configuración**:
- Operation: Create a new document
- Title: `Reporte de Literatura - {{ $json.searchQuery }} - {{ $json.searchDate }}`

**Content Template**:
```
# Reporte de Búsqueda de Literatura

**Consulta**: {{ $json.searchQuery }}
**Fecha**: {{ $json.searchDate }}
**Papers encontrados**: {{ $json.totalPapersFound }}

---

## Resumen Ejecutivo

{{ $json.analysis.overview }}

---

## Papers Más Relevantes

{{#each $json.analysis.topPapers}}
### {{ this.rank }}. {{ this.title }}

**Justificación**: {{ this.reason }}

{{/each}}

---

## Gaps de Investigación Identificados

{{#each $json.analysis.gaps}}
- {{ this }}
{{/each}}

---

## Recomendaciones

{{#each $json.analysis.recommendations}}
- {{ this }}
{{/each}}

---

## Metodologías Utilizadas en la Literatura

{{#each $json.analysis.methodologies}}
- {{ this }}
{{/each}}

---

## Listado Completo de Papers

{{#each $json.papers}}
### {{ @index }}. {{ this.title }}

**Autores**: {{ this.authors.join(', ') }}
**Año**: {{ this.year }}
**Citaciones**: {{ this.citationCount }}
**Fuente**: {{ this.source }}
**URL**: {{ this.url }}

**Abstract**:
{{ this.abstract }}

---
{{/each}}

---

*Reporte generado automáticamente por el Sistema de Investigación Académica*
```

#### Nodo 13: Webhook Response
```json
{
  "success": true,
  "searchId": "{{ $('PostgreSQL').item.json.id }}",
  "reportUrl": "{{ $('Google Docs').item.json.documentUrl }}",
  "summary": {
    "query": "{{ $json.searchQuery }}",
    "papersFound": "{{ $json.totalPapersFound }}",
    "topPapers": "{{ $json.analysis.topPapers.length }}"
  },
  "message": "Búsqueda completada exitosamente. Reporte disponible en Google Docs."
}
```

### Tiempo de Ejecución Estimado
- APIs paralelas: 5-10 segundos
- Deduplicación: 1-2 segundos
- Análisis Claude: 30-60 segundos
- Storage + Google Docs: 5-10 segundos
- **Total: 40-80 segundos**

### Costo por Ejecución
- APIs: $0 (gratuitas)
- Claude: ~100K tokens = $0.30-0.45
- n8n: 1 ejecución
- **Total: ~$0.35**

---

## Workflow 3: Generador de Citaciones APA 7

### Descripción
Toma un DOI, URL o metadatos parciales y genera citación en formato APA 7.

### Nodos

#### 1. Webhook Trigger
```json
{
  "method": "POST",
  "path": "generate-citation",
  "input": {
    "doi": "10.1234/example.2023.001",
    "OR": "url": "https://example.com/paper",
    "OR": "manual": {
      "authors": ["Smith, J.", "Doe, A."],
      "year": 2023,
      "title": "Example Paper",
      "journal": "Journal of Examples",
      "volume": 10,
      "issue": 2,
      "pages": "123-145"
    }
  }
}
```

#### 2. Switch Node - Determine Input Type
```
- Mode: Based on rules
- Rules:
  * If {{ $json.doi }} exists → Route: DOI
  * If {{ $json.url }} exists → Route: URL
  * Else → Route: Manual
```

#### 3a. HTTP Request - Fetch from DOI (Crossref)
```
- URL: https://api.crossref.org/works/{{ $json.doi }}
- Method: GET
```

#### 3b. Code - Extract DOI from URL (if needed)
```javascript
// Intentar extraer DOI de URL
const url = $json.url;
const doiMatch = url.match(/10\.\d{4,9}\/[-._;()\/:a-zA-Z0-9]+/);

if (doiMatch) {
  return { doi: doiMatch[0] };
} else {
  // No DOI found, intentar web scraping o manual input
  return { error: 'No DOI found in URL' };
}
```

#### 4. Code - Format APA 7 Citation
```javascript
// Obtener metadata del paper
const metadata = $json.message || $json;

// Extraer campos necesarios
const authors = metadata.author || [];
const year = metadata.published?.['date-parts']?.[0]?.[0] || 'n.d.';
const title = metadata.title?.[0] || '';
const journal = metadata['container-title']?.[0] || '';
const volume = metadata.volume || '';
const issue = metadata.issue || '';
const pages = metadata.page || '';
const doi = metadata.DOI || '';

// Formatear autores según APA 7
function formatAuthors(authorList) {
  if (authorList.length === 0) return 'Author Unknown';
  if (authorList.length === 1) {
    return `${authorList[0].family}, ${authorList[0].given[0]}.`;
  }
  if (authorList.length === 2) {
    return `${authorList[0].family}, ${authorList[0].given[0]}., & ${authorList[1].family}, ${authorList[1].given[0]}.`;
  }
  if (authorList.length <= 20) {
    const formatted = authorList.slice(0, -1).map(a => `${a.family}, ${a.given[0]}.`).join(', ');
    const last = authorList[authorList.length - 1];
    return `${formatted}, & ${last.family}, ${last.given[0]}.`;
  }
  // Más de 20 autores - APA 7 ellipsis rule
  const first19 = authorList.slice(0, 19).map(a => `${a.family}, ${a.given[0]}.`).join(', ');
  const last = authorList[authorList.length - 1];
  return `${first19}, ... ${last.family}, ${last.given[0]}.`;
}

const formattedAuthors = formatAuthors(authors);

// Construir citación completa
let reference = `${formattedAuthors} (${year}). ${title}. `;

if (journal) {
  reference += `*${journal}*`;
  if (volume) reference += `, *${volume}*`;
  if (issue) reference += `(${issue})`;
  if (pages) reference += `, ${pages}`;
  reference += '.';
}

if (doi) {
  reference += ` https://doi.org/${doi}`;
}

// Generar variantes de citación in-text
const firstAuthor = authors[0];
const inTextNarrative = authors.length === 1
  ? `${firstAuthor.family} (${year})`
  : authors.length === 2
  ? `${authors[0].family} & ${authors[1].family} (${year})`
  : `${firstAuthor.family} et al. (${year})`;

const inTextParenthetical = authors.length === 1
  ? `(${firstAuthor.family}, ${year})`
  : authors.length === 2
  ? `(${authors[0].family} & ${authors[1].family}, ${year})`
  : `(${firstAuthor.family} et al., ${year})`;

return {
  reference: reference,
  inTextNarrative: inTextNarrative,
  inTextParenthetical: inTextParenthetical,
  metadata: {
    authors: formattedAuthors,
    year: year,
    title: title,
    journal: journal,
    doi: doi
  }
};
```

#### 5. Webhook Response
```json
{
  "success": true,
  "citation": {
    "reference": "{{ $json.reference }}",
    "inText": {
      "narrative": "{{ $json.inTextNarrative }}",
      "parenthetical": "{{ $json.inTextParenthetical }}"
    },
    "format": "APA 7th Edition"
  },
  "metadata": "{{ $json.metadata }}"
}
```

### Ejemplo de Output

```json
{
  "success": true,
  "citation": {
    "reference": "Smith, J., Doe, A., & Johnson, M. (2023). The impact of machine learning on education. *Journal of Educational Technology*, *45*(3), 234-251. https://doi.org/10.1234/jet.2023.001",
    "inText": {
      "narrative": "Smith et al. (2023)",
      "parenthetical": "(Smith et al., 2023)"
    },
    "format": "APA 7th Edition"
  },
  "metadata": {
    "authors": "Smith, J., Doe, A., & Johnson, M.",
    "year": 2023,
    "title": "The impact of machine learning on education",
    "journal": "Journal of Educational Technology",
    "doi": "10.1234/jet.2023.001"
  }
}
```

---

## Workflow 4: Revisor de Texto Académico

Ver documento FACTIBILIDAD-TECNICA.md para detalles completos. Resumen:

**Entrada**: Texto académico (hasta 5000 palabras)

**Proceso**:
1. Split texto en chunks
2. Claude análisis multi-pass (gramática, estilo, argumentación)
3. Query Academic Phrasebank para sugerencias
4. Generar reporte con cambios sugeridos

**Salida**: Documento con texto mejorado y justificación de cambios

---

## Workflow 5: Monitor Automático de Nuevas Publicaciones

### Trigger: Cron (Diario a las 8 AM)

### Flujo:

1. **Load User Preferences** (PostgreSQL)
   - Obtener topics de interés de cada usuario
   - Últimas fechas de búsqueda

2. **For Each Topic** (Loop)
   - Buscar en Semantic Scholar (filter: last 24h)
   - Buscar en arXiv (filter: last 24h)

3. **Filter & Rank**
   - Eliminar ya vistos (check DB)
   - Rankear por relevancia
   - Top 5 por topic

4. **Claude Summarization**
   - Generar resumen de 2-3 oraciones por paper

5. **Compile Email Digest**
   - Agrupar por topic
   - Formatear HTML email

6. **Send Email** (Gmail/SendGrid node)

7. **Update DB** (marcar papers como notificados)

### Código del Nodo de Búsqueda Diaria

```javascript
// Calcular fecha de hace 24 horas
const yesterday = new Date();
yesterday.setDate(yesterday.getDate() - 1);
const dateFilter = yesterday.toISOString().split('T')[0]; // YYYY-MM-DD

// Topics del usuario
const topics = $json.interests; // ['machine learning', 'education technology']

// Buscar papers nuevos para cada topic
const newPapers = [];

for (const topic of topics) {
  // Aquí iría la llamada a Semantic Scholar API con date filter
  // Ejemplo simplificado:
  const searchResults = []; // Resultados de API

  searchResults.forEach(paper => {
    if (paper.publicationDate >= dateFilter) {
      newPapers.push({
        topic: topic,
        ...paper
      });
    }
  });
}

return newPapers.map(p => ({ json: p }));
```

---

## 🛠️ Herramientas de Debugging y Testing

### Test Node Configuration

Para testear cada nodo individualmente:

```javascript
// Test Data Node
return [
  {
    json: {
      query: "artificial intelligence in healthcare",
      limit: 10,
      userId: "test-user-123"
    }
  }
];
```

### Error Handling Pattern

Agregar error handling a cada HTTP request:

```javascript
// En "Continue On Fail" settings:
- Continue On Fail: true

// Luego agregar un IF node:
if ($json.error || !$json.data) {
  return {
    error: true,
    message: 'API request failed',
    details: $json
  };
}
```

---

## 📚 Próximos Pasos

1. **Importar workflows** a tu instancia de n8n
2. **Configurar credenciales** (Claude API, PostgreSQL, etc.)
3. **Testear cada workflow** individualmente
4. **Ajustar timeouts** y rate limits según necesidad
5. **Monitorear costos** y optimizar

---

**Nota**: Estos son ejemplos funcionales que requieren ajustes según tu configuración específica de n8n, APIs y bases de datos.

Para workflows completos en formato JSON importable a n8n, ver carpeta `n8n-workflows/` (próximamente).
