---
name: visual-reporting
version: 1.0.0
description: >
  Standard for generating premium executive dashboards and reports using
  HTML, CSS, and Mermaid.js diagrams. Enforces the 'March 2026' visual brand.
---

# visual-reporting

Use this skill when generating system dashboards, project status reports, or cost-benefit analyses that require executive-level visual fidelity.

## 1. Visual Specification (CSS Blueprint)

All reports MUST embed the following CSS for consistency:

```css
@import url('https://fonts.googleapis.com/css2?family=Source+Serif+4:ital,wght@0,300;0,400;0,600;0,700;1,400&family=Source+Sans+3:wght@400;600;700&family=Source+Code+Pro:wght@400;500&display=swap');

body {
  font-family: 'Source Serif 4', Georgia, serif;
  font-size: 10.5pt;
  line-height: 1.65;
  color: #1a1a1a;
  max-width: 190mm;
  margin: 0 auto;
  padding: 25mm 22mm;
}

h2 {
  font-size: 16pt;
  border-bottom: 1.5px solid #1a1a1a;
  margin: 36px 0 18px 0;
}

table {
  width: 100%;
  border-collapse: collapse;
  font-size: 9.5pt;
}

thead th {
  font-family: 'Source Sans 3', sans-serif;
  text-transform: uppercase;
  border-top: 2px solid #1a1a1a;
  border-bottom: 1px solid #1a1a1a;
}

.mermaid {
  background: #fcfcfa;
  border: 1px solid #e0e0e0;
  padding: 16px 8px;
}

.key-figures {
  display: flex;
  gap: 0;
  border: 1px solid #999;
}

.key-fig {
  flex: 1;
  text-align: center;
  padding: 14px 10px;
}
```

## 2. Diagramming Spec (Mermaid.js)

Always initialize Mermaid.js with these system-wide theme variables:

```javascript
mermaid.initialize({
  startOnLoad: true,
  theme: 'base',
  themeVariables: {
    primaryColor: '#e8e8e8',
    primaryTextColor: '#1a1a1a',
    primaryBorderColor: '#666',
    lineColor: '#555',
    fontFamily: '"Source Sans 3", sans-serif',
    fontSize: '13px'
  },
  gantt: {
    fontSize: 11,
    barHeight: 24,
    useWidth: 800
  }
});
```

## 3. Reporting Protocols

### 3.1 Cronogramas (Gantt)
- Always include a Gantt chart for projects with multiple phases.
- Use `crit` for overdue or high-priority tasks.
- Format dates as `YYYY-MM-DD`.

### 3.2 Key Figures
- Use the `.key-figures` container to highlight critical metrics:
  - **ROI**: Savings vs. Investment.
  - **Status**: Progress percentage.
  - **Criticality**: Risk level.

### 3.3 Financial Logic
- Reports MUST calculate the **Multiplicador de Eficiencia** (Manual Hours / Automated Hours).
- Use tables to show the comparison between the 2025 (Manual) vs 2026 (Automated) reporting cycles.

---
*Reference: C:\Users\msi\central_command\Projects\data\PLAN_MARZO_2026_v2.html*
