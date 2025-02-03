import json
import time
import threading
import requests
import tkinter as tk
from tkinter import font
from datetime import datetime

# Telegram API nustatymai
TELEGRAM_BOT_TOKEN = "YOUR_BOT_TOKEN_HERE"
CHAT_ID = "YOUR_CHAT_ID_HERE"

# JSON failai
DARBAI_FAILAS = "namu_darbai.json"

# Funkcija įkelti duomenis iš JSON
def uzkrauti_duomenis():
    try:
        with open(DARBAI_FAILAS, "r") as failas:
            duomenys = json.load(failas)
            if isinstance(duomenys, list):  # Patikriname, ar tai sąrašas
                return duomenys
            else:
                return []
    except (FileNotFoundError, json.JSONDecodeError):
        return []

# Įkeliame esamus duomenis
namu_darbai = uzkrauti_duomenis()

# Funkcija siųsti pranešimą į Telegram
def siusti_telegram_zinute(pranesimas):
    url = f"https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendMessage"
    params = {"chat_id": CHAT_ID, "text": pranesimas, "parse_mode": "Markdown"}
    requests.get(url, params=params)

# Funkcija pridėti naują darbą
def prideti_darba():
    pavadinimas = ivedimo_laukas.get().strip()
    terminas = termino_laukas.get().strip()

    try:
        datetime.strptime(terminas, "%Y-%m-%d %H:%M")  # Patikrinam formato teisingumą
    except ValueError:
        saraso_laukas.insert(tk.END, "❌ Klaida: Netinkamas datos formatas! YYYY-MM-DD HH:MM\n")
        return

    if pavadinimas and terminas:
        naujas_darbas = {"pavadinimas": pavadinimas, "terminas": terminas}
        namu_darbai.append(naujas_darbas)
        issaugoti_faila()
        atnaujinti_sarasa()
        siusti_telegram_zinute(f"✅ **Pridėta užduotis:** {pavadinimas}\n📅 **Terminas:** {terminas}")

# Funkcija atnaujinti sąrašą lange
def atnaujinti_sarasa():
    saraso_laukas.delete("1.0", tk.END)
    if namu_darbai:
        sarasas = "\n".join(
            f"📌 {d.get('pavadinimas', 'Nežinomas')} - ⏳ {d.get('terminas', 'Nežinomas')}"
            for d in namu_darbai if isinstance(d, dict)  # Tikriname ar d yra žodynas
        )
        saraso_laukas.insert(tk.END, sarasas)
    else:
        saraso_laukas.insert(tk.END, "📝 Nėra namų darbų")

# Funkcija tikrinti terminus
def tikrinti_terminus():
    while True:
        dabar = datetime.now()
        for darbas in namu_darbai:
            if isinstance(darbas, dict) and "terminas" in darbas:  # Patikriname ar tai žodynas
                try:
                    terminas = datetime.strptime(darbas["terminas"], "%Y-%m-%d %H:%M")
                    liko_laiko = (terminas - dabar).total_seconds()

                    if 0 < liko_laiko <= 3600:  # Jei liko mažiau nei 1 valanda
                        siusti_telegram_zinute(f"⏳ *Priminimas:* Užduotis **{darbas['pavadinimas']}** turi būti atlikta per 1 valandą!")

                except ValueError:
                    print(f"Klaida: neteisingas datos formatas {darbas.get('terminas', 'Nežinomas terminas')}")
            else:
                print(f"Klaida: netinkama duomenų struktūra - {darbas}")

        time.sleep(900)  # Tikriname kas 15 minučių

# Funkcija išsaugoti duomenis
def issaugoti_faila():
    with open(DARBAI_FAILAS, "w") as failas:
        json.dump(namu_darbai, failas, indent=4)

# Grafinė sąsaja
langas = tk.Tk()
langas.title("Vaiko namų darbai")
langas.geometry("450x450")

custom_font = font.Font(size=12)

ivedimo_laukas = tk.Entry(langas, width=50, font=custom_font)
ivedimo_laukas.pack()

termino_laukas = tk.Entry(langas, width=50, font=custom_font)
termino_laukas.insert(0, "YYYY-MM-DD HH:MM")
termino_laukas.pack()

btn_prideti = tk.Button(langas, text="Pridėti namų darbą", command=prideti_darba, font=custom_font)
btn_prideti.pack()

saraso_laukas = tk.Text(langas, width=50, height=10, font=custom_font)
saraso_laukas.pack()

btn_atlikta = tk.Button(langas, text="Pažymėti kaip atliktą", command=lambda: saraso_laukas.insert(tk.END, "✅ Atlikta!\n"), font=custom_font)
btn_atlikta.pack()

atnaujinti_sarasa()
threading.Thread(target=tikrinti_terminus, daemon=True).start()

langas.mainloop()
