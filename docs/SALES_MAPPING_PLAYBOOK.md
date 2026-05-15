# Playbook: Mapeo de Proyecto de Ventas Nacionales

## 1. Preparación del Entorno
Ejecuta estos comandos en tu terminal local antes de iniciar:
```bash
# Asegurar que I: esté montada
sudo mount -t drvfs I: /mnt/i 2>/dev/null
# Iniciar stack local
./cc-init-ai.sh
```

## 2. Instrucción para Gemini (Estrategia)
Pega esto en la nueva sesión de Gemini:
> "Actúa como el Central Command Orchestrator. Mi objetivo es mapear el proyecto de ventas en: `/mnt/i/Mi unidad/grupo_del_valle/scripts python/reportes_ventas_nacional`. 
> 
> Analiza las dependencias y genera un comando de delegación para que el modelo local **qwen2.5-coder:14b** (puerto 11435) cree un documento `SALES_ARCH.md` con la estructura de carpetas, los módulos de Python principales y las conexiones a base de datos."

## 3. Ejecución Local (El "Músculo")
Gemini te dará un comando similar a este, el cual debes correr en tu terminal:
```bash
./cc-run.sh --local "Analiza '/mnt/i/Mi unidad/grupo_del_valle/scripts python/reportes_ventas_nacional' y documenta en SALES_ARCH.md: 1. Estructura de archivos. 2. Lógica del consolidado. 3. Puntos de integración con SQL/DuckDB."
```

## 4. Resultado Esperado
Al terminar, tendrás un archivo llamado `SALES_ARCH.md` en tu raíz con todo el conocimiento técnico del proyecto de ventas, generado localmente y sin costo.
