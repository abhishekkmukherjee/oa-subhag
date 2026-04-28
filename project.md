# OpenOA AI Dashboard — Claude Code Context

## 🎯 Project Goal
Build a **Streamlit web dashboard** on top of the OpenOA wind plant analysis library, with AI-powered insights via Google Gemini API. Deploy it live on Render.com as a technical assignment for Subhag HealthTech.

This demonstrates: full-stack deployment skills, AI integration, data visualization, and ability to extend an existing open-source Python codebase.

---

## 📁 Base Repo
- **Repo**: https://github.com/NatLabRockies/OpenOA (already cloned)
- **What it is**: Python library for wind plant operational analysis (AEP estimation, wake losses, electrical losses, turbine performance, yaw misalignment)
- **Sample data**: `examples/la_haute_borne.zip` — unzip to `examples/data/la_haute_borne/` — this is a real French wind farm dataset (La Haute Borne)
- **Key Python modules**:
  - `openoa.plant` — PlantData class (loads SCADA, met, reanalysis data)
  - `openoa.analysis.aep` — MonteCarloAEP analysis
  - `openoa.analysis.electrical_losses` — ElectricalLosses
  - `openoa.analysis.wake_losses` — WakeLosses
  - `openoa.analysis.turbine_long_term_gross_energy` — TurbineLongTermGrossEnergy
  - `openoa.utils.plot` — Plotting utilities
  - `examples/project_ENGIE.py` — Helper to load the La Haute Borne dataset

---

## 🏗️ What We Are Building

### File Structure
```
OpenOA/
├── dashboard.py              ← MAIN FILE (Streamlit app)
├── ai_insights.py            ← Gemini AI insights module
├── requirements_dashboard.txt ← Extra deps for dashboard
├── render.yaml               ← Render.com deployment config
├── .streamlit/
│   └── config.toml           ← Streamlit theme config
├── examples/
│   ├── project_ENGIE.py      ← existing data loader
│   └── data/la_haute_borne/  ← unzipped wind farm data
└── openoa/                   ← existing library (do not modify core)
```

---

## 🖥️ Dashboard Features to Build

### 1. Sidebar — Plant Overview
- Wind farm name, location, number of turbines
- Date range selector for analysis period
- "Run Analysis" button

### 2. Tab 1 — 📊 Plant Performance
- **Power Curve chart**: scatter plot of wind speed vs power output per turbine (plotly)
- **Availability heatmap**: turbine × month grid showing uptime %
- **Energy production bar chart**: monthly gross vs net energy (MWh)
- **Key KPI cards**: Total AEP (GWh), Capacity Factor (%), Availability (%), Wake Loss (%)

### 3. Tab 2 — 🔍 Loss Analysis
- **Waterfall/breakdown chart**: Gross Energy → Wake Losses → Electrical Losses → Availability Losses → Net AEP
- **Wake loss map**: spatial layout of turbines colored by wake loss %
- **Electrical losses trend**: monthly line chart

### 4. Tab 3 — 🤖 AI Insights (KEY FEATURE)
- Text area showing auto-generated analysis summary
- Gemini API call with plant metrics as context
- Insights like:
  - "Turbine T4 is underperforming by 8% — likely blade degradation"
  - "Wake losses peak in Q1 due to prevailing westerly winds"
  - "Recommend curtailment strategy for T1-T3 cluster"
- "Regenerate Insights" button
- Export insights as PDF button

### 5. Tab 4 — 📈 Turbine Deep Dive
- Turbine selector dropdown
- Individual turbine power curve
- Anomaly detection chart (flag outliers in SCADA data)
- Performance score vs fleet average

### 6. Tab 5 — 🌤️ Reanalysis & Wind Resource
- ERA5/MERRA2 wind speed trend (monthly)
- Wind rose chart (direction frequency)
- Long-term vs short-term wind resource comparison

---

## 🤖 AI Insights Module (`ai_insights.py`)

```python
import google.generativeai as genai

def generate_plant_insights(metrics: dict, api_key: str) -> str:
    """
    metrics = {
        "plant_name": "La Haute Borne",
        "num_turbines": 4,
        "aep_gwh": 21.3,
        "capacity_factor_pct": 24.1,
        "availability_pct": 97.2,
        "wake_loss_pct": 3.8,
        "electrical_loss_pct": 1.2,
        "worst_turbine": "T4",
        "worst_turbine_underperformance_pct": 8.1,
        "analysis_period": "2014-2016"
    }
    """
    genai.configure(api_key=AIzaSyCkDsRT7pZLpOyjmuDTZNhoAjA5iQmq1kA)
    curl "https://generativelanguage.googleapis.com/v1beta/models/gemini-flash-latest:generateContent" \
  -H 'Content-Type: application/json' \
  -H 'X-goog-api-key: AIzaSyCkDsRT7pZLpOyjmuDTZNhoAjA5iQmq1kA' \
  -X POST \
  -d '{
    "contents": [
      {
        "parts": [
          {
            "text": "Explain how AI works in a few words"
          }
        ]
      }
    ]
  }'
    
    prompt = f"""
    You are an expert wind energy analyst. Analyze the following wind plant performance data and provide:
    1. Executive summary (2-3 sentences)
    2. Top 3 performance issues with probable causes
    3. Specific actionable recommendations
    4. Risk flags if any metrics are concerning
    
    Plant Data:
    {metrics}
    
    Be specific, technical, and concise. Use wind energy industry terminology.
    """
    response = model.generate_content(prompt)
    return response.text
```

---

## ⚙️ Streamlit Config (`.streamlit/config.toml`)

```toml
[theme]
primaryColor = "#00C49A"
backgroundColor = "#0F1117"
secondaryBackgroundColor = "#1A1D2E"
textColor = "#FAFAFA"
font = "monospace"

[server]
headless = true
enableCORS = false
```

---

## 📦 Requirements (`requirements_dashboard.txt`)

```
streamlit>=1.32.0
plotly>=5.18.0
google-generativeai>=0.5.0
pandas>=2.2.0
numpy>=1.24.0
scipy>=1.7.0
matplotlib>=3.6.0
reportlab>=4.0.0
windrose>=1.9.0
```

---

## 🚀 Render.com Deployment (`render.yaml`)

```yaml
services:
  - type: web
    name: openoa-ai-dashboard
    env: python
    buildCommand: |
      pip install -e .
      pip install -r requirements_dashboard.txt
      cd examples && unzip -o la_haute_borne.zip -d data/la_haute_borne/
    startCommand: streamlit run dashboard.py --server.port $PORT --server.address 0.0.0.0
    envVars:
      - key: GEMINI_API_KEY
        sync: false
```

---

## 🔑 Environment Variables Needed
- `GEMINI_API_KEY` — Google Gemini API key (set in Render dashboard, NOT in code)

---

## 📝 Extra Features Added Beyond Base Repo (Impress Factor)

1. **AI narrative insights** — Gemini generates human-readable analysis (not in original OpenOA)
2. **Anomaly detection** — Flag turbines with statistical outliers in power curve using IQR/Z-score
3. **PDF export** — Download AI insights + charts as a report (using reportlab)
4. **Interactive turbine comparison** — Multi-select turbines and overlay their power curves
5. **Performance scoring** — Each turbine gets a 0-100 score vs fleet baseline
6. **Dark theme dashboard** — Professional dark UI unlike OpenOA's default matplotlib plots
7. **Responsive KPI cards** — Live metric cards at top (AEP, CF, Availability, Wake Loss)

---

## ⚡ Loading Strategy
OpenOA analysis (especially MonteCarloAEP) is slow. Use:
```python
@st.cache_data
def run_aep_analysis():
    # ... heavy computation here
    return results
```
Show `st.spinner("Running Monte Carlo AEP analysis...")` while computing.

---

## 🎨 UI Style Guide
- Dark background: `#0F1117`
- Accent green: `#00C49A`  
- Card background: `#1A1D2E`
- All charts: Plotly (not matplotlib) for interactivity
- Font: monospace for data, clean sans for labels
- KPI cards: metric with delta vs previous period

---

## 🧪 Local Dev Commands
```bash
# Setup
git clone https://github.com/NatLabRockies/OpenOA.git
cd OpenOA
python -m venv venv
source venv/bin/activate        # Mac/Linux
venv\Scripts\activate           # Windows

pip install -e .
pip install -r requirements_dashboard.txt

# Unzip data
cd examples
unzip la_haute_borne.zip -d data/la_haute_borne/
cd ..

# Run
streamlit run dashboard.py

# Set Gemini key locally
export GEMINI_API_KEY="your_key_here"   # Mac/Linux
set GEMINI_API_KEY=your_key_here        # Windows
```

---

## 📤 Deployment Steps (After Building)
1. Push everything to your GitHub fork of OpenOA
2. Go to render.com → New Web Service
3. Connect GitHub repo
4. Set `GEMINI_API_KEY` in Environment Variables
5. Deploy — takes ~5 mins
6. Share the live URL with Subhag HR

---

## ✅ Definition of Done
- [ ] `streamlit run dashboard.py` works locally with real data
- [ ] All 5 tabs render without errors
- [ ] AI Insights tab generates real Gemini response
- [ ] Deployed on Render with public URL
- [ ] URL shared with hr@subhag.in

---

## 🗒️ Notes for Claude Code
- Do NOT modify anything inside `openoa/` core library
- Use `examples/project_ENGIE.py` to understand how to load PlantData
- The La Haute Borne dataset has 4 turbines, 2014-2016 data
- If MonteCarloAEP is too slow, use pre-computed dummy metrics for demo
- Prioritize working > perfect — EOD deadline
- All Plotly charts should use `template="plotly_dark"`