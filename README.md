## Linux User Creation Bash Script

### Log and Password File Setup
```bash
LOG_FILE="/var/log/user_management.log"
SECURE_PASSWORD_FILE="/var/secure/user_passwords.txt"
SECURE_PASSWORD_CSV="/var/secure/user_passwords.csv"
```

* The script ensures that the **/var/secure** directory exists and has the appropriate permissions.
* It creates the password file **/var/secure/user_passwords.csv** and ensures only the owner can read it.
```bash
sudo mkdir -p /var/log
sudo mkdir -p /var/secure
sudo touch "$LOG_FILE"
sudo touch "$SECURE_PASSWORD_FILE"
sudo touch "$SECURE_PASSWORD_CSV"
sudo chmod 600 "$SECURE_PASSWORD_FILE"
sudo chmod 600 "$SECURE_PASSWORD_CSV"

```

### Message_Log Function
The log_message function logs messages to **/var/log/user_management.log** with a timestamp.
```bash
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | sudo tee -a "$LOG_FILE" > /dev/null
}
```

### generate_password Function
The generate_password function generates a random password of a specified length (12 characters in this case).
```bash
generate_random_password() {
    < /dev/urandom tr -dc A-Za-z0-9 | head -c12
}
```


### Function to validate input format for username and groups
```bash
validate_input() {
    local username=$1
    local groups=$2

    if [[ -z "$username" || -z "$groups" ]]; then
        log "Error: Invalid input. Username and groups are required."
        send_slack_notification "Invalid input provided. Username and groups are required."
        exit $E_INVALID_INPUT
    fi
}
```

### Setup_User Function
The create_user function creates users, adds them to groups, sets up home directories with appropriate permissions, and logs each action. It also generates and stores passwords securely.
```bash
# Function to create a user and set up their home directory
create_user() {
    local username=$1
    local password=$2

    # Check if the user already exists
    if id "$username" &>/dev/null; then
        log "User $username already exists. Skipping user creation."
    else
        log "Creating user $username."
        # Attempt to create the user with a timeout
        timeout 10 sudo useradd -m -s /bin/bash "$username" || {
            log "Failed to create user $username."
            # send_slack_notification "Failed to create user $username."
            exit $E_USER_CREATION_FAILED
        }
        # Set the user's password and home directory permissions
        echo "$username:$password" | sudo chpasswd
        sudo chmod 700 "/home/$username"
        sudo chown "$username:$username" "/home/$username"
        log "User $username created successfully with password $password."
        echo "$username:$password" | sudo tee -a "$SECURE_PASSWORD_FILE" > /dev/null
        echo "$username,$password" | sudo tee -a "$SECURE_PASSWORD_CSV" > /dev/null
    fi
}
```

### Function to create a group
```bash
create_group() {
    local groupname=$1

    # Check if the group already exists
    if getent group "$groupname" &>/dev/null; then
        log "Group $groupname already exists."
    else
        log "Creating group $groupname."
        # Attempt to create the group with a timeout
        timeout 10 sudo groupadd "$groupname" || {
            log "Failed to create group $groupname."
            send_slack_notification "Failed to create group $groupname."
            exit $E_GROUP_CREATION_FAILED
        }
        log "Group $groupname created successfully."
    fi
}
```

### Function to add a user to a group
```bash
add_user_to_group() {
    local username=$1
    local groupname=$2

    # Check if the user is already a member of the group
    if id -nG "$username" | grep -qw "$groupname"; then
        log "User $username is already a member of $groupname."
    else
        log "Adding user $username to group $groupname."
        # Attempt to add the user to the group with a timeout
        timeout 10 sudo usermod -aG "$groupname" "$username" || {
            log "Failed to add user $username to group $groupname."
            # send_slack_notification "Failed to add user $username to group $groupname."
            exit $E_ADD_USER_TO_GROUP_FAILED
        }
        log "User $username added to group $groupname successfully."
    fi
}
```bash


### Main Script
The main part of the script takes an input file as an argument, reads it line by line, and processes each line to create users and groups, set up home directories, and log actions.
```bash
if [[ $# -ne 1 ]]; then
    echo "Usage: $0 <users_file>"
    exit 10  # Invalid input exit code
fi
```
### This makes sure you run the script with an input_file, i.e input.txt
```bash
    users_file="$1"
```

### Usage
To use this script, save it to a file (e.g., create_users.sh), and create a file `input.txt` `input file containing user;groups entries`.

Change the user to root `sudo -s`, make the file executable, and run it with the path to your input file as an argument:

input.txt
```bash
    user1; group1,group2
    user2; group3,group4
```

on the Command Line(CMD) | Terminal
```bash
    sudo -s
    chmod +x user_management.sh
    ./create_users.sh input.txt
```

## Conclusion

This script should be run with sudo to ensure it has the necessary permissions to create users, modify groups, and write to the specified log and password files.


## verify the script
```bash
  ~ cd /var/log/
  ~ ls
  ~ user_management.log
  ~ cd /var/secure/
  ~ ls
  ~ user_passwords.csv
  ~ user_passwords.txt

```
