# Instalación Rápida - Custom Skill Investigación Académica

## 🚀 Instalación en 3 Pasos

### Paso 1: Localiza tu carpeta de Claude Desktop

Abre tu terminal y ejecuta el comando según tu sistema operativo:

**macOS:**
```bash
open ~/Library/Application\ Support/Claude/
```

**Linux:**
```bash
xdg-open ~/.config/Claude/
```

**Windows (PowerShell):**
```powershell
explorer "$env:APPDATA\Claude\"
```

### Paso 2: Crea la estructura de carpetas

Dentro de la carpeta de Claude, crea esta estructura:

```
Claude/
└── skills/
    └── research/
```

**Comando rápido:**

**macOS/Linux:**
```bash
mkdir -p ~/Library/Application\ Support/Claude/skills/research/
```

**Windows (PowerShell):**
```powershell
New-Item -ItemType Directory -Force -Path "$env:APPDATA\Claude\skills\research\"
```

### Paso 3: Copia el archivo de la skill

**macOS:**
```bash
cp investigacion.md ~/Library/Application\ Support/Claude/skills/research/
```

**Linux:**
```bash
cp investigacion.md ~/.config/Claude/skills/research/
```

**Windows (PowerShell):**
```powershell
Copy-Item investigacion.md "$env:APPDATA\Claude\skills\research\"
```

### Paso 4: Reinicia Claude Desktop

1. Cierra completamente Claude Desktop
2. Vuelve a abrirlo
3. Escribe `/skill research` para activar la skill

## ✅ Verificación

Para verificar que la skill está instalada correctamente:

1. Abre Claude Desktop
2. Escribe: `/skills` o `/skill` (según la versión)
3. Deberías ver "research" o "investigacion" en la lista

## 🎯 Primer Uso

Prueba la skill con este comando:

```
BUSCAR: metodología de investigación cualitativa
```

Deberías recibir una respuesta estructurada con búsqueda de literatura.

## ❓ Problemas Comunes

### La skill no aparece

**Solución 1:** Verifica la ubicación del archivo
```bash
# macOS
ls ~/Library/Application\ Support/Claude/skills/research/

# Debe mostrar: investigacion.md
```

**Solución 2:** Verifica que el archivo no tenga extensión doble
- ❌ `investigacion.md.txt`
- ✅ `investigacion.md`

**Solución 3:** Reinicia Claude Desktop completamente
- Cierra todas las ventanas
- En macOS: Cmd+Q
- En Windows: Cierra desde el administrador de tareas si es necesario

### La skill se activa pero no responde bien

**Solución:** Verifica que el archivo se copió completamente
```bash
# El archivo debe tener aproximadamente 4-5 KB
ls -lh investigacion.md
```

### Los comandos no funcionan

**Recuerda:**
- Primero activa la skill: `/skill research`
- Luego usa los comandos: `BUSCAR:`, `HIPÓTESIS:`, etc.
- Los comandos son sensibles a mayúsculas/minúsculas

## 📚 Siguiente Paso

Lee el archivo `EJEMPLOS.md` para ver casos de uso prácticos y dominar la skill.

## 🆘 Soporte

Si sigues teniendo problemas:

1. Verifica que estés usando una versión de Claude Desktop compatible con Custom Skills
2. Revisa la documentación oficial de Claude Desktop
3. Consulta el archivo README.md completo para más detalles

---

**Tiempo estimado de instalación:** 2-3 minutos

**¡Listo para investigar con IA!** 🎓✨
