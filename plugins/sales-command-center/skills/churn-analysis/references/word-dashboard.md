# Word Document AM Account Health Dashboard — Template

## When to use this template

Use this when the user requests a Word document (.docx) report. This script generates the complete Word document: the styled AM account-health dashboard (matching the HTML artifact visual design) followed by all 9 report sections.

**Do not invoke `anthropic-skills:docx` when using this template** — this script replaces that companion skill for churn reports.

---

## How to produce the Word report

### Step A — Fill in all data values

Copy the Python script below. Replace every `[PLACEHOLDER]` with real computed values from the analysis (Steps 1–6). Use numeric types (not strings) for chart data arrays.

### Step B — Install dependencies if needed

```bash
pip install python-docx matplotlib --break-system-packages
```

### Step C — Run the script

Execute the filled-in script in the bash environment. It writes the .docx file to the outputs folder and prints the file path on success.

---

## Python Script Template

```python
# ═══════════════════════════════════════════════════════════════
#  Churn Analysis Report — Word Account Health Dashboard Generator
#  Fill every [PLACEHOLDER] before running.
# ═══════════════════════════════════════════════════════════════

import sys, io, os
import matplotlib; matplotlib.use('Agg')
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker

try:
    from docx import Document
    from docx.shared import Inches, Pt, RGBColor, Cm
    from docx.enum.text import WD_ALIGN_PARAGRAPH
    from docx.oxml.ns import qn
    from docx.oxml import OxmlElement
except ImportError:
    os.system("pip install python-docx --break-system-packages -q")
    from docx import Document
    from docx.shared import Inches, Pt, RGBColor, Cm
    from docx.enum.text import WD_ALIGN_PARAGRAPH
    from docx.oxml.ns import qn
    from docx.oxml import OxmlElement

OUTPUT_PATH = "/sessions/ecstatic-funny-faraday/mnt/outputs/churn-am-report.docx"

# ─── DATA — fill in from analysis ─────────────────────────────
PERIOD_LABEL      = "[PERIOD_LABEL]"     # e.g. "Jan 2026 – Jun 2026"
ISSUE_DATE        = "[ISSUE_DATE]"       # e.g. "May 25, 2026"
HEALTH_LABEL      = "[HEALTH_LABEL]"     # "High Priority" / "Medium Priority" / "Monitor"
# Priority hex color — no '#': "C0392B" (red) / "E67E22" (orange) / "27AE60" (green)
HEALTH_COLOR_HEX  = "[HEALTH_COLOR_HEX]"

# KPI values
GRR_VALUE       = "[GRR_VALUE]%"   # e.g. "81.4%"
GRR_DELTA_TEXT  = "[GRR_DELTA_TEXT]"  # optional; leave blank if no reliable prior period exists
# GRR color tuple: (192,57,43)=red / (230,126,34)=orange / (39,174,96)=green
GRR_COLOR       = ([GRR_R], [GRR_G], [GRR_B])

CHURNED_ARR     = "[CHURNED_ARR]"    # e.g. "$1.2M"
CHURNED_DELTA   = "[CHURNED_ARR_DELTA_TEXT]"  # optional; leave blank if no reliable prior period exists
CHURNED_WORSE   = True   # True if more ARR churned vs prior (delta color = red)

ACCOUNTS_LOST   = "[ACCOUNTS_LOST]"  # e.g. "18"
ACCOUNTS_DELTA  = "[ACCOUNTS_DELTA_TEXT]"  # optional; leave blank if no reliable prior period exists
ACCOUNTS_WORSE  = True   # True if more accounts lost vs prior

NRR_VALUE       = "[NRR_VALUE]"     # e.g. "94.2%" or "—"
NRR_DELTA_TEXT  = "[NRR_DELTA]"     # optional; leave blank if unavailable
NRR_COLOR       = ([NRR_R], [NRR_G], [NRR_B])  # same scheme as GRR_COLOR

AVG_DEAL_SIZE   = "[AVG_DEAL_SIZE]"  # e.g. "$67K"

# Top churned accounts — up to 5, highest ARR first
TOP_ACCOUNTS = [
    {"name": "[ACCOUNT_1_NAME]", "subtype": "[ACCOUNT_1_SUBTYPE]",
     "arr": "[ACCOUNT_1_ARR]", "type": "[ACCOUNT_1_CHURN_TYPE]",
     "reason": "[ACCOUNT_1_LOSS_REASON]", "product": "[ACCOUNT_1_PRODUCT_LINE]",
     "signal": "[ACCOUNT_1_SIGNAL]", "action": "[ACCOUNT_1_NEXT_ACTION]", "risk": "[ACCOUNT_1_RISK]"},
    {"name": "[ACCOUNT_2_NAME]", "subtype": "[ACCOUNT_2_SUBTYPE]",
     "arr": "[ACCOUNT_2_ARR]", "type": "[ACCOUNT_2_CHURN_TYPE]",
     "reason": "[ACCOUNT_2_LOSS_REASON]", "product": "[ACCOUNT_2_PRODUCT_LINE]",
     "signal": "[ACCOUNT_2_SIGNAL]", "action": "[ACCOUNT_2_NEXT_ACTION]", "risk": "[ACCOUNT_2_RISK]"},
    {"name": "[ACCOUNT_3_NAME]", "subtype": "[ACCOUNT_3_SUBTYPE]",
     "arr": "[ACCOUNT_3_ARR]", "type": "[ACCOUNT_3_CHURN_TYPE]",
     "reason": "[ACCOUNT_3_LOSS_REASON]", "product": "[ACCOUNT_3_PRODUCT_LINE]",
     "signal": "[ACCOUNT_3_SIGNAL]", "action": "[ACCOUNT_3_NEXT_ACTION]", "risk": "[ACCOUNT_3_RISK]"},
    {"name": "[ACCOUNT_4_NAME]", "subtype": "[ACCOUNT_4_SUBTYPE]",
     "arr": "[ACCOUNT_4_ARR]", "type": "[ACCOUNT_4_CHURN_TYPE]",
     "reason": "[ACCOUNT_4_LOSS_REASON]", "product": "[ACCOUNT_4_PRODUCT_LINE]",
     "signal": "[ACCOUNT_4_SIGNAL]", "action": "[ACCOUNT_4_NEXT_ACTION]", "risk": "[ACCOUNT_4_RISK]"},
    {"name": "[ACCOUNT_5_NAME]", "subtype": "[ACCOUNT_5_SUBTYPE]",
     "arr": "[ACCOUNT_5_ARR]", "type": "[ACCOUNT_5_CHURN_TYPE]",
     "reason": "[ACCOUNT_5_LOSS_REASON]", "product": "[ACCOUNT_5_PRODUCT_LINE]",
     "signal": "[ACCOUNT_5_SIGNAL]", "action": "[ACCOUNT_5_NEXT_ACTION]", "risk": "[ACCOUNT_5_RISK]"},
]

# Loss reasons — top 5, in order. pct = % share of all churned accounts (0–100)
LOSS_REASONS = [
    {"label": "[REASON_1_LABEL]", "pct": [REASON_1_PCT], "count": "[REASON_1_COUNT]", "arr": "[REASON_1_ARR]"},
    {"label": "[REASON_2_LABEL]", "pct": [REASON_2_PCT], "count": "[REASON_2_COUNT]", "arr": "[REASON_2_ARR]"},
    {"label": "[REASON_3_LABEL]", "pct": [REASON_3_PCT], "count": "[REASON_3_COUNT]", "arr": "[REASON_3_ARR]"},
    {"label": "[REASON_4_LABEL]", "pct": [REASON_4_PCT], "count": "[REASON_4_COUNT]", "arr": "[REASON_4_ARR]"},
    {"label": "[REASON_5_LABEL]", "pct": [REASON_5_PCT], "count": "[REASON_5_COUNT]", "arr": "[REASON_5_ARR]"},
]

# Product-specific cancellations. pct = % share of churned ARR (0-100)
PRODUCT_CANCELLATIONS = [
    {"label": "[PRODUCT_1_LABEL]", "pct": [PRODUCT_1_PCT], "count": "[PRODUCT_1_COUNT]", "arr": "[PRODUCT_1_ARR]"},
    {"label": "[PRODUCT_2_LABEL]", "pct": [PRODUCT_2_PCT], "count": "[PRODUCT_2_COUNT]", "arr": "[PRODUCT_2_ARR]"},
    {"label": "[PRODUCT_3_LABEL]", "pct": [PRODUCT_3_PCT], "count": "[PRODUCT_3_COUNT]", "arr": "[PRODUCT_3_ARR]"},
]

# Actions — 4 items from Section 9 recommendations
ACTIONS = [
    {"text": "[ACTION_1_TEXT]", "owner": "[ACTION_1_OWNER]", "due": "[ACTION_1_DUE]"},
    {"text": "[ACTION_2_TEXT]", "owner": "[ACTION_2_OWNER]", "due": "[ACTION_2_DUE]"},
    {"text": "[ACTION_3_TEXT]", "owner": "[ACTION_3_OWNER]", "due": "[ACTION_3_DUE]"},
    {"text": "[ACTION_4_TEXT]", "owner": "[ACTION_4_OWNER]", "due": "[ACTION_4_DUE]"},
]

# GRR trend — 4 quarterly values from Section 6. Numeric, e.g. 88.1 (not strings). Use None for missing.
GRR_TREND_LABELS = ["[PERIOD_Q1]", "[PERIOD_Q2]", "[PERIOD_Q3]", "[PERIOD_Q4]"]
GRR_TREND_VALUES = [[GRR_Q1], [GRR_Q2], [GRR_Q3], [GRR_Q4]]
CHART_Y_MIN = [CHART_Y_MIN]   # e.g. 70

DATA_SOURCE_SUMMARY = "[DATA_SOURCE_SUMMARY]"
CONFIDENCE_LEVEL    = "[CONFIDENCE_LEVEL]"

# ─── Full report text (9 sections) ─────────────────────────────
# Paste the complete text of each section as a string.
# Keep headings as lines starting with "##" or "###".
# Keep table rows as pipe-separated lines starting with "|".
FULL_REPORT_TEXT = """
[PASTE THE FULL 9-SECTION REPORT TEXT HERE — copy exactly from the analysis output]
"""
# ──────────────────────────────────────────────────────────────

# ═══ HELPER FUNCTIONS ═════════════════════════════════════════

def set_cell_bg(cell, hex_color):
    tc = cell._tc
    tcPr = tc.get_or_add_tcPr()
    shd = OxmlElement('w:shd')
    shd.set(qn('w:val'), 'clear')
    shd.set(qn('w:color'), 'auto')
    shd.set(qn('w:fill'), hex_color.lstrip('#'))
    tcPr.append(shd)

def remove_borders(table):
    tbl = table._tbl
    tblPr = tbl.find(qn('w:tblPr'))
    if tblPr is None:
        tblPr = OxmlElement('w:tblPr')
        tbl.insert(0, tblPr)
    tblB = OxmlElement('w:tblBorders')
    for side in ['top','left','bottom','right','insideH','insideV']:
        el = OxmlElement(f'w:{side}')
        el.set(qn('w:val'), 'none')
        tblB.append(el)
    tblPr.append(tblB)

def set_cell_bottom_border(cell, color_hex='E0E0E0'):
    tc = cell._tc
    tcPr = tc.get_or_add_tcPr()
    tblB = OxmlElement('w:tcBorders')
    b = OxmlElement('w:bottom')
    b.set(qn('w:val'), 'single')
    b.set(qn('w:sz'), '4')
    b.set(qn('w:color'), color_hex.lstrip('#'))
    tblB.append(b)
    tcPr.append(tblB)

def cell_para(cell, text, bold=False, size=10, color=None, align=None, italic=False, space_before=4, space_after=4):
    p = cell.paragraphs[0]
    p.clear()
    if align: p.alignment = align
    p.paragraph_format.space_before = Pt(space_before)
    p.paragraph_format.space_after = Pt(space_after)
    run = p.add_run(str(text))
    run.bold = bold; run.italic = italic
    run.font.size = Pt(size)
    if color: run.font.color.rgb = RGBColor(*color)
    return p

def add_cell_line(cell, text, bold=False, size=10, color=None, align=None, space_before=0, space_after=4):
    p = cell.add_paragraph()
    if align: p.alignment = align
    p.paragraph_format.space_before = Pt(space_before)
    p.paragraph_format.space_after = Pt(space_after)
    run = p.add_run(str(text))
    run.bold = bold
    run.font.size = Pt(size)
    if color: run.font.color.rgb = RGBColor(*color)
    return p

def section_label(doc, text):
    p = doc.add_paragraph()
    r = p.add_run(text)
    r.bold = True; r.font.size = Pt(8)
    r.font.color.rgb = RGBColor(150, 150, 150)
    p.paragraph_format.space_before = Pt(10)
    p.paragraph_format.space_after = Pt(3)
    return p

def embed_chart(doc, buf, width_inches=7.0):
    buf.seek(0)
    doc.add_picture(buf, width=Inches(width_inches))
    doc.paragraphs[-1].alignment = WD_ALIGN_PARAGRAPH.LEFT

# ═══ BUILD DOCUMENT ═══════════════════════════════════════════

doc = Document()
for sec in doc.sections:
    sec.top_margin    = Cm(1.5)
    sec.bottom_margin = Cm(1.5)
    sec.left_margin   = Cm(1.8)
    sec.right_margin  = Cm(1.8)
doc.styles['Normal'].font.name = 'Calibri'
doc.styles['Normal'].font.size = Pt(10)

# ─── 1. HEADER ────────────────────────────────────────────────
ht = doc.add_table(rows=1, cols=2)
remove_borders(ht)
ht.autofit = False
ht.columns[0].width = Cm(12)
ht.columns[1].width = Cm(7)
hl, hr = ht.cell(0,0), ht.cell(0,1)
set_cell_bg(hl, '111111'); set_cell_bg(hr, '111111')

hl.paragraphs[0].clear()
hl.paragraphs[0].paragraph_format.space_before = Pt(10)
r1 = hl.paragraphs[0].add_run('Churn Analysis Report')
r1.bold = True; r1.font.size = Pt(13); r1.font.color.rgb = RGBColor(255,255,255)

p2 = hl.add_paragraph()
p2.paragraph_format.space_before = Pt(2)
r2 = p2.add_run('Account Health Tracking  ·  Renewal, customer, and account-team evidence')
r2.font.size = Pt(9); r2.font.color.rgb = RGBColor(180,180,180)

p3 = hl.add_paragraph()
p3.paragraph_format.space_before = Pt(6); p3.paragraph_format.space_after = Pt(10)
badge = p3.add_run(f'  {HEALTH_LABEL}  ')
badge.bold = True; badge.font.size = Pt(9); badge.font.color.rgb = RGBColor(255,255,255)
rPr = badge._r.get_or_add_rPr()
shd = OxmlElement('w:shd')
shd.set(qn('w:val'), 'clear'); shd.set(qn('w:color'), 'auto'); shd.set(qn('w:fill'), HEALTH_COLOR_HEX)
rPr.append(shd)

hr.paragraphs[0].clear()
hr.paragraphs[0].alignment = WD_ALIGN_PARAGRAPH.RIGHT
hr.paragraphs[0].paragraph_format.space_before = Pt(10)
r3 = hr.paragraphs[0].add_run(f'Week of {ISSUE_DATE}')
r3.font.size = Pt(9); r3.font.color.rgb = RGBColor(170,170,170)
for line in [f'Prepared by Account Management', f'Period: {PERIOD_LABEL}']:
    px = hr.add_paragraph()
    px.alignment = WD_ALIGN_PARAGRAPH.RIGHT
    rx = px.add_run(line)
    rx.font.size = Pt(9); rx.font.color.rgb = RGBColor(170,170,170)
hr.paragraphs[-1].paragraph_format.space_after = Pt(10)

doc.add_paragraph().paragraph_format.space_after = Pt(2)

# ─── 2. KPI CARDS ─────────────────────────────────────────────
section_label(doc, 'KEY PERFORMANCE INDICATORS')

kpi_table = doc.add_table(rows=1, cols=5)
remove_borders(kpi_table)
kpi_table.autofit = False
card_w = Cm(3.8)
for col in kpi_table.columns:
    col.width = card_w

DELTA_RED   = (192, 57, 43)
DELTA_GREEN = (39, 174, 96)
DELTA_GREY  = (150, 150, 150)

KPI_CARDS = [
    ('GROSS RENEWAL RATE', GRR_VALUE,    GRR_DELTA_TEXT,   GRR_COLOR),
    ('CHURNED ARR',        CHURNED_ARR,  CHURNED_DELTA,    DELTA_RED if CHURNED_WORSE else DELTA_GREEN),
    ('ACCOUNTS LOST',      ACCOUNTS_LOST,ACCOUNTS_DELTA,   DELTA_RED if ACCOUNTS_WORSE else DELTA_GREEN),
    ('NET REV. RETENTION', NRR_VALUE,    NRR_DELTA_TEXT,   NRR_COLOR),
    ('AVG CHURNED DEAL',   AVG_DEAL_SIZE,f'across {ACCOUNTS_LOST} accounts', DELTA_GREY),
]

for ci, (label, value, delta, dcolor) in enumerate(KPI_CARDS):
    cell = kpi_table.cell(0, ci)
    set_cell_bg(cell, 'FFFFFF')
    set_cell_bottom_border(cell, 'DDDDDD')

    cell.paragraphs[0].clear()
    cell.paragraphs[0].paragraph_format.space_before = Pt(10)
    lr = cell.paragraphs[0].add_run(label)
    lr.bold = True; lr.font.size = Pt(8); lr.font.color.rgb = RGBColor(140, 140, 140)

    vp = cell.add_paragraph()
    vp.paragraph_format.space_before = Pt(4); vp.paragraph_format.space_after = Pt(3)
    vr = vp.add_run(value)
    vr.bold = True; vr.font.size = Pt(20)
    vr.font.color.rgb = RGBColor(*dcolor) if label in ('GROSS RENEWAL RATE', 'NET REV. RETENTION') else RGBColor(17, 17, 17)

    if str(delta).strip():
        dp = cell.add_paragraph()
        dp.paragraph_format.space_after = Pt(10)
        dr = dp.add_run(delta)
        dr.font.size = Pt(8); dr.font.color.rgb = RGBColor(*dcolor)
    else:
        cell.paragraphs[-1].paragraph_format.space_after = Pt(10)

doc.add_paragraph().paragraph_format.space_after = Pt(4)

# ─── 3. ACCOUNTS REQUIRING AM FOLLOW-UP ──────────────────────
section_label(doc, f'ACCOUNTS REQUIRING AM FOLLOW-UP  ·  {PERIOD_LABEL}')

RISK_COLORS = {'High': (192,57,43), 'Med': (230,126,34), 'Low': (39,174,96)}
ACCT_HDRS = ['ACCOUNT', 'ARR LOST', 'CHURN TYPE', 'LOSS REASON', 'PRODUCT', 'KEY SIGNAL', 'NEXT AM ACTION', 'RISK']
ACCT_WIDTHS = [Cm(2.9), Cm(1.5), Cm(2.2), Cm(2.1), Cm(1.8), Cm(2.7), Cm(3.6), Cm(1.2)]

at = doc.add_table(rows=1 + len(TOP_ACCOUNTS), cols=8)
remove_borders(at)
at.autofit = False
for ci, w in enumerate(ACCT_WIDTHS):
    for row in at.rows:
        row.cells[ci].width = w

for ci, hdr in enumerate(ACCT_HDRS):
    c = at.cell(0, ci)
    set_cell_bg(c, 'F7F7F7')
    set_cell_bottom_border(c, 'DDDDDD')
    cell_para(c, hdr, bold=True, size=8, color=(140,140,140))

for ri, acct in enumerate(TOP_ACCOUNTS):
    row_data = [acct['name'], acct['arr'], acct['type'],
                acct['reason'], acct['product'], acct['signal'], acct['action'], acct['risk']]
    for ci, (cell, val) in enumerate(zip([at.cell(ri+1, j) for j in range(8)], row_data)):
        set_cell_bottom_border(cell, 'F0F0F0')
        if ci == 0:
            cell_para(cell, val, bold=True, size=9, space_before=6)
            if acct.get('subtype'):
                add_cell_line(cell, acct['subtype'], size=8, color=(150,150,150), space_after=6)
        elif ci == 7:
            cell_para(cell, val, bold=True, size=9,
                      color=RISK_COLORS.get(val, (150,150,150)), space_before=6, space_after=6)
        else:
            cell_para(cell, val, size=9, space_before=6, space_after=6)

doc.add_paragraph().paragraph_format.space_after = Pt(6)

# ─── 4. GRR TREND CHART ───────────────────────────────────────
section_label(doc, 'GROSS RENEWAL RATE — QUARTERLY TREND')

bar_cols = ['#27ae60' if (v or 0) >= 85 else '#e67e22' if (v or 0) >= 75 else '#c0392b'
            for v in GRR_TREND_VALUES]
clean_vals = [v if v is not None else 0 for v in GRR_TREND_VALUES]

fig, ax = plt.subplots(figsize=(8.5, 2.5), facecolor='white')
bars = ax.bar(GRR_TREND_LABELS, clean_vals, color=bar_cols,
              width=0.55, edgecolor='none', zorder=3)
ax.set_ylim(CHART_Y_MIN, 100)
ax.yaxis.set_major_formatter(mticker.FuncFormatter(lambda x, _: f'{int(x)}%'))
ax.tick_params(labelsize=9, color='#cccccc')
ax.spines['top'].set_visible(False); ax.spines['right'].set_visible(False)
ax.spines['left'].set_color('#e0e0e0'); ax.spines['bottom'].set_color('#e0e0e0')
ax.yaxis.grid(True, color='#f0f0f0', zorder=0)
ax.set_axisbelow(True)
for bar, val in zip(bars, clean_vals):
    if val:
        ax.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 0.3,
                f'{val:.1f}%', ha='center', va='bottom', fontsize=9,
                color='#444444', fontweight='bold')
plt.tight_layout(pad=0.4)
buf1 = io.BytesIO()
plt.savefig(buf1, format='png', dpi=150, bbox_inches='tight', facecolor='white')
plt.close()
embed_chart(doc, buf1, width_inches=7.0)

doc.add_paragraph().paragraph_format.space_after = Pt(6)

# ─── 5. LOSS REASON + PRODUCT CANCELLATION CHARTS ────────────
section_label(doc, 'LOSS REASON DISTRIBUTION  &  PRODUCT-SPECIFIC CANCELLATION IMPACT')

fig2, (ax_l, ax_r) = plt.subplots(1, 2, figsize=(9.5, 3.0), facecolor='white')

# Left: horizontal loss reason bars
lr_labels  = [r['label'] for r in LOSS_REASONS]
lr_values  = [r['pct']   for r in LOSS_REASONS]
lr_annots  = [f"{r['count']} accts · {r['arr']}" for r in LOSS_REASONS]
lr_colors  = ['#E74C3C', '#E67E22', '#F39C12', '#3498DB', '#95A5A6']

bars_l = ax_l.barh(lr_labels[::-1], lr_values[::-1],
                    color=lr_colors[::-1], height=0.55, edgecolor='none', zorder=3)
ax_l.set_xlabel('% of churned accounts', fontsize=9, color='#666666')
ax_l.set_title('Loss Reason Distribution', fontsize=10, fontweight='bold', pad=10, color='#222')
ax_l.tick_params(labelsize=9, color='#cccccc')
ax_l.spines['top'].set_visible(False); ax_l.spines['right'].set_visible(False)
ax_l.spines['left'].set_color('#e0e0e0'); ax_l.spines['bottom'].set_color('#e0e0e0')
ax_l.xaxis.grid(True, color='#f0f0f0', zorder=0)
ax_l.set_axisbelow(True)
for bar, ann in zip(bars_l, lr_annots[::-1]):
    ax_l.text(bar.get_width() + 0.5, bar.get_y() + bar.get_height()/2,
              ann, va='center', fontsize=8, color='#666666')
ax_l.set_xlim(0, max(lr_values) * 1.5)

# Right: product cancellation bar chart
lc_labels = [l['label'] for l in PRODUCT_CANCELLATIONS]
lc_values = [l['pct']   for l in PRODUCT_CANCELLATIONS]
lc_annots = [f"{l['count']} accts\n{l['arr']}" for l in PRODUCT_CANCELLATIONS]
lc_colors = ['#C0392B', '#E67E22', '#3498DB']

bars_r = ax_r.bar(lc_labels, lc_values, color=lc_colors,
                   width=0.5, edgecolor='none', zorder=3)
ax_r.set_ylabel('% of churned ARR', fontsize=9, color='#666666')
ax_r.set_title('Product Cancellation Impact', fontsize=10, fontweight='bold', pad=10, color='#222')
ax_r.tick_params(labelsize=9, color='#cccccc')
ax_r.spines['top'].set_visible(False); ax_r.spines['right'].set_visible(False)
ax_r.spines['left'].set_color('#e0e0e0'); ax_r.spines['bottom'].set_color('#e0e0e0')
ax_r.yaxis.grid(True, color='#f0f0f0', zorder=0)
ax_r.set_axisbelow(True)
for bar, ann in zip(bars_r, lc_annots):
    ax_r.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 0.5,
              ann, ha='center', va='bottom', fontsize=8, color='#555555')

plt.tight_layout(pad=0.6)
buf2 = io.BytesIO()
plt.savefig(buf2, format='png', dpi=150, bbox_inches='tight', facecolor='white')
plt.close()
embed_chart(doc, buf2, width_inches=7.0)

doc.add_paragraph().paragraph_format.space_after = Pt(6)

# ─── 6. AM ACTIONS REQUIRED ──────────────────────────────────
section_label(doc, 'AM ACTIONS REQUIRED')

act_tbl = doc.add_table(rows=len(ACTIONS), cols=2)
remove_borders(act_tbl)
act_tbl.autofit = False
act_tbl.columns[0].width = Cm(0.8)
act_tbl.columns[1].width = Cm(18.4)

for ri, action in enumerate(ACTIONS):
    num_cell = act_tbl.cell(ri, 0)
    txt_cell = act_tbl.cell(ri, 1)
    set_cell_bottom_border(num_cell, 'F0F0F0')
    set_cell_bottom_border(txt_cell, 'F0F0F0')

    cell_para(num_cell, str(ri+1), bold=True, size=11,
              color=(200,200,200), space_before=6, space_after=2)
    cell_para(txt_cell, action['text'], size=10, space_before=6, space_after=2)
    add_cell_line(txt_cell,
                  f"Owner: {action['owner']}  ·  Due: {action['due']}",
                  size=8, color=(150,150,150), space_after=6)

doc.add_paragraph().paragraph_format.space_after = Pt(4)

# Footer
fp = doc.add_paragraph()
fp.alignment = WD_ALIGN_PARAGRAPH.RIGHT
fr = fp.add_run(f'AI-synthesized from {DATA_SOURCE_SUMMARY}  ·  Confidence: {CONFIDENCE_LEVEL}  ·  Sources verified')
fr.font.size = Pt(8); fr.font.color.rgb = RGBColor(180,180,180)

# ─── 7. FULL REPORT (9 sections) ──────────────────────────────
# Continue naturally after the dashboard to avoid blank pages or unnecessary discontinuity.
# Parse the FULL_REPORT_TEXT and render it as formatted Word content.
# Headings, tables, and body text are detected by line prefix.

def is_table_row(line):
    return line.strip().startswith('|') and line.strip().endswith('|')

def is_separator_row(line):
    return is_table_row(line) and all(c in '|- :' for c in line.strip())

def add_report_section(doc, text):
    lines = text.strip().split('\n')
    i = 0
    current_table_rows = []

    def flush_table():
        if not current_table_rows:
            return
        header = current_table_rows[0]
        data_rows = [r for r in current_table_rows[1:] if not is_separator_row(r)]
        cols = [c.strip() for c in header.strip('|').split('|')]
        n_cols = len(cols)
        t = doc.add_table(rows=1 + len(data_rows), cols=n_cols)
        remove_borders(t)
        t.autofit = True
        for ci, col in enumerate(cols):
            c = t.cell(0, ci)
            set_cell_bg(c, 'F7F7F7')
            set_cell_bottom_border(c, 'CCCCCC')
            cell_para(c, col, bold=True, size=9, color=(100,100,100))
        for ri, row in enumerate(data_rows):
            cells = [c.strip() for c in row.strip('|').split('|')]
            for ci in range(n_cols):
                val = cells[ci] if ci < len(cells) else ''
                c = t.cell(ri+1, ci)
                set_cell_bottom_border(c, 'F0F0F0')
                cell_para(c, val, size=9, space_before=5, space_after=5)
        doc.add_paragraph().paragraph_format.space_after = Pt(4)
        current_table_rows.clear()

    while i < len(lines):
        line = lines[i]
        stripped = line.strip()

        if is_table_row(stripped):
            current_table_rows.append(stripped)
            i += 1
            continue
        else:
            flush_table()

        if stripped.startswith('## '):
            p = doc.add_paragraph()
            p.paragraph_format.space_before = Pt(16)
            p.paragraph_format.space_after = Pt(6)
            r = p.add_run(stripped[3:])
            r.bold = True; r.font.size = Pt(13); r.font.color.rgb = RGBColor(17,17,17)
            # Underline via border
        elif stripped.startswith('### '):
            p = doc.add_paragraph()
            p.paragraph_format.space_before = Pt(10)
            r = p.add_run(stripped[4:])
            r.bold = True; r.font.size = Pt(11); r.font.color.rgb = RGBColor(50,50,50)
        elif stripped.startswith('**') and stripped.endswith('**') and len(stripped) > 4:
            p = doc.add_paragraph()
            r = p.add_run(stripped.strip('*'))
            r.bold = True; r.font.size = Pt(10)
        elif stripped.startswith('> '):
            p = doc.add_paragraph()
            p.paragraph_format.left_indent = Cm(1)
            r = p.add_run(stripped[2:])
            r.italic = True; r.font.size = Pt(10); r.font.color.rgb = RGBColor(80,80,80)
        elif stripped.startswith('*') and stripped.endswith('*') and stripped.count('*') == 2:
            p = doc.add_paragraph()
            r = p.add_run(stripped.strip('*'))
            r.italic = True; r.font.size = Pt(9); r.font.color.rgb = RGBColor(120,120,120)
        elif stripped == '---':
            p = doc.add_paragraph()
            p.paragraph_format.space_before = Pt(4)
            p.paragraph_format.space_after = Pt(4)
        elif stripped == '':
            doc.add_paragraph().paragraph_format.space_after = Pt(2)
        else:
            p = doc.add_paragraph()
            r = p.add_run(stripped)
            r.font.size = Pt(10)
        i += 1

    flush_table()

add_report_section(doc, FULL_REPORT_TEXT)

# Footer on last page
doc.add_paragraph()
ep = doc.add_paragraph()
ep.alignment = WD_ALIGN_PARAGRAPH.CENTER
er = ep.add_run(f'Churn Analysis Report  ·  {PERIOD_LABEL}  ·  Account Management  ·  Internal — Confidential')
er.font.size = Pt(8); er.font.color.rgb = RGBColor(180,180,180)

# ─── SAVE ──────────────────────────────────────────────────────
doc.save(OUTPUT_PATH)
print(f"SUCCESS: Report saved to {OUTPUT_PATH}")
```
