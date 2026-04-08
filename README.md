# SQL-injection
Blind SQL injection occurs when an application is vulnerable to SQL injection but does not return error messages or data directly. Instead, you infer information by observing behavior — like response time or true/false responses.

Time-based blind SQL injection** uses commands like `SLEEP(5)` to cause intentional delays. If the response takes 5 seconds, you know the condition was true.


