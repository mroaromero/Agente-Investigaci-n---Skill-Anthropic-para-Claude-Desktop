# Custom Skill: Investigación Académica

Esta Custom Skill convierte a Claude Desktop en un asistente especializado en investigación académica de alto nivel.

## ¿Qué es una Custom Skill?

Las Custom Skills son instrucciones especializadas que le dan a Claude capacidades específicas. Esta skill está diseñada para:

- Búsqueda y análisis de literatura académica
- Desarrollo de hipótesis de investigación
- Citación en formato APA 7
- Escritura académica rigurosa
- Estructuración de documentos de investigación

## Instalación

### Paso 1: Localizar la carpeta de configuración de Claude Desktop

La ubicación depende de tu sistema operativo:

**macOS:**
```bash
~/Library/Application Support/Claude/
```

**Windows:**
```
%APPDATA%\Claude\
```

**Linux:**
```bash
~/.config/Claude/
```

### Paso 2: Crear la estructura de carpetas

Dentro de la carpeta de Claude, crea esta estructura si no existe:

```
Claude/
└── skills/
    └── research/
```

### Paso 3: Copiar el archivo de la skill

Copia el archivo `investigacion.md` de este repositorio a la carpeta de skills:

```bash
# En macOS/Linux
cp investigacion.md ~/Library/Application\ Support/Claude/skills/research/

# En Windows (PowerShell)
Copy-Item investigacion.md "$env:APPDATA\Claude\skills\research\"
```

O manualmente:
1. Abre la carpeta `Claude/skills/research/`
2. Copia el archivo `investigacion.md` dentro

### Paso 4: Reiniciar Claude Desktop

Cierra completamente Claude Desktop y vuelve a abrirlo para que cargue la nueva skill.

## Cómo Usar la Skill

### Activación Manual

En Claude Desktop, escribe:

```
/skill research
```

o

```
/skill investigacion
```

### Comandos Disponibles

Una vez activada la skill, puedes usar estos comandos especiales:

- `BUSCAR: [tema]` - Búsqueda exhaustiva de literatura
- `HIPÓTESIS: [descripción]` - Desarrollo de hipótesis
- `CITAR: [información]` - Generación de citación APA 7
- `REVISAR: [texto]` - Revisión de texto académico
- `ESTRUCTURA: [tipo]` - Ayuda con estructura de documentos

### Ejemplos de Uso

**Ejemplo 1: Búsqueda de Literatura**
```
BUSCAR: machine learning en educación
```

**Ejemplo 2: Desarrollo de Hipótesis**
```
HIPÓTESIS: Quiero investigar si el uso de gamificación mejora el aprendizaje de matemáticas en estudiantes de secundaria
```

**Ejemplo 3: Citación APA 7**
```
CITAR: Artículo de María García publicado en 2023 en la Revista de Educación, volumen 15, páginas 45-67, sobre metodologías activas
```

**Ejemplo 4: Revisión de Texto**
```
REVISAR: La inteligencia artificial está cambiando la educación. Es muy importante porque ayuda a los estudiantes.
```

## Características Principales

### 1. Búsqueda y Análisis
- Búsquedas exhaustivas de literatura
- Identificación de papers clave
- Análisis de gaps en investigación
- Resúmenes críticos

### 2. Desarrollo de Hipótesis
- Formulación de hipótesis basadas en evidencia
- Identificación de variables
- Sugerencias metodológicas
- Evaluación de viabilidad

### 3. Citación APA 7
- Generación de citas correctas
- Listas de referencias completas
- Verificación de formato
- Paráfrasis apropiadas

### 4. Escritura Académica
- Construcción de párrafos académicos
- Conectores y transiciones
- Claridad y precisión
- Tono académico y objetivo

### 5. Estructura de Investigación
- Organización de documentos
- Secciones académicas estándar
- Argumentos coherentes
- Esquemas de trabajo

## Recursos del Proyecto

Esta skill tiene acceso a los siguientes recursos en el repositorio:

- `Academic-Phrasebank-2023-ES.pdf`: Banco de frases académicas en español
- Guías de citación APA 7 (`05_apa-7-citacion-rigurosa.md`)
- Plantillas de tesis (`07_plantilla-tesis.md`)
- Guías de construcción de párrafos (`09_construccion-parrafos-academicos.md`)
- Recursos sobre hipótesis (`10_hipotesis-y-recursos-retoricos.md`)

## Principios de Trabajo

La skill opera bajo estos principios:

1. **Rigor Académico**: Máximo estándar de precisión
2. **Citación Apropiada**: Evita plagio
3. **Pensamiento Crítico**: Evaluación crítica de información
4. **Claridad**: Comunicación clara de ideas complejas
5. **Objetividad**: Enfoque imparcial basado en evidencia

## Configuración Avanzada

### Personalización

Puedes editar el archivo `investigacion.md` para personalizar:

- Estilos de citación (agregar otros además de APA 7)
- Niveles académicos específicos
- Áreas de especialización
- Comandos personalizados

### Integración con MCP

Si usas Model Context Protocol (MCP), puedes integrar esta skill con:

- Bases de datos académicas
- Gestores de referencias (Zotero, Mendeley)
- Herramientas de búsqueda (Google Scholar, PubMed)

## Solución de Problemas

### La skill no aparece
- Verifica que el archivo esté en la ubicación correcta
- Asegúrate de haber reiniciado Claude Desktop
- Revisa que el archivo tenga extensión `.md`

### Los comandos no funcionan
- Activa primero la skill con `/skill research`
- Usa el formato exacto de los comandos (ej: `BUSCAR:` no `buscar`)

### La skill no tiene acceso a recursos
- Asegúrate de estar en el directorio correcto del proyecto
- Verifica que los archivos PDF y MD estén disponibles

## Contribuir

Si deseas mejorar esta skill:

1. Haz un fork del repositorio
2. Crea una rama para tus cambios
3. Realiza tus modificaciones
4. Envía un pull request

## Licencia

Este proyecto está bajo [tu licencia aquí].

## Contacto

Para preguntas o sugerencias sobre esta Custom Skill, [agrega información de contacto].

---

**Versión**: 1.0.0
**Última actualización**: Noviembre 2025
**Compatible con**: Claude Desktop (todas las versiones con soporte de Custom Skills)
