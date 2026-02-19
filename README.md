# ai-auto-rename
this app auto rename new files using ai


✅ Runs on Ubuntu / Windows / macOS
✅ Uses Ollama Llama3.1
✅ Uses OCR (Tesseract) for images/screenshots
✅ Auto-detects new files in Downloads/Desktop/Documents
✅ Auto-classifies file type using AI
✅ Auto-renames intelligently
✅ Auto-moves into organized folders
✅ Shows a live GUI log (like a real app)
✅ Takes NO user input (fully autonomous)


sudo apt update
sudo apt install tesseract-ocr python3-tk python3-venv -y


python3 -m venv fileguru_env
source fileguru_env/bin/activate


pip install --upgrade pip
pip install psutil pillow pytesseract requests watchdog PyPDF2


sudo apt install gnome-screenshot -y


ollama serve

# FileGuru AI PRO – README (Full Setup Guide)

This project is a **Smart AI File Organizer** that automatically watches your folders like:

- Downloads
- Desktop
- Documents

Then it:
- Detects new files
- Uses OCR + PDF text extraction
- Sends file info to **Ollama (Llama 3.1)**
- Predicts category + renames file
- Moves file into organized folders automatically
- Shows **Daily AI Report**
- Shows **AI Next Action Prediction**
- Shows **AI Productivity Insights**
- Detects duplicate files using SHA256 hash

---

# 1) System Requirements

## Supported OS
✅ Ubuntu / Linux (Recommended)  
✅ Windows 11 (Works)  
✅ macOS (Works)

## Required Hardware
- Minimum: 8 GB RAM
- Recommended: 16 GB RAM
- Recommended CPU: i5 / Ryzen 5 or higher
- GPU not required, but improves Ollama performance

---

# 2) Required Software

You need:

- Python 3.9+
- pip
- Tkinter GUI support
- Tesseract OCR
- Ollama AI server
- Llama 3.1 model

---

# 3) Ubuntu / Linux Installation (Recommended)

## Step 1: Update system
Open Terminal and run:

```bash
sudo apt update
```

---

## Step 2: Install required APT packages

```bash
sudo apt install -y python3 python3-pip python3-tk tesseract-ocr
```

### What these packages do
- `python3` → runs the program
- `python3-pip` → installs python libraries
- `python3-tk` → required for GUI (Tkinter)
- `tesseract-ocr` → OCR for image text extraction

---

## Step 3: Install required Python libraries

```bash
pip3 install watchdog pillow pytesseract PyPDF2 requests
```

### What these libraries do
- `watchdog` → watches folders for new files
- `pillow` → image processing
- `pytesseract` → OCR integration
- `PyPDF2` → PDF text extraction
- `requests` → connects to Ollama API

---

# 4) Install Ollama (Linux)

## Step 1: Install Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

---

## Step 2: Start Ollama server

```bash
ollama serve
```

⚠️ Keep this terminal running.

---

## Step 3: Download the model used in the code

Open a NEW terminal and run:

```bash
ollama pull llama3.1
```

---

# 5) Running the Program

Go to your file location:

```bash
cd /path/to/your/python/file
```

Then run:

```bash
python3 fileguru_pro_ai.py
```

---

# 6) Important Notes

## Organized folder location
Your files will be moved to:

```
~/FileGuru_Organized/
```

Inside it, folders will be created like:

- Bank
- Education
- Work
- Receipts
- Random
- Duplicates
- Photos
- etc...

---

## Files watched automatically
The app watches these folders:

- Downloads
- Desktop
- Documents

---

# 7) Verify Everything Works (Quick Test)

Run this command:

```bash
python3 -c "import watchdog, PIL, pytesseract, requests; from PyPDF2 import PdfReader; print('All required packages installed ✅')"
```

If you see:

```
All required packages installed ✅
```

Then everything is perfect.

---

# 8) If You Use Conda Environment (Optional)

If you use Conda:

## Step 1: Create environment

```bash
conda create -n fileguru python=3.11 -y
conda activate fileguru
```

## Step 2: Install python packages

```bash
pip install watchdog pillow pytesseract PyPDF2 requests
```

⚠️ Still install system dependencies with APT:

```bash
sudo apt install -y python3-tk tesseract-ocr
```

---

# 9) Common Errors and Fixes

## Error: ModuleNotFoundError: watchdog
Fix:

```bash
pip3 install watchdog
```

---

## Error: tkinter not found
Fix:

```bash
sudo apt install python3-tk
```

---

## Error: pytesseract not working / tesseract not found
Fix:

```bash
sudo apt install tesseract-ocr
```

---

## Error: Ollama is not running
Fix:

Run:

```bash
ollama serve
```

---

## Error: Connection refused localhost:11434
This means Ollama is not running.
Start it:

```bash
ollama serve
```

---

# 10) Recommended Usage Tips

✅ Keep Ollama running in background  
✅ Download files normally (PDF, images, zip, docs)  
✅ FileGuru will automatically organize them  
✅ Use "Daily AI Report" button anytime  
✅ Use "Smart Suggestions" for next action prediction  
✅ Use "AI Insights" for productivity report  

---

# 11) What This App Does Automatically

✔ Watches folders  
✔ Organizes new files  
✔ Renames files safely  
✔ Avoids duplicates  
✔ Creates daily report  
✔ Shows AI suggestions  
✔ Summarizes PDFs  
✔ Stores memory database in JSON  

---

# 12) Files Created by the Program

Inside:

```
~/FileGuru_Organized/
```

You will see:

- `fileguru_memory.json`  (main database of organized files)
- `fileguru_hashes.json`  (duplicate detection hash database)
- `daily_report.txt`      (daily AI report)

---

# 13) Optional: Improve OCR Accuracy (Recommended)

Install English OCR extra language support:

```bash
sudo apt install tesseract-ocr-eng
```

If you want Hindi OCR support:

```bash
sudo apt install tesseract-ocr-hin
```

---

# 14) Done 🎉

Now your FileGuru AI PRO is ready.

Run:

```bash
ollama serve
```

Then run:

```bash
python3 fileguru_pro_ai.py
```

Enjoy your AI file organizer 🚀🔥


