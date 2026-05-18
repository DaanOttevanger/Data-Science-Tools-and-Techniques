# ▶️ How to run

## Step 1 — Install Python
Make sure Python 3.10 or higher is installed.  

## Step 2 — Make sure the project files are in one folder
Place these files together in the same project folder:

- `dashboard.py`
- `requirements.txt`
- `README.md`
- `sales.csv`

**Important:** the project needs the file `sales.csv` to run if you do not upload another dataset manually in the dashboard.  
So make sure `sales.csv` is present in the same folder as `dashboard.py`.

## Step 3 — Install the required packages
Open your terminal (or Command Prompt / PowerShell on Windows) in the project folder and run:

```bash
pip install -r requirements.txt

##If pip does not work, try:

py -m pip install -r requirements.txt
