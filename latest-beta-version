import os
import time
import shutil
import platform
import threading
import json
import requests
import hashlib
import tkinter as tk
from tkinter import ttk, scrolledtext, messagebox
from datetime import datetime, date

from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

from PIL import Image
import pytesseract
from PyPDF2 import PdfReader

APP_NAME = "FileGuru AI PRO - Smart Organizer"
MODEL_NAME = "llama3.1"
OLLAMA_URL = "http://localhost:11434/api/generate"

HOME = os.path.expanduser("~")

WATCH_FOLDERS = [
    os.path.join(HOME, "Downloads"),
    os.path.join(HOME, "Desktop"),
    os.path.join(HOME, "Documents"),
]

BASE_ORG_FOLDER = os.path.join(HOME, "FileGuru_Organized")
DB_FILE = os.path.join(BASE_ORG_FOLDER, "fileguru_memory.json")
HASH_DB_FILE = os.path.join(BASE_ORG_FOLDER, "fileguru_hashes.json")
REPORT_FILE = os.path.join(BASE_ORG_FOLDER, "daily_report.txt")


# ------------------------- UTILITIES -------------------------

def ensure_base_folders():
    os.makedirs(BASE_ORG_FOLDER, exist_ok=True)

    if not os.path.exists(DB_FILE):
        with open(DB_FILE, "w", encoding="utf-8") as f:
            json.dump([], f, indent=2)

    if not os.path.exists(HASH_DB_FILE):
        with open(HASH_DB_FILE, "w", encoding="utf-8") as f:
            json.dump({}, f, indent=2)


def safe_printable(text, limit=2000):
    if not text:
        return ""
    text = text.replace("\n", " ").replace("\r", " ")
    text = " ".join(text.split())
    return text[:limit]


def ollama(prompt):
    try:
        payload = {"model": MODEL_NAME, "prompt": prompt, "stream": False}
        r = requests.post(OLLAMA_URL, json=payload, timeout=120)
        r.raise_for_status()
        return r.json().get("response", "").strip()
    except Exception as e:
        return f"[LLM ERROR] {e}"


def sanitize_filename(name):
    bad_chars = ['\\', '/', ':', '*', '?', '"', '<', '>', '|']
    for c in bad_chars:
        name = name.replace(c, "")
    name = name.strip()
    if len(name) < 3:
        name = "file"
    return name[:80]


def compute_file_hash(path):
    try:
        sha256 = hashlib.sha256()
        with open(path, "rb") as f:
            while True:
                chunk = f.read(8192)
                if not chunk:
                    break
                sha256.update(chunk)
        return sha256.hexdigest()
    except:
        return None


def load_db():
    try:
        with open(DB_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    except:
        return []


def save_db(db):
    with open(DB_FILE, "w", encoding="utf-8") as f:
        json.dump(db[-3000:], f, indent=2)


def load_hash_db():
    try:
        with open(HASH_DB_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    except:
        return {}


def save_hash_db(hdb):
    with open(HASH_DB_FILE, "w", encoding="utf-8") as f:
        json.dump(hdb, f, indent=2)


def build_destination_path(category):
    folder = os.path.join(BASE_ORG_FOLDER, category)
    os.makedirs(folder, exist_ok=True)
    return folder


# ------------------------- TEXT EXTRACTION -------------------------

def extract_text_from_pdf(path):
    try:
        reader = PdfReader(path)
        text = ""
        for page in reader.pages[:3]:
            text += page.extract_text() or ""
        return safe_printable(text, 1500)
    except:
        return ""


def extract_text_from_image(path):
    try:
        img = Image.open(path)
        text = pytesseract.image_to_string(img)
        return safe_printable(text, 1200)
    except:
        return ""


def get_file_preview_text(path):
    ext = os.path.splitext(path)[1].lower()

    if ext == ".pdf":
        return extract_text_from_pdf(path)

    if ext in [".png", ".jpg", ".jpeg", ".bmp", ".webp"]:
        return extract_text_from_image(path)

    if ext in [".txt", ".md", ".log"]:
        try:
            with open(path, "r", encoding="utf-8", errors="ignore") as f:
                return safe_printable(f.read(1500), 1500)
        except:
            return ""

    return ""


# ------------------------- AI CLASSIFICATION -------------------------

def classify_file_with_ai(filename, ext, preview_text):
    prompt = f"""
You are FileGuru AI PRO.

You classify and rename files to organize a person's life.

FILENAME: {filename}
EXTENSION: {ext}

TEXT PREVIEW (may be empty):
{preview_text}

Return JSON ONLY.

Format:
{{
  "category": "Identity|Bank|Education|Medical|Receipts|Bills|Travel|Work|Personal|Photos|Screenshots|Software|Videos|Music|Archives|Random",
  "suggested_name": "short safe filename without extension",
  "reason": "short reason"
}}

Rules:
- suggested_name must be short and meaningful
- no special characters like : * ? < > |
- If unsure, choose Random.
"""

    response = ollama(prompt)

    try:
        start = response.find("{")
        end = response.rfind("}")
        if start != -1 and end != -1:
            data = json.loads(response[start:end + 1])
            return data
    except:
        pass

    return {
        "category": "Random",
        "suggested_name": filename.replace(ext, ""),
        "reason": "AI could not classify confidently"
    }


def summarize_document_with_ai(filename, preview_text):
    if not preview_text.strip():
        return ""

    prompt = f"""
You are FileGuru AI PRO.

Summarize this document in simple human-friendly way.
Write max 5 bullet points.

DOCUMENT NAME: {filename}
TEXT:
{preview_text}

Output format:
SUMMARY:
- point
- point
"""

    return ollama(prompt)


# ------------------------- DAILY REPORT (FIXED + BETTER) -------------------------

def generate_daily_report():
    db = load_db()
    today = date.today().strftime("%Y-%m-%d")

    todays_files = []
    for x in db:
        t = x.get("time", "")
        if t.startswith(today):
            todays_files.append(x)

    if not todays_files:
        if db:
            last_file = db[-1]
            return f"""DAILY REPORT:
- No new files organized today.
- Last activity: {last_file.get("original_file")}
- Category: {last_file.get("category")}
- Time: {last_file.get("time")}
"""
        return "DAILY REPORT:\n- No file activity found yet.\n- Start downloading files and FileGuru will organize them.\n"

    prompt = f"""
You are FileGuru AI PRO.

Create a daily report of what the user downloaded and organized today.
Be short, clean, useful.
Also include:
- Most common category
- Quick recommendation for tomorrow

DATA:
{json.dumps(todays_files[-50:], indent=2)}

Output format:
DAILY REPORT:
- ...
"""

    return ollama(prompt)


def generate_ai_next_action_prediction():
    db = load_db()

    if not db:
        return "Next Action: Download any file (PDF, image, zip) and I will auto-organize it."

    last_items = db[-20:]
    categories = [x.get("category", "Random") for x in last_items]
    most_common = max(set(categories), key=categories.count)

    prompt = f"""
You are FileGuru AI PRO.

Based on user's recent file history, predict the next action user should do.
Write ONLY one powerful suggestion line.

Example:
Next Action: Clean your Downloads folder and scan duplicate files.

DATA:
{json.dumps(last_items, indent=2)}

Most common category: {most_common}

Output format:
Next Action: ...
"""

    response = ollama(prompt)

    if "Next Action:" in response:
        return response.strip()

    return f"Next Action: You are downloading many {most_common} files. Consider cleaning and backing them up."


def generate_ai_insights():
    db = load_db()

    if not db:
        return "AI Insights:\n- No data yet. Start organizing files to unlock insights.\n"

    last_items = db[-80:]
    categories = [x.get("category", "Random") for x in last_items]

    most_common = max(set(categories), key=categories.count)

    prompt = f"""
You are FileGuru AI PRO.

Generate productivity insights from user's organized files.
Give 6 bullet points max.

DATA:
{json.dumps(last_items, indent=2)}

Output format:
AI INSIGHTS:
- ...
"""

    response = ollama(prompt)

    if "AI INSIGHTS:" not in response:
        return f"AI INSIGHTS:\n- Your most common category recently: {most_common}\n- You should consider backup and cleanup.\n"

    return response.strip()


# ------------------------- GUI -------------------------

class FileGuruUI:
    def __init__(self):
        self.root = tk.Tk()
        self.root.title(APP_NAME)
        self.root.geometry("1550x850")
        self.root.configure(bg="#0b0f1a")

        self.db_cache = load_db()

        self.setup_style()
        self.build_ui()

        self.refresh_recent_files()
        self.auto_refresh_report()

    def setup_style(self):
        style = ttk.Style()
        style.theme_use("clam")

        style.configure("TFrame", background="#0b0f1a")
        style.configure("TLabel", background="#0b0f1a", foreground="#aab4c3", font=("Segoe UI", 11))
        style.configure("Header.TLabel", font=("Segoe UI", 24, "bold"), foreground="#00ffd5", background="#0b0f1a")
        style.configure("Title2.TLabel", font=("Segoe UI", 13, "bold"), foreground="#00ff90", background="#0b0f1a")
        style.configure("TButton", font=("Segoe UI", 11, "bold"), padding=10)

        style.configure("Search.TEntry", padding=10, font=("Segoe UI", 12))

    def build_ui(self):
        header = ttk.Label(self.root, text="📂 FileGuru AI PRO", style="Header.TLabel")
        header.pack(pady=10)

        sysinfo = ttk.Label(
            self.root,
            text=f"{platform.system()} | OCR + AI Organizer | Model: {MODEL_NAME} | Organized Folder: {BASE_ORG_FOLDER}"
        )
        sysinfo.pack()

        # TOP STATS
        stats_frame = ttk.Frame(self.root)
        stats_frame.pack(fill="x", padx=15, pady=8)

        self.stats_label = ttk.Label(stats_frame, text="📊 Loading stats...", font=("Segoe UI", 12, "bold"), foreground="#ffcc00")
        self.stats_label.pack(anchor="w")

        # MAIN AREA
        main_frame = ttk.Frame(self.root)
        main_frame.pack(fill="both", expand=True, padx=15, pady=15)

        left_frame = ttk.Frame(main_frame)
        left_frame.pack(side="left", fill="both", expand=True, padx=10)

        right_frame = ttk.Frame(main_frame)
        right_frame.pack(side="right", fill="both", expand=True, padx=10)

        # SEARCH SECTION
        ttk.Label(left_frame, text="🔍 Smart Search (Type like: bank, invoice, aadhaar, resume)", style="Title2.TLabel").pack(anchor="w")
        self.search_entry = ttk.Entry(left_frame, width=65, style="Search.TEntry")
        self.search_entry.pack(anchor="w", pady=6)

        self.search_btn = ttk.Button(left_frame, text="🔍 Search", command=self.search_files)
        self.search_btn.pack(anchor="w")

        self.search_results = scrolledtext.ScrolledText(left_frame, font=("Consolas", 11), bg="#101827", fg="#ffcc00", height=12, insertbackground="white")
        self.search_results.pack(fill="both", expand=False, pady=10)

        # RECENT FILES SECTION
        ttk.Label(left_frame, text="🕒 Recently Organized Files", style="Title2.TLabel").pack(anchor="w")
        self.recent_box = scrolledtext.ScrolledText(left_frame, font=("Consolas", 11), bg="#0f172a", fg="#00ff90", height=18, insertbackground="white")
        self.recent_box.pack(fill="both", expand=True, pady=8)

        # RIGHT SIDE: LIVE LOGS
        ttk.Label(right_frame, text="📢 Live Activity Logs", style="Title2.TLabel").pack(anchor="w")
        self.log_box = scrolledtext.ScrolledText(right_frame, font=("Consolas", 11), bg="#070b14", fg="white", height=16, insertbackground="white")
        self.log_box.pack(fill="both", expand=True, pady=8)

        # NEXT ACTION PREDICTION (NEW FEATURE)
        ttk.Label(right_frame, text="🧠 AI Next Action Prediction", style="Title2.TLabel").pack(anchor="w")

        self.next_action_box = scrolledtext.ScrolledText(
            right_frame,
            font=("Segoe UI", 16, "bold"),
            bg="#121a2a",
            fg="#00ffd5",
            height=4,
            insertbackground="white"
        )
        self.next_action_box.pack(fill="both", expand=False, pady=8)

        # DAILY REPORT
        ttk.Label(right_frame, text="📅 Daily AI Report", style="Title2.TLabel").pack(anchor="w")

        self.report_box = scrolledtext.ScrolledText(
            right_frame,
            font=("Consolas", 12, "bold"),
            bg="#0f172a",
            fg="#00ffd5",
            height=8,
            insertbackground="white"
        )
        self.report_box.pack(fill="both", expand=True, pady=8)

        # AI INSIGHTS (NEW FEATURE)
        ttk.Label(right_frame, text="⚡ AI Productivity Insights", style="Title2.TLabel").pack(anchor="w")

        self.insights_box = scrolledtext.ScrolledText(
            right_frame,
            font=("Consolas", 11),
            bg="#101827",
            fg="#ffcc00",
            height=8,
            insertbackground="white"
        )
        self.insights_box.pack(fill="both", expand=True, pady=8)

        # BUTTONS
        button_frame = ttk.Frame(self.root)
        button_frame.pack(fill="x", padx=15, pady=8)

        ttk.Button(button_frame, text="📂 Open Organized Folder", command=self.open_organized_folder).pack(side="left", padx=6)
        ttk.Button(button_frame, text="🔄 Refresh Recent", command=self.refresh_recent_files).pack(side="left", padx=6)
        ttk.Button(button_frame, text="📅 Generate Daily Report", command=self.show_daily_report).pack(side="left", padx=6)
        ttk.Button(button_frame, text="🧠 Smart Suggestions", command=self.show_suggestions).pack(side="left", padx=6)
        ttk.Button(button_frame, text="⚡ AI Insights", command=self.show_insights).pack(side="left", padx=6)
        ttk.Button(button_frame, text="❌ Exit", command=self.root.destroy).pack(side="right", padx=6)

        self.status = ttk.Label(self.root, text="Status: Watching folders...", font=("Segoe UI", 12, "bold"), foreground="#00ff90")
        self.status.pack(anchor="w", padx=15, pady=8)

    def log(self, msg):
        now = datetime.now().strftime("%H:%M:%S")
        self.log_box.insert("end", f"[{now}] {msg}\n")
        self.log_box.see("end")

    def set_status(self, msg):
        self.status.config(text=f"Status: {msg}")

    def refresh_stats(self):
        db = load_db()
        total = len(db)

        today = date.today().strftime("%Y-%m-%d")
        today_count = len([x for x in db if x.get("time", "").startswith(today)])

        self.stats_label.config(text=f"📊 Total Organized Files: {total}   |   📅 Today: {today_count}")

    def refresh_recent_files(self):
        self.db_cache = load_db()
        self.recent_box.delete("1.0", "end")

        self.refresh_stats()

        recent = self.db_cache[-15:]
        if not recent:
            self.recent_box.insert("end", "No files organized yet.\n")
            return

        for item in reversed(recent):
            self.recent_box.insert("end", f"📄 {item.get('original_file')}\n➡️ {item.get('new_path')}\n📌 {item.get('category')} | {item.get('reason')}\n\n")

        self.recent_box.see("1.0")

    def search_files(self):
        query = self.search_entry.get().strip().lower()
        self.search_results.delete("1.0", "end")

        if not query:
            self.search_results.insert("end", "Type something like: invoice, aadhaar, bank statement, resume\n")
            return

        matches = []
        for item in self.db_cache[::-1]:
            combined = (
                item.get("original_file", "").lower() + " " +
                item.get("category", "").lower() + " " +
                item.get("reason", "").lower() + " " +
                item.get("preview", "").lower()
            )
            if query in combined:
                matches.append(item)

        if not matches:
            self.search_results.insert("end", "No matching file found.\n")
            return

        for item in matches[:15]:
            self.search_results.insert("end", f"✅ {item.get('original_file')}\n📂 {item.get('new_path')}\n📌 {item.get('category')} | {item.get('reason')}\n\n")

    def open_organized_folder(self):
        try:
            if platform.system() == "Windows":
                os.startfile(BASE_ORG_FOLDER)
            elif platform.system() == "Darwin":
                os.system(f'open "{BASE_ORG_FOLDER}"')
            else:
                os.system(f'xdg-open "{BASE_ORG_FOLDER}"')
        except Exception as e:
            messagebox.showerror("Error", str(e))

    def show_daily_report(self):
        self.set_status("Generating daily report...")
        self.report_box.delete("1.0", "end")

        def worker():
            report = generate_daily_report()

            self.root.after(0, lambda: self.report_box.insert("end", report + "\n"))
            self.root.after(0, lambda: self.report_box.see("end"))

            with open(REPORT_FILE, "w", encoding="utf-8") as f:
                f.write(report)

            self.root.after(0, lambda: self.set_status("Daily report generated."))

        threading.Thread(target=worker, daemon=True).start()

    def show_suggestions(self):
        self.set_status("Generating next action prediction...")
        self.next_action_box.delete("1.0", "end")

        def worker():
            suggestion = generate_ai_next_action_prediction()
            self.root.after(0, lambda: self.next_action_box.insert("end", suggestion + "\n"))
            self.root.after(0, lambda: self.next_action_box.see("end"))
            self.root.after(0, lambda: self.set_status("Next action updated."))

        threading.Thread(target=worker, daemon=True).start()

    def show_insights(self):
        self.set_status("Generating AI insights...")
        self.insights_box.delete("1.0", "end")

        def worker():
            insights = generate_ai_insights()
            self.root.after(0, lambda: self.insights_box.insert("end", insights + "\n"))
            self.root.after(0, lambda: self.insights_box.see("end"))
            self.root.after(0, lambda: self.set_status("Insights updated."))

        threading.Thread(target=worker, daemon=True).start()

    def auto_refresh_report(self):
        # Automatically refresh report + suggestions every 60 seconds
        def auto_worker():
            try:
                self.show_daily_report()
                self.show_suggestions()
                self.refresh_stats()
            except:
                pass
            self.root.after(60000, auto_worker)

        self.root.after(5000, auto_worker)

    def run(self):
        self.root.mainloop()


# ------------------------- FILE WATCHER -------------------------

class FileEventHandler(FileSystemEventHandler):
    def __init__(self, ui):
        super().__init__()
        self.ui = ui

    def on_created(self, event):
        if event.is_directory:
            return
        threading.Thread(target=self.process_file, args=(event.src_path,), daemon=True).start()

    def process_file(self, file_path):
        try:
            time.sleep(2)

            if not os.path.exists(file_path):
                return

            filename = os.path.basename(file_path)
            ext = os.path.splitext(filename)[1].lower()

            if filename.startswith("."):
                return

            if filename.endswith(".crdownload") or filename.endswith(".part"):
                return

            if filename.endswith(".tmp"):
                return

            self.ui.set_status("Analyzing new file...")
            self.ui.log(f"🆕 New file detected: {filename}")

            file_hash = compute_file_hash(file_path)
            if not file_hash:
                self.ui.log("⚠️ Could not compute file hash. Skipping.")
                self.ui.set_status("Waiting...")
                return

            hash_db = load_hash_db()
            if file_hash in hash_db:
                self.ui.log("♻️ Duplicate file detected! Moving to Duplicates folder.")

                dup_folder = build_destination_path("Duplicates")
                dest = os.path.join(dup_folder, filename)

                counter = 1
                while os.path.exists(dest):
                    dest = os.path.join(dup_folder, f"duplicate_{counter}_{filename}")
                    counter += 1

                shutil.move(file_path, dest)

                self.ui.log(f"✅ Duplicate moved: {dest}")
                self.ui.set_status("Waiting...")
                return

            preview_text = get_file_preview_text(file_path)

            ai_result = classify_file_with_ai(filename, ext, preview_text)

            category = sanitize_filename(ai_result.get("category", "Random"))
            suggested_name = sanitize_filename(ai_result.get("suggested_name", filename.replace(ext, "")))
            reason = ai_result.get("reason", "No reason")

            date_prefix = datetime.now().strftime("%Y-%m-%d")
            final_name = f"{date_prefix}_{suggested_name}{ext}"

            destination_folder = build_destination_path(category)
            destination_path = os.path.join(destination_folder, final_name)

            counter = 1
            while os.path.exists(destination_path):
                destination_path = os.path.join(destination_folder, f"{date_prefix}_{suggested_name}_{counter}{ext}")
                counter += 1

            shutil.move(file_path, destination_path)

            summary = ""
            if ext == ".pdf" and preview_text.strip():
                self.ui.log("🧠 Generating document summary (PDF)...")
                summary = summarize_document_with_ai(filename, preview_text)

            self.ui.log(f"✅ Moved to: {destination_path}")
            self.ui.log(f"📌 Category: {category}")
            self.ui.log(f"📝 Reason: {reason}")

            if summary.strip():
                self.ui.log("📌 Summary generated successfully.")
                self.ui.log(summary)

            db = load_db()
            db.append({
                "original_file": filename,
                "new_path": destination_path,
                "category": category,
                "reason": reason,
                "summary": summary,
                "time": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                "preview": preview_text[:500]
            })
            save_db(db)

            hash_db[file_hash] = destination_path
            save_hash_db(hash_db)

            self.ui.refresh_recent_files()
            self.ui.show_suggestions()
            self.ui.set_status("Waiting for new files...")

        except Exception as e:
            self.ui.log(f"❌ ERROR processing file: {e}")
            self.ui.set_status("Error occurred")


def start_watcher(ui):
    observer = Observer()
    handler = FileEventHandler(ui)

    for folder in WATCH_FOLDERS:
        if os.path.exists(folder):
            ui.log(f"👀 Watching folder: {folder}")
            observer.schedule(handler, folder, recursive=False)
        else:
            ui.log(f"⚠️ Folder not found: {folder}")

    observer.start()

    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        observer.stop()

    observer.join()


# ------------------------- MAIN -------------------------

if __name__ == "__main__":
    ensure_base_folders()

    ui = FileGuruUI()
    ui.log("🚀 FileGuru AI PRO started.")
    ui.log("⚡ Make sure Ollama is running: ollama serve")
    ui.log(f"📂 Organized Folder: {BASE_ORG_FOLDER}")

    watcher_thread = threading.Thread(target=start_watcher, args=(ui,), daemon=True)
    watcher_thread.start()

    ui.show_daily_report()
    ui.show_suggestions()
    ui.show_insights()

    ui.run()
