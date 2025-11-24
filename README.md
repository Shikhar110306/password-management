# password-management
python password_manager.py
import json
import os
import getpass
pip install cryptography
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
import base64

# Configuration
KEY_FILE = "key.key"
PASSWORDS_FILE = "passwords.json"

# Key Management Functions

def load_key():
    """Loads the key from the current directory or creates a new one."""
    if not os.path.exists(KEY_FILE):
        print(" Key file not found. Generating a new encryption key...")
        
        key = Fernet.generate_key()
        with open(KEY_FILE, "wb") as key_file:
            key_file.write(key)
        return key
    
    with open(KEY_FILE, "rb") as key_file:
        return key_file.read()

def get_fernet():
    """Returns the Fernet object initialized with the loaded key."""
    key = load_key()
    return Fernet(key)

# store data here

def load_passwords():
    """Loads the password data from the JSON file."""
    if not os.path.exists(PASSWORDS_FILE):
        return {}
    
    with open(PASSWORDS_FILE, "r") as f:
        try:
            return json.load(f)
        except json.JSONDecodeError:
            # file correpted
            return {}

def save_passwords(data):
    """Saves the password data to the JSON file."""
    with open(PASSWORDS_FILE, "w") as f:
        json.dump(data, f, indent=4)



def add_password(fernet):
    """Adds a new website/password entry."""
    website = input(" Enter website/service name: ").strip()
    
    
    password = getpass.getpass(" Enter password to save: ")
    
    
    encrypted_password = fernet.encrypt(password.encode()).decode()
    
    
    data = load_passwords()
    data[website] = encrypted_password
    save_passwords(data)
    
    print(f" Password for '{website}' successfully added/updated.")

def view_password(fernet):
    """Retrieves and decrypts a password for a given website."""
    website = input(" Enter website/service name to retrieve: ").strip()
    
    data = load_passwords()
    
    if website in data:
        try:
            encrypted_password = data[website].encode()
            
            decrypted_password = fernet.decrypt(encrypted_password).decode()
            print(f"🔑 Password for '{website}': **{decrypted_password}**")
        except Exception as e:
            print(f" Error decrypting password. Key might be corrupted. ({e})")
    else:
        print(f" Password for '{website}' not found.")

def list_websites():
    """Lists all stored websites/services."""
    data = load_passwords()
    if not data:
        print(" No passwords saved yet.")
        return
    
    print(" Stored Services ")
    for website in data.keys():
        print(f"* {website}")
    



def main():
    """Main function to run the password manager."""
    
    print("=")
    print("Simple Python Password Manager")
    print("=")
    
    # 1. Initialize Fernet (loads or creates key.key)
    fernet = get_fernet()

    while True:
        print("\n--- Menu ---")
        print("1. Add/Update Password")
        print("2. View Password")
        print("3. List Saved Websites")
        print("4. Exit")
        
        choice = input("Enter your choice (1-4): ")

        if choice == '1':
            add_password(fernet)
        elif choice == '2':
            view_password(fernet)
        elif choice == '3':
            list_websites()
        elif choice == '4':
            print(" Exiting Password Manager. Goodbye!")
            break
        else:
            print(" Invalid choice. Please select 1, 2, 3, or 4.")

if __name__ == "__main__":
    main()
