import can
import sqlite3
import tkinter as tk
from datetime import datetime
import threading

# ---------------------------------------------------
# DATABASE SETUP (SQLite)
# ---------------------------------------------------
conn = sqlite3.connect("stand_status.db", check_same_thread=False)
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS can_logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp TEXT,
    arbitration_id INTEGER,
    data_hex TEXT,
    stand_status TEXT
)
""")
conn.commit()

def log_to_db(timestamp, arbitration_id, data_hex, status):
    cursor.execute("""
        INSERT INTO can_logs (timestamp, arbitration_id, data_hex, stand_status)
        VALUES (?, ?, ?, ?)
    """, (timestamp, arbitration_id, data_hex, status))
    conn.commit()

# ---------------------------------------------------
# Initialize CAN interface
# ---------------------------------------------------
bus = can.interface.Bus(channel='can0', bustype='socketcan')

# Tkinter setup
root = tk.Tk()
root.title("Stand Status Monitor")
root.geometry("400x200")

status_label = tk.Label(root, text="Waiting for CAN data...", font=("Arial", 20))
status_label.pack(pady=30)

time_label = tk.Label(root, text="", font=("Arial", 12))
time_label.pack()

stand_status = {"text": "Waiting for CAN data...", "color": "black", "timestamp": ""}

# ---------------------------------------------------
# GUI refresh function
# ---------------------------------------------------
def refresh_gui():
    status_label.config(text=stand_status["text"], fg=stand_status["color"])
    time_label.config(text=f"Last update: {stand_status['timestamp']}")
    root.after(100, refresh_gui)

# ---------------------------------------------------
# CAN listener thread
# ---------------------------------------------------
def can_listener():
    while True:
        try:
            message = bus.recv(timeout=1.0)

            if message and message.arbitration_id == 0x775:
                data_hex = message.data.hex().upper()

                if data_hex.startswith("20"):
                    status_text = "STAND ON 🟢"
                    status_color = "green"
                else:
                    status_text = "STAND OFF 🔴"
                    status_color = "red"

                timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

                # Update shared GUI state
                stand_status["text"] = status_text
                stand_status["color"] = status_color
                stand_status["timestamp"] = timestamp

                # LOG TO DATABASE
                log_to_db(timestamp, message.arbitration_id, data_hex, status_text)

        except Exception as e:
            print("CAN read error:", e)

# Start threads
threading.Thread(target=can_listener, daemon=True).start()
refresh_gui()
root.mainloop()
