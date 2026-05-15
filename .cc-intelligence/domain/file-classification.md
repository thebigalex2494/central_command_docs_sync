---
name: File Classification
description: This skill should be used when Claude needs to classify files by type, organize files into categories, apply the classification taxonomy, determine file destinations, or suggest file organization actions.
version: 1.0.0
---

# File Classification Skill

## Overview

This skill provides the complete taxonomy and classification logic for organizing files within the Central Agent system.

## Classification Taxonomy

### Code Files

| Extension | Language | Category | Destination |
|-----------|----------|----------|-------------|
| .ps1 | PowerShell | Scripts | Projects/scripts/maintenance |
| .py | Python | Tools | Projects/code/tools |
| .ts, .tsx | TypeScript | Web | Projects/code/web |
| .js, .jsx | JavaScript | Web | Projects/code/web |
| .cs | C# | Apps | Projects/code/apps |
| .go | Go | Backend | Projects/code/backend |
| .rs | Rust | Systems | Projects/code/systems |
| .json | JSON | Config | Projects/config |
| .yaml, .yml | YAML | Config | Projects/config |
| .xml | XML | Config | Projects/config |
| .md | Markdown | Docs | Projects/docs |
| .sql | SQL | Database | Projects/code/database |

### Document Files

| Pattern | Category | Destination |
|---------|----------|-------------|
| *factura*, *invoice* | Financial | Documents/Financial/Invoices |
| *estado*cuenta*, *statement* | Financial | Documents/Financial/Statements |
| *cotizacion*, *quote*, *presupuesto* | Financial | Documents/Financial/Quotes |
| *contrato*, *contract*, *agreement* | Legal | Documents/Legal/Contracts |
| *acuerdo*, *convenio* | Legal | Documents/Legal/Agreements |
| *reporte*, *report*, *informe* | Reports | Documents/Reports |
| *manual*, *guide*, *tutorial* | Manuals | Documents/Manuals |
| *cv*, *resume*, *curriculum* | Personal | Documents/Personal/CV |

### Media Files

| Extension | Type | Destination |
|-----------|------|-------------|
| .jpg, .jpeg | Image | Pictures |
| .png | Image | Pictures |
| .gif | Image/Animated | Pictures |
| .webp | Image/Web | Pictures |
| .svg | Vector | Pictures/Vectors |
| .ico | Icon | Pictures/Icons |
| .mp4, .mov, .avi | Video | Videos |
| .mp3, .wav, .flac | Audio | Music |
| .psd | Photoshop | Pictures/Source |
| .ai | Illustrator | Pictures/Source |

### Archive Files

| Extension | Type | Action |
|-----------|------|--------|
| .zip | ZIP Archive | Downloads/Archives |
| .rar | RAR Archive | Downloads/Archives |
| .7z | 7-Zip Archive | Downloads/Archives |
| .tar, .gz, .tar.gz | TAR Archive | Downloads/Archives |

### Executable Files

| Pattern | Type | Action |
|---------|------|--------|
| *Setup*.exe | Installer | Downloads/Installers |
| *Installer*.exe | Installer | Downloads/Installers |
| *Install*.exe | Installer | Downloads/Installers |
| *Portable*.exe | Portable App | Downloads/Portable |
| .msi | MSI Installer | Downloads/Installers |
| .iso | Disk Image | Downloads/ISOs |
| .img | Disk Image | Downloads/ISOs |

## Classification Algorithm

```
function ClassifyFile(file):
    1. Get file extension
    2. Check code file extensions → return code category
    3. Check document extensions:
       a. If .pdf, .docx, .xlsx → check filename patterns
       b. Apply pattern matching for subcategories
    4. Check media extensions → return media category
    5. Check archive extensions → return archive category
    6. Check executable patterns → return executable category
    6. If no match → return "Uncategorized"

    Return {
        category: string,
        subcategory: string,
        destination: string,
        confidence: high|medium|low,
        action: move|review|skip
    }
```

## Classification Result Format

```json
{
  "file": "factura_enero_2026.pdf",
  "classification": {
    "category": "Documents",
    "subcategory": "Financial/Invoices",
    "destination": "%HOME%\\Documents\\Financial\\Invoices",
    "confidence": "high",
    "matched_pattern": "*factura*"
  },
  "action": {
    "type": "move",
    "source": "%HOME%\\Downloads\\factura_enero_2026.pdf",
    "target": "%HOME%\\Documents\\Financial\\Invoices\\factura_enero_2026.pdf"
  }
}
```

## Special Rules

### Priority Order
1. Check for sensitive patterns first (credentials, keys) → Flag for review
2. Check executable files → Apply security consideration
3. Check by extension
4. Check by filename pattern
5. Fall back to uncategorized

### Sensitive File Detection
Flag these patterns for manual review:
- `*password*`, `*credential*`, `*secret*`
- `*.pem`, `*.key`, `*.pfx`
- `*api_key*`, `*token*`
- `.env`, `*.env.*`

### Large File Handling
Files > 100MB:
- Log separately in large-files.log
- Suggest compression or archival
- Don't auto-move without confirmation

### Duplicate Detection
When moving files:
1. Check if destination exists
2. If exists, compare size and date
3. If identical → skip, log as duplicate
4. If different → rename with suffix `_1`, `_2`, etc.

## Usage Examples

### Classify Single File
```
Input: %HOME%\Downloads\proyecto_contrato_v2.docx
Output:
  Category: Documents
  Subcategory: Legal/Contracts
  Destination: Documents/Legal/Contracts/
  Action: Move
```

### Classify Directory
```
Input: %HOME%\Downloads\
Output:
  Total: 45 files
  Classified: 42 files
  Uncategorized: 3 files

  By Category:
  - Documents: 15
  - Code: 8
  - Media: 12
  - Archives: 5
  - Installers: 2

  Recommended Actions:
  1. Move 15 documents to Documents/
  2. Move 8 code files to Projects/
  3. Review 3 uncategorized files
```

## Integration Points

- **file-monitor agent**: Uses this skill to classify detected files
- **/classify command**: Applies this taxonomy to specified directories
- **CLAUDE.md**: References this taxonomy for organization decisions
