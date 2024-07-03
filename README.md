## Linux User Creation Bash Script

### Log and Password File Setup

* The script ensures that the **/var/secure** directory exists and has the appropriate permissions.
* It creates the password file **/var/secure/user_passwords.csv** and ensures only the owner can read it.
```bash
mkdir -p /var/secure
chmod 700 /var/secure
touch "$PASSWORD_FILE"
chmod 600 "$PASSWORD_FILE"
```

### Message_Log Function
The log_message function logs messages to **/var/log/user_management.log** with a timestamp.
```bash
log_message() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE"
}
```

### generate_password Function
The generate_password function generates a random password of a specified length (12 characters in this case).
```bash
generate_password() {
    local password_length=12
    tr -dc A-Za-z0-9 </dev/urandom | head -c $password_length
}
```
### Setup_User Function
The setup_user function creates users, adds them to groups, sets up home directories with appropriate permissions, and logs each action. It also generates and stores passwords securely.
```bash
setup_user() {
    local username=$1
    local groups=$2

    # Create the user
    if ! id -u "$username" &>/dev/null; then
        password=$(generate_password)
        useradd -m -s /bin/bash "$username"
        echo "$username:$password" | chpasswd
        log_message "User $username created."

        # Store the username and password
        echo "$username,$password" >> "$PASSWORD_FILE"
        log_message "Password for $username stored."
    else
        log_message "User $username already exists."
    fi
```

### Main Script
The main part of the script takes an input file as an argument, reads it line by line, and processes each line to create users and groups, set up home directories, and log actions.
```bash
if [ $# -eq 0 ]; then
    log_message "Usage: $0 <input_file>"
    exit 1
fi
```
### This makes sure you run the script with an input_file, i.e input.txt
```bash
    input_file=$1
    log_message "Starting user management script."
```

### Usage
To use this script, save it to a file (e.g., create_users.sh), and create a file `input.txt` `input file containing user;groups entries`.

Change the user to root, make the file executable, and run it with the path to your input file as an argument:

input.txt
```bash
    user1;group1,group2
    user2;group3,group4
```

on the Command Line(CMD) | Terminal
```bash
    sudo -s
    chmod +x user_management.sh
    ./create_users.sh input.txt
```

## Conclusion

In this module, we have seen how to configure EKS to provide finer access to users combining IAM Groups and Kubernetes RBAC.
You can create different groups depending on your needs, configure their associated RBAC access in your cluster, and simply add or remove users from the group to grant or revoke access to your cluster.

Users will only have to configure their AWS CLI in order to automatically retrieve their associated rights in your cluster.


## Clean Up

rm /tmp/*.json
rm /tmp/kubeconfig*

# reset aws credentials and config files
rm  ~/.aws/{config,credentials}
aws configure set default.region ${AWS_REGION}