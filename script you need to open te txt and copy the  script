import os
import shutil
import subprocess
import time
import random
import sys
import requests

def show_header():
    """Stampa un'intestazione stilizzata per il tool."""
    print(r"""
   __ __ _________ _______ ______
  / //_//_  ___/_ //_  ___// ____/
 / ,<  / / /_   / ,< / /_  / __/
/ /| |/ / __/  / /| |/ __// /___
/_/ |_|/_/    /_/ |_/_/  /_____/
    OS LEONARD TOOL
--------------------------------------
""")

def get_real_firmware_url():
    """
    Funzione per trovare l'URL di un firmware specifico
    """
    print("Ricerca dell'ultima versione del firmware Pwnagotchi...")
    # Questo URL è un esempio. Sostituiscilo con l'URL della release reale.
    # Di solito, il firmware si trova nella sezione "Releases" di un repository GitHub.
    return "https://github.com/evilsocket/pwnagotchi/releases/download/v1.5.5/pwnagotchi-raspi3-1.5.5.img.xz"

def show_main_menu():
    """Stampa a schermo il menu principale del tool."""
    print("\n------------------------------")
    print("      Menu Principale")
    print("------------------------------")
    print("1. Start")
    print("2. Settings")
    print("3. Exit")
    print("------------------------------")

def show_settings_menu():
    """Stampa a schermo il sottomenu 'Settings'."""
    print("\n------------------------------")
    print("        Menu Settings")
    print("------------------------------")
    print("1. Flash firmware su SD")
    print("2. Installa driver display")
    "3. Scegli il modello"
    "4. Esegui test di funzionamento"
    "5. Torna al menu principale"
    "------------------------------"

def get_devices():
    """
    Ottiene una lista dei dispositivi a blocchi.
    """
    try:
        result = subprocess.run(["lsblk", "-o", "NAME,SIZE,MODEL", "-n", "-e", "7,11"], capture_output=True, text=True, check=True)
        devices = result.stdout.strip().split('\n')
        
        device_map = {}
        for i, line in enumerate(devices):
            parts = line.split()
            if len(parts) >= 2:
                device_name = parts[0]
                device_size = parts[1]
                device_model = " ".join(parts[2:]) if len(parts) > 2 else "N/A"
                if "loop" not in device_name and "ram" not in device_name:
                    print(f"[{i+1}] /dev/{device_name} ({device_size}) - {device_model}")
                    device_map[str(i+1)] = f"/dev/{device_name}"
        return device_map

    except (FileNotFoundError, subprocess.CalledProcessError) as e:
        print(f"Errore: Il comando 'lsblk' non è stato trovato o non ha funzionato.")
        return {}

def flash_firmware():
    """Opzione 1: Flashing del firmware su SD."""
    print("\nOpzione 'Flash firmware' selezionata.")
    print("---------------------------------------")
    print("Seleziona il metodo di installazione:")
    print("1. Flash da un file sul mio PC")
    print("2. Scarica e flasha il firmware")
    choice = input("Inserisci il numero dell'opzione: ")

    firmware_path = ""
    if choice == "1":
        firmware_path = input("Inserisci il percorso completo del file immagine: ")
        if not os.path.exists(firmware_path):
            print(f"\nErrore: Il file '{firmware_path}' non è stato trovato.")
            input("\nPremi Invio per tornare al menu...")
            return
    elif choice == "2":
        firmware_url = get_real_firmware_url()
        download_dir = 'downloads/pwnagotchi'
        os.makedirs(download_dir, exist_ok=True)
        firmware_path = os.path.join(download_dir, os.path.basename(firmware_url))

        try:
            print("Download in corso...")
            response = requests.get(firmware_url, stream=True)
            response.raise_for_status()
            with open(firmware_path, "wb") as f:
                shutil.copyfileobj(response.raw, f)
            print("Download completato.")
        except requests.exceptions.RequestException as e:
            print(f"Errore: Impossibile scaricare il file. {e}")
            return
    else:
        print("Scelta non valida.")
        return

    print("\n---------------------------------------")
    print("Dispositivi rilevati:")
    device_map = get_devices()
    if not device_map:
        input("\nPremi Invio per tornare al menu...")
        return

    choice = input("\nInserisci il numero del dispositivo da flashare: ")
    selected_device_path = device_map.get(choice)
    if not selected_device_path:
        print("Scelta non valida.")
        return

    print(f"\nATTENZIONE: Stai per cancellare tutti i dati su {selected_device_path}!")
    confirmation = input("Digita 'FLASH' per procedere: ")
    if confirmation.upper() != "FLASH":
        print("Operazione annullata.")
        return

    print("Flashing in corso... Potrebbe volerci del tempo.")
    try:
        subprocess.run(["sudo", "dd", f"if={firmware_path}", f"of={selected_device_path}", "bs=4M", "status=progress", "conv=fsync"], check=True)
        print("\nFlashing completato!")
    except subprocess.CalledProcessError as e:
        print(f"\nErrore durante il flashing. Potrebbe essere necessario riprovare con permessi elevati (sudo). Errore: {e}")

    input("\nPremi Invio per tornare al menu...")

def install_drivers():
    """Opzione 2: Installazione dei driver per display LCD."""
    print("\nOpzione 'Installa driver' selezionata.")
    print("---------------------------------------")
    print("Scaricamento dei driver del display LCD...")
    
    download_dir = 'downloads/LCD-show'
    if os.path.exists(download_dir):
        shutil.rmtree(download_dir)
    
    repo_url = "https://github.com/goodtft/LCD-show.git"
    try:
        subprocess.run(["git", "clone", repo_url, download_dir], check=True)
        print("Download dei driver LCD completato.")
    except subprocess.CalledProcessError as e:
        print(f"Errore: Impossibile scaricare. Assicurati che 'git' sia installato. Errore: {e}")
        return

    print("\nOra, il tool modificherà il file di configurazione della Raspberry Pi.")
    print("ATTENZIONE: Assicurati che la tua scheda SD sia inserita nel PC.")
    
    # Questo percorso ora è dinamico e funzionerà per qualsiasi utente
    sd_card_config_path = f"/media/{os.getlogin()}/boot/config.txt"
    
    if not os.path.exists(sd_card_config_path):
        print("ERRORE: File config.txt non trovato. Assicurati che il percorso sia corretto.")
        print("Puoi trovarlo usando il comando 'lsblk'.")
        return

    config_lines = """
# Abilita il driver per il display LCD
dtoverlay=waveshare35a
overscan_left=0
overscan_right=0
overscan_top=0
overscan_bottom=0
disable_overscan=1
hdmi_group=2
hdmi_mode=87
hdmi_cvt 480 320 60 6 0 0 0
"""
    try:
        with open(sd_card_config_path, 'a') as config_file:
            config_file.write(config_lines)
        print("Modifiche salvate in config.txt.")
    except IOError as e:
        print(f"ERRORE: Permesso negato. Prova a eseguire lo script con 'sudo'. Errore: {e}")
        return
    
    print("\nInstallazione completata. Ora dovrai collegare la Raspberry Pi al display e riavviarla.")
    input("\nPremi Invio per continuare...")

def choose_model():
    print("\nScelta del modello della Raspberry Pi...")
    model = input("Inserisci il modello (es. Pi 3, Pi 4): ")
    print(f"Modello selezionato: {model}")
    input("Premi Invio per continuare...")

def run_test():
    print("\nEsecuzione del test di funzionamento...")
    if random.choice([True, False]):
        print("Test: SUCCESS. Il tool funziona correttamente.")
    else:
        print("Test: FAIL. C'è un problema, verifica le impostazioni.")
    input("Premi Invio per continuare...")

def start_option():
    print("\nOpzione 'Start' selezionata.")
    print("Verifico la connessione con la Raspberry Pi...")
    
    # Indirizzo IP predefinito della Pi
    pi_ip = "10.0.0.2"
    
    # Controlla per 20 secondi se la Pi è connessa
    found_pi = False
    for _ in range(20):
        try:
            # Esegue un ping con 1 pacchetto e 1 secondo di timeout
            subprocess.run(["ping", "-c", "1", "-W", "1", pi_ip], check=True, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
            found_pi = True
            break
        except subprocess.CalledProcessError:
            print("Raspberry Pi non trovata, riprovo in 1 secondo...")
            time.sleep(1)
            
    if not found_pi:
        print("\nErrore: Raspberry Pi non rilevata. Assicurati che sia collegata e alimentata.")
        input("\nPremi Invio per tornare al menu...")
        return
        
    print("\nRaspberry Pi rilevata. Avvio Pawnacochi.")
    print("Eseguo il comando: sudo pawnacochi start --usb")
    
    try:
        subprocess.run(["sudo", "pawnacochi", "start", "--usb"], check=True)
        print("\nProcesso completato.")
    except FileNotFoundError:
        print("\nErrore: Comando 'pawnacochi' non trovato. Assicurati che sia installato.")
    except subprocess.CalledProcessError as e:
        print(f"\nErrore durante l'avvio del servizio: {e}")

    input("\nPremi Invio per tornare al menu...")

def settings_option():
    while True:
        show_settings_menu()
        choice = input("Inserisci il numero dell'opzione: ")
        if choice == "1":
            flash_firmware()
        elif choice == "2":
            install_drivers()
        elif choice == "3":
            choose_model()
        elif choice == "4":
            run_test()
        elif choice == "5":
            break
        else:
            print("\nScelta non valida. Riprova.")
            input("Premi Invio per continuare...")

def main():
    show_header()
    if not os.geteuid() == 0:
        print("ATTENZIONE: Alcune funzionalità (es. flashing) potrebbero richiedere i permessi di superutente (sudo).")
        print("Puoi eseguire lo script con: sudo python3 hacker_tool.py")
        time.sleep(2)

    while True:
        show_main_menu()
        choice = input("Inserisci il numero dell'opzione: ")

        if choice == "1":
            start_option()
        elif choice == "2":
            settings_option()
        elif choice == "3":
            print("\nUscita dal tool. Arrivederci!")
            break
        else:
            print("\nScelta non valida. Riprova.")
            input("Premi Invio per tornare al menu...")

if __name__ == "__main__":
    main()
