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
