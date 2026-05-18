## ▶️ How to run

### Step 1 — Install Python
Make sure Python 3.10 or higher is installed:
https://www.python.org/downloads/

### Step 2 — Install Streamlit
Open your terminal (or Command Prompt on Windows) and run:

```
pip install streamlit
```

### Step 3 — Run the app
Navigate to the folder where you saved `dashboard.py` and run:

```
py -m streamlit run dashboard.py 
```

Your browser will automatically open at: http://localhost:8501

---

## 📁 Required Files
- dashboard.py → the main Streamlit dashboard
- requirements.txt → the required Python packages
- README.md → explanation of the project and how to run it
- sales.csv → the dataset used by the dashboard

## Important note about sales.csv

The dashboard uses sales.csv as the default dataset.
That means:

- if sales.csv is in the same folder as dashboard.py, the dashboard can start immediately
- if sales.csv is missing, the user must upload another CSV file manually inside the dashboard
