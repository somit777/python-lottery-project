import time
import random
import threading
import os
from datetime import datetime, timedelta

REGISTER_DURATION = 60 * 60
EXTENDED_DURATION = 30 * 60
SAVE_INTERVAL = 5 * 60
TIME_UPDATE_INTERVAL = 10 * 60

LOG_FILE = "lottery_log.txt"
USER_FILE = "registered_users.txt"

registered_users = set()
start_time = time.time()

def save_progress():
    with open(USER_FILE, "w") as f:
        for user in registered_users:
            f.write(f"{user}\n")

def log_event(message):
    with open(LOG_FILE, "a") as log:
        log.write(f"[{datetime.now()}] {message}\n")

def load_previous_users():
    if os.path.exists(USER_FILE):
        with open(USER_FILE, "r") as f:
            for line in f:
                registered_users.add(line.strip())

def is_valid_username(username):
    return username.isalnum() and username != ""

def register_users(end_time):
    print("=== Lottery Registration Started ===")
    while time.time() < end_time:
        remaining = int(end_time - time.time())
        print(f"\nTime left for registration: {remaining // 60} minutes")
        print(f"Currently registered users: {len(registered_users)}")

        username = input("Enter your username: ").strip()

        if not is_valid_username(username):
            print("Invalid username. Use only letters and numbers, no spaces.")
            continue

        if username in registered_users:
            print("This username is already registered. Try a different one.")
        else:
            registered_users.add(username)
            log_event(f"User registered: {username}")
            print(f"{username} registered successfully.")

        time.sleep(1)

def auto_save_thread():
    while True:
        time.sleep(SAVE_INTERVAL)
        save_progress()
        log_event("Auto-saved registered users.")

def show_periodic_timer():
    while True:
        time.sleep(TIME_UPDATE_INTERVAL)
        time_left = max(0, int((end_time - time.time()) // 60))
        print(f"[INFO] {time_left} minutes left to register.")

def draw_winner():
    if len(registered_users) == 0:
        log_event("No users registered. Lottery canceled.")
        print("No users registered. Exiting...")
        return

    winner = random.choice(list(registered_users))
    log_event(f"Winner Selected: {winner}")
    log_event(f"Total Participants: {len(registered_users)}")

    print("\n=== 🎉 Lottery Winner Announcement 🎉 ===")
    print(f"Total Participants: {len(registered_users)}")
    print(f"🏆 Winner: {winner}")
    print("========================================")

load_previous_users()

threading.Thread(target=auto_save_thread, daemon=True).start()

end_time = start_time + REGISTER_DURATION
threading.Thread(target=show_periodic_timer, daemon=True).start()
register_users(end_time)

if len(registered_users) < 5:
    print("\nLess than 5 users registered. Extending registration by 30 minutes.")
    log_event("Extended registration due to low user count.")
    end_time += EXTENDED_DURATION
    register_users(end_time)

if len(registered_users) == 0:
    print("Still no users registered. Exiting.")
    log_event("No users after extension. Exiting.")
else:
    save_progress()
    draw_winner()
