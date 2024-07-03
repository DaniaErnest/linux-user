Linux User Creation Bash Script

# Log and Password File Setup

* The script ensures that the **/var/secure** directory exists and has the appropriate permissions.
* It creates the password file **/var/secure/user_passwords.csv** and ensures only the owner can read it.
```bash
mkdir -p /var/secure
chmod 700 /var/secure
touch "$PASSWORD_FILE"
chmod 600 "$PASSWORD_FILE"
```

## Message_Log Function
Logging Function: The log_message function logs messages to **/var/log/user_management.log** with a timestamp.
```bash
log_message() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE"
}
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