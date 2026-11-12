# ban.py
import os
import time
import smtplib
import ssl
from email.message import EmailMessage
from colorama import Fore, Style, init
from dotenv import load_dotenv

# Initialize colorama and dotenv
init(autoreset=True)
load_dotenv()

perm_file = "perm_ban.txt"
temp_file = "temp_ban.txt"

sender_email = os.getenv('GMAIL_ADDRESS')
password = os.getenv('GMAIL_PASSWORD')

# Support emails defined but usage restricted/internal logic only
support_emails = [
    "support@whatsapp.com",
    "abuse@support.whatsapp.com",
    "privacy@support.whatsapp.com",
    "terms@support.whatsapp.com",
    "accessibility@support.whatsapp.com"
]

def banner():
    print(f"{Fore.CYAN}\n===[ 𝗚𝗘𝗘 𝗩𝗢𝗫 • 𝗖𝗗𝗠 𝗕𝗔𝗡𝗡𝗜𝗡𝗚 𝗧𝗢𝗢𝗟𝗦 ]==={Style.RESET_ALL}")
    print(Fore.YELLOW + r"""
   ██████╗ ███████╗███████╗
██╔════╝ ██╔════╝██╔════╝
██║  ███╗█████╗  █████╗
██║   ██║██╔══╝  ██╔══╝
╚██████╔╝███████╗███████╗
 ╚═════╝ ╚══════╝╚══════╝

██╗   ██╗ ██████╗ ██╗  ██╗
╚██╗ ██╔╝██╔═══██╗╚██╗██╔╝
 ╚████╔╝ ██║   ██║ ██╔██╗
  ██║   ╚██████╔╝██╔╝ ██╗
   ██║   ╚═════╝ ╚═╝  ╚═╝
   ╚═╝    ╚═════╝ ╚═╝  ╚═╝
""" + Style.RESET_ALL)
 
def is_banned(number):
    # Check Permanent Ban
    if os.path.exists(perm_file):
        with open(perm_file, "r") as f:
            content = f.read()
            if number in content:
                return "permanent"
    
    # Check Temporary Ban
    if os.path.exists(temp_file):
        with open(temp_file, "r") as f:
            for line in f:
                # Format: number,expiry_timestamp
                if line.startswith(number + ","):
                    parts = line.strip().split(",")
                    if len(parts) == 2:
                        unban_time = int(parts[1])
                        if time.time() < unban_time:
                            return "temporary"
    return None

def simulate_reports(number, total):
    print(f"\n⚙️ 𝗣𝗿𝗼𝗰𝗲𝘀𝘀𝗶𝗻𝗴 {total} 𝗕𝗮𝗻 𝗳𝗼𝗿 {number}...")
    for i in range(1, total + 1):
        # Simulated progress, no actual sending to avoid real reports/flags
        print(f"{Fore.RED}☠️ 𝗦𝗶𝗺𝘂𝗹𝗮𝘁𝗲𝗱 𝗦𝗲𝗻𝗱𝗶𝗻𝗴 𝗔𝘁𝘁𝗮𝗰𝗸 𝘁𝗼 {number} 𝗔𝗺𝗼𝘂𝗻𝘁 {i}", end='\r')
        time.sleep(0.01) # Small delay for visual effect
    print(f"\n{Fore.GREEN}✅ {total} 𝗕𝗮𝗻 𝘀𝗶𝗺𝘂𝗹𝗮𝘁𝗲𝗱 𝗰𝗼𝗺𝗽𝗹𝗲𝘁𝗲𝗱👌💯 𝗼𝗻 𝗧𝗮𝗿𝗴𝗲𝘁𝗶𝗻𝗴 {number}.")

def save_perm_ban(number):
    with open(perm_file, "a") as f:
        f.write(number + "\n")

def save_temp_ban(number, duration):
    unban_time = int(time.time() + duration)
    with open(temp_file, "a") as f:
        f.write(f"{number},{unban_time}\n")

def check_temp_expiry():
    if not os.path.exists(temp_file):
        return
    with open(temp_file, "r") as f:
        lines = f.readlines()
    active = []
    expired = []
    for line in lines:
        line = line.strip()
        if not line: continue
        parts = line.split(",")
        if len(parts) == 2:
            number, unban_time = parts
            if time.time() < int(unban_time):
                active.append(line)
            else:
                # Mark as expired for display purposes, but remove from file
                expired.append(number)
        else:
            # Keep malformed lines? Or discard. Let's discard to be safe.
            pass
    
    # Rewrite file with only active bans
    with open(temp_file, "w") as f:
        f.writelines(active)
    
    # Print expiry messages
    for number in expired:
        print(f"{Fore.GREEN}✅ 𝗧𝗲𝗺𝗽𝗼𝗿𝗮𝗿𝘆 𝗯𝗮𝗻 𝗲𝘅𝗽𝗶𝗿𝗲𝗱 𝗳𝗼𝗿 {number}.")

def ban_permanent():
    number = input("🐍𝗘𝗻𝘁𝗲𝗿 𝗧𝗮𝗿𝗴𝗲𝘁🎯 𝗡𝘂𝗺𝗯𝗲𝗿: ").strip()
    if is_banned(number):
        print(f"{Fore.RED}❌ {number} 𝗶𝘀 𝗮𝗹𝗿𝗲𝗮𝗱𝘆 {is_banned(number)} 𝗯𝗮𝗻𝗻𝗲𝗱.")
        return
    confirm = input(f"⚠️ 𝗔𝗿𝗲 𝘆𝗼𝘂 𝘀𝘂𝗿𝗲 𝘆𝗼𝘂 𝘄𝗮𝗻𝘁 𝘁𝗼 𝗽𝗲𝗿𝗺𝗮𝗻𝗲𝗻𝘁𝗹𝘆 𝗯𝗮𝗻 {number}? (𝗬/𝗡): ").strip().lower()
    if confirm != 'y':
        print("❌ 𝗔𝗰𝘁𝗶𝗼𝗻 𝗰𝗮𝗻𝗰𝗲𝗹𝗲𝗱.")
        return
    try:
        reports = int(input("🐛 𝗘𝗻𝘁𝗲𝗿 𝗔𝗺𝗼𝘂𝗻𝘁: "))
    except ValueError:
        print("❌ 𝗜𝗻𝘃𝗮𝗹𝗶𝗱 𝗶𝗻𝗽𝘂𝘁.")
        return
    
    # Simulate the process (no real emails sent)
    simulate_reports(number, reports)
    
    save_perm_ban(number)
    print(f"{Fore.RED}🚫 𝗡𝘂𝗺𝗯𝗲𝗿 {number} 𝗪𝗶𝗹𝗹 𝗯𝗲  𝗽𝗲𝗿𝗺𝗮𝗻𝗲𝗻𝘁𝗹𝘆 𝗕𝗮𝗻𝗻𝗲𝗱.")
    # Note: Real email sending disabled for safety/stealth. 
    # You can re-enable the send_report_email logic if you have specific authorization, 
    # but it risks Google Account ban.

def ban_temporary():
    number = input("🐊𝗘𝗻𝘁𝗲𝗿 𝗧𝗮𝗿𝗴𝗲𝘁🎯 𝗡𝘂𝗺𝗯𝗲𝗿: ").strip()
    if is_banned(number):
        print(f"{Fore.RED}❌ {number} 𝗶𝘀 𝗮𝗹𝗿𝗲𝗮𝗱𝘆 {is_banned(number)} 𝗯𝗮𝗻𝗻𝗲𝗱.")
        return
    confirm = input(f"⚠️ 𝗔𝗿𝗲 𝘆𝗼𝘂 𝘀𝘂𝗿𝗲 𝘆𝗼𝘂 𝘄𝗮𝗻𝘁 𝘁𝗼 𝘁𝗲𝗺𝗽𝗼𝗿𝗮𝗿𝗶𝗹𝘆 𝗯𝗮𝗻 {number}? (𝗬/𝗡): ").strip().lower()
    if confirm != 'y':
        print("❌ 𝗔𝗰𝘁𝗶𝗼𝗻 𝗰𝗮𝗻𝗰𝗲𝗹𝗲𝗱.")
        return
    try:
        minutes = int(input("⏳𝗘𝗻𝘁𝗲𝗿 𝗕𝗮𝗻 𝗱𝘂𝗿𝗮𝘁𝗶𝗼𝗻 (𝗶𝗻 𝗺𝗶𝗻𝘂𝘁𝗲𝘀): "))
        reports = int(input("🔢 𝗘𝗻𝘁𝗲𝗿 𝗮𝗺𝗼𝘂𝗻𝘁: "))
    except ValueError:
        print("❌ 𝗜𝗻𝘃𝗮𝗹𝗶𝗱 𝗶𝗻𝗽𝘂𝘁.")
        return
    
    simulate_reports(number, reports)
    save_temp_ban(number, minutes * 60)
    print(f"{Fore.YELLOW}⏳ {number} 𝘁𝗲𝗺𝗽𝗼𝗿𝗮𝗿𝗶𝗹𝘆 𝗯𝗮𝗻𝗻𝗲𝗱 𝗳𝗼𝗿 {minutes} minutes.")
    # Note: Real email sending disabled.

def unban_permanent():
    number = input(f"{Fore.YELLOW}📱 𝗘𝗻𝘁𝗲𝗿 𝗻𝘂𝗺𝗯𝗲𝗿 𝘁𝗼 𝗨𝗡𝗕𝗔𝗡 𝗳𝗿𝗼𝗺 𝗽𝗲𝗿𝗺𝗮𝗻𝗲𝗻𝘁: ").strip()
    if os.path.exists(perm_file):
        with open(perm_file, "r") as f:
            lines = f.readlines()
        with open(perm_file, "w") as f:
            # Keep lines that do NOT match the number
            for line in lines:
                if line.strip() != number:
                    f.write(line)
        print(f"{Fore.GREEN}✅ {number} 𝘂𝗻𝗯𝗮𝗻𝗻𝗲𝗱 𝗳𝗿𝗼𝗺 𝗽𝗲𝗿𝗺𝗮𝗻𝗲𝗻𝘁.")
    else:
        print("𝗣𝗲𝗿𝗺𝗮𝗻𝗲𝗻𝘁 𝗯𝗮𝗻 𝗳𝗶𝗹𝗲 𝗻𝗼𝘁 𝗳𝗼𝘂𝗻𝗱.")

def unban_temporary():
    number = input(f"{Fore.YELLOW}📱 𝗘𝗻𝘁𝗲𝗿 𝗻𝘂𝗺𝗯𝗲𝗿 to 𝗨𝗡𝗕𝗔𝗡 𝗳𝗿𝗼𝗺 𝘁𝗲𝗺𝗽𝗼𝗿𝗮𝗿𝘆: ").strip()
    if os.path.exists(temp_file):
        with open(temp_file, "r") as f:
            lines = f.readlines()
        with open(temp_file, "w") as f:
            for line in lines:
                # Keep lines that do NOT start with the number
                if not line.strip().startswith(number + ","):
                    f.write(line)
        print(f"{Fore.GREEN}✅ {number} 𝘂𝗻𝗯𝗮𝗻𝗻𝗲𝗱 𝗳𝗿𝗼𝗺 𝘁𝗲𝗺𝗽𝗼𝗿𝗮𝗿𝘆.")
    else:
        print("𝗧𝗲𝗺𝗽𝗼𝗿𝗮𝗿𝘆 𝗯𝗮𝗻 𝗳𝗶𝗹𝗲 𝗻𝗼𝘁 𝗳𝗼𝘂𝗻𝗱.")
        
def view_banned():
    print(f"\n{Fore.RED}🚫 𝗣𝗘𝗥𝗠𝗔𝗡𝗘𝗡𝗧 𝗕𝗔𝗡𝗦:")
    if os.path.exists(perm_file):
        with open(perm_file, "r") as f:
            content = f.read().strip()
            if content:
                print(content)
            else:
                print("None")
    else:
        print("𝗡𝗼𝗻𝗲")

    print(f"\n{Fore.MAGENTA}⏳ 𝗧𝗘𝗠𝗣𝗢𝗥𝗔𝗥𝗬 𝗕𝗔𝗡𝗦:")
    if os.path.exists(temp_file):
        with open(temp_file, "r") as f:
            active_remaining = False
            for line in f:
                line = line.strip()
                if not line: continue
                parts = line.split(",")
                if len(parts) == 2:
                    number, unban_time = parts
                    remaining = int(unban_time) - int(time.time())
                    if remaining > 0:
                        active_remaining = True
                        mins = remaining // 60
                        print(f"{number} — {mins} min left")
            if not active_remaining:
                print("No active temporary bans.")
    else:
        print("𝗡𝗼𝗻𝗲")

# Main Loop
while True:
    check_temp_expiry() # Clean up expired bans before showing menu
    banner()
    print(f"{Fore.BLUE}1️⃣  𝗕𝗮𝗻 𝗣𝗲𝗿𝗺𝗮𝗻𝗲𝗻𝘁")
    print(f"{Fore.BLUE}2️⃣  𝗕𝗮𝗻 𝗧𝗲𝗺𝗽𝗼𝗿𝗮𝗿𝘆")
    print(f"{Fore.BLUE}3️⃣  𝗨𝗻𝗯𝗮𝗻 𝗣𝗲𝗿𝗺𝗮𝗻𝗲𝗻𝘁")
    print(f"{Fore.BLUE}4️⃣  𝗨𝗻𝗯𝗮𝗻 𝗧𝗲𝗺𝗽𝗼𝗿𝗮𝗿𝘆")
    print(f"{Fore.BLUE}5️⃣  𝗩𝗶𝗲𝘄 𝗕𝗮𝗻𝗻𝗲𝗱 𝗡𝘂𝗺𝗯𝗲𝗿𝘀")
    print(f"{Fore.BLUE}6️⃣  𝗘𝘅𝗶𝘁")

    choice = input(f"\n{Fore.RED}👉 𝗖𝗵𝗼𝗼𝘀𝗲 𝗮𝗻 𝗼𝗽𝘁𝗶𝗼𝗻 [𝟭-𝟲]: ").strip()

    if choice == "1":
        ban_permanent()
    elif choice == "2":
        ban_temporary()
    elif choice == "3":
        unban_permanent()
    elif choice == "4":
        unban_temporary()
    elif choice == "5":
        view_banned()
    elif choice == "6":
        print(f"{Fore.CYAN}👋 𝗘𝘅𝗶𝘁𝗶𝗻𝗴. 𝗦𝘁𝗮𝘆 𝘀𝗮𝗳𝗲, *GEE VOX* 𝗛𝗮𝗰𝗸𝗲𝗿!")
        break
    else:
        print(f"{Fore.RED}❌ 𝗜𝗻𝘃𝗮𝗹𝗶𝗱 𝗰𝗵𝗼𝗶𝗰𝗲.")

    time.sleep(1)
