# Analysing Socio-Economic Parameters (World Bank API)

**Project Type:** Data retrieval → cleaning/EDA → interactive visualization
**Scope:** G20 economies, **2018–2023**
**Tools:** Python (`wbdata`, `pandas`, `matplotlib/seaborn`), Power BI

<img width="576" height="325" alt="WB Dash" src="https://github.com/user-attachments/assets/dd3ae42b-48c8-458d-96e4-b8a950e1a830" />

---

## Table of Contents

* [Project Overview](#project-overview)
* [Aim and Objectives](#aim-and-objectives)
* [Data Sources](#data-sources)
* [Indicators Tracked](#indicators-tracked)
* [Countries](#countries)
* [Repository Structure](#repository-structure)
* [Environment & Setup](#environment--setup)
* [Data Retrieval Process (in detail)](#data-retrieval-process-in-detail)
* [Data Preparation & EDA](#data-preparation--eda)
* [Power BI Visualization](#power-bi-visualization)
* [Key Analytical Questions](#key-analytical-questions)
* [How to Reproduce](#how-to-reproduce)
* [Notes & Limitations](#notes--limitations)
* [License](#license)

---

## Project Overview

This project analyzes **socio-economic performance across countries** using the **World Bank’s open data API**.
I automated the data pull in Python, reshaped and cleaned the data, performed light EDA (distributions, outliers, correlations), and built an interactive **Power BI** dashboard to compare growth, inflation, poverty, and other development indicators.

---
## Aim and Objectives
**Aim**

The aim of this project is to gather, analyze and visualize key socio-economic indicators of few countries between 2018 and 2023, using World Bank open data, in order to uncover trends, highlight disparities, and provide insights into how major global economies are evolving.

**Objectives**

- Automate Data Retrieval
Collect socio-economic data directly from the World Bank API to ensure accuracy, reproducibility, and transparency.

- Clean and Transform Data
Normalize values (e.g., monetary indicators into billions), restructure datasets, and handle missing values to prepare analysis-ready data.

- Perform Exploratory Data Analysis (EDA)
Investigate distributions, detect outliers, and explore correlations between indicators such as GDP growth, inflation, and poverty.

- Develop Comparative Insights
Compare socio-economic performance across G20 countries, identifying both convergences and divergences.

- Build an Interactive Dashboard
Design a Power BI dashboard that allows stakeholders to explore economic trends, filter by country, and answer guiding analytical questions.

- Support Policy and Research Discussions
Provide a data-driven foundation for understanding how global economic giants are balancing growth with social development.

---

## Data Sources

* **World Bank Open Data API** via the Python package **`wbdata`**.
* Pulls are performed programmatically for a fixed date window (**2018–2023**).

---

## Indicators Tracked

World Bank series IDs used in the notebook:

* `NY.GDP.MKTP.CD` — **GDP (current US\$)**
* `NY.GDP.MKTP.KD.ZG` — **GDP growth (annual %)**
* `NY.GNP.MKTP.CD` — **GNI (current US\$)**
* `NY.GNP.MKTP.KD.ZG` — **GNI growth (annual %)**
* `FP.CPI.TOTL.ZG` — **Inflation, consumer prices (annual %)**
* `BX.KLT.DINV.CD.WD` — **FDI, net inflows (BoP, current US\$)**
* `NE.GDI.TOTL.CD` — **Gross capital formation (current US\$)**
* `SL.UEM.TOTL.ZS` — **Unemployment rate (overall unemployment)**
* `SI.POV.NAHC` — **Poverty headcount ratio at national poverty lines (% of population)**
* `SE.ADT.LITR.ZS` — **Adult literacy rate (% ages 15+)**
* `SP.DYN.LE00.IN` — **Life expectancy at birth (years)**
* `SP.POP.GROW` — **Population growth (annual %)**
* `SE.XPD.TOTL.GB.ZS` — **Government expenditure on education (% of GDP)**

---

## Countries

Filtered to **21 country** members from the World Bank:
Argentina, Australia, Brazil, Canada, China, **European Union**, France, Germany, India, Indonesia, Italy, Japan, Mexico, Russia, Saudi Arabia, South Africa, South Korea, Turkey, United Kingdom, Nigeria, United States.


---

## Repository Structure

```
.
├── assets/
│   └── dashboard.png            # dashboard screenshot
├── data/
│   └── G20.csv, World.csv, NG.csv                  # cleaned, analysis-ready export for BI
├── notebooks/
│   └── Socio-Economic Parameter Data Gathering & EDA.ipynb
├── README.md
└── license
```

---

## Environment & Setup

```bash
# Create & activate environment (example with pip)
python -m venv .venv
source .venv/bin/activate    # on Windows: .venv\Scripts\activate

pip install pandas numpy matplotlib seaborn wbdata
```

---

## Data Retrieval Process (in detail)

The **notebook** automates the end-to-end pull with `wbdata`:

1. **Define date range**

   ```python
   from datetime import datetime
   data_date = (datetime(2018, 1, 1), datetime(2023, 12, 31))
   ```

2. **Declare indicators** (friendly names ↔︎ World Bank codes)
   The notebook builds a mapping (see *Indicators Tracked*) into a small DataFrame (`df_indicators2`) to keep names consistent downstream.

3. **Bulk download for all economies**

   ```python
   import wbdata, pandas as pd

   df_list = []
   for code in df_indicators2['Indicator ID']:
       raw = wbdata.get_data(indicator=code, country="all", date=data_date)
       tmp = pd.DataFrame(raw).reset_index()[['country','countryiso3code','date','value']]
       # normalize "country" from dict to string
       tmp['country'] = tmp['country'].apply(lambda x: x['value'] if isinstance(x, dict) else x)
       tmp.columns = ['country','countryiso3code','year','value']
       tmp['Indicator Name'] = df_indicators2.loc[
           df_indicators2['Indicator ID'] == code, 'Indicator Name'
       ].values[0]
       df_list.append(tmp)

   df_long = pd.concat(df_list, ignore_index=True)
   ```

4. **Pivot to a wide, analysis-friendly shape**

   ```python
   df_wide = df_long.pivot_table(
       index=['country','countryiso3code','year'],
       columns='Indicator Name',
       values='value'
   ).reset_index()
   ```

5. **Filter to G20 + Nigeria economies**

   ```python
   g20 = ["Argentina","Australia","Brazil","Canada","China","France","Germany",
          "India","Indonesia","Italy","Japan","Mexico","Russia","Saudi Arabia",
          "South Africa","South Korea","Turkey","United Kingdom","United States", "Nigeria",
          "European Union"]
   df_g20 = df_wide[df_wide['country'].isin(g20)].copy()

   ```

6. **Unit normalization for monetary series**
   Convert large monetary values to **billions of US\$** for readability:

   ```python
   df_g20['Foreign Domestic Investment(FDI)'] = df_g20['FDI'] / 1**9
   df_g20['Gross Capital Formation(GCF)']    = df_g20['GCF'] / 1**9
   df_g20['Gross Domestic Product(GDP)']     = df_g20['GDP'] / 1**9
   df_g20['Gross National Income(GNI)']      = df_g20['GNI'] / 1**9
   ```

7. **Export to CSV for BI tools**

   ```python
   df_export = df_g20.loc[:, ~df_g20.columns.isin(["FDI","GCF","GDP","GNI"])]
   df_export.to_csv("data/G20.csv", index=False)
   ```

---

## Data Preparation & EDA

* **Types & Dates**
  Cast `year` to datetime for reliable time operations.

* **Descriptive statistics**
  Quick `describe()` across indicators to spot data scales and missingness.

* **Distributions & Outliers**

  * Histograms and boxplots for each indicator.
  * Outlier scoring using the **IQR rule**:

    ```python
    def is_outlier(x):
        x = pd.to_numeric(x, errors='coerce').dropna()
        q1, q3 = x.quantile(0.25), x.quantile(0.75)
        iqr = q3 - q1
        return (x < q1 - 1.5*iqr) | (x > q3 + 1.5*iqr)
    ```

  > Percentage of outliers per column helps decide where winsorization or further cleaning may be needed.

* **Correlation analysis**
  A correlation matrix/heatmap across numeric indicators to surface patterns (e.g., the relationship between growth, inflation, and poverty).

---

## Power BI Visualization

**Data:** `data/G20.csv` (one row per `country × year` with indicator columns)

**Main visuals (as in the screenshot):**

* **Cards/Donuts:** *GDP growth %* and *GNI growth %* with **% change** from 2018 to 2023.
* **Bar chart (time series):** *Population growth*, *Inflation rate*, *Poverty rate* by year.
* **Filled map:** *Global country overview* (21 highlighted).
* **Bar chart (macro):** *FDI*, *GCF*, *GDP*, *GNI* (in **billions**), compared by year.
* **Slicer:** Country (left panel).

**Example DAX (percent change, 2018→2023):**

```DAX
%Change GDP = 
VAR _2018 = 
    CALCULATE(
        SUM('G20 Countries'[GDP growth %]),
        VALUE('G20 Countries'[Year])= 2018
    )
VAR _2023 = 
    CALCULATE(
        SUM('G20 Countries'[GDP growth %]),
        VALUE('G20 Countries'[Year])= 2023
    )

VAR _Change = DIVIDE(_2023 - _2018, _2018, 0)

VAR _direction = 
    SWITCH(
        TRUE(),
        _Change > 0, UNICHAR(11165), 
         _Change < 0, UNICHAR(11167),  
         _Change = 0, "●"             
    )
RETURN
    ":" & " " & _direction & " " & FORMAT(_Change, "0.0%")

```

> Adjust column/table names if yours differ.

**Design cues:**

* Keep monetary axes in **billions**.
* Use **consistent year filters** (2018–2023).
* Place the **country slicer** prominently (e.g., “Which country’s economy do you want to explore?”).

---

## Key Analytical Questions

* **Which country's economy experienced the strongest rebound in growth post-2020?**
* **How do inflation spikes align with dips in growth or rises in poverty?**
* **Which countries sustained high capital formation alongside strong GDP/GNI growth?**
* **Where is FDI concentrated and how has it shifted since 2018?**
* **What socio-economic factors (inflation, literacy, education spend) track with poverty outcomes?**

---

## How to Reproduce

1. **Clone the repo & set up the environment** (see [Environment & Setup](#environment--setup)).
2. **Run the notebook**
   `notebooks/Socio-Economic Parameter Data Gathering & EDA.ipynb`
   This will pull data (2018–2023), filter to 21 countries, normalize values, and export `data/G20.csv`.
3. **Open Power BI** and load `data/G20.csv`.
   Recreate visuals (or import the provided `.pbix` if included) and add DAX measures for % change.
4. **Export a screenshot** and place it at `assets/dashboard.png` for the README.

---

## Notes & Limitations

* **API availability:** The World Bank API occasionally returns missing values for certain series/years. Visuals handle gaps, but analyses should note coverage differences.
* **EU aggregate:** Including the **EU** alongside individual countries is standard for G20 but can bias group summaries if not segmented.

---

## License

MIT (or your preferred license)

---

### Acknowledgements

* World Bank Open Data team and maintainers of **`wbdata`**.
