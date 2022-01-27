# mac-oath-mfa
Secure command line MFA for mac using oathtool and key-store

### Install
```
$ brew install oath-toolkit
```

### Setup
Add the following functions to your .bash_profile
```
function mfa () {
   local MFA_DEVICE_NAME="${1}"
   if [[ -z "${MFA_DEVICE_NAME}" ]]; then
     echo "USAGE: mfa MFA_DEVICE_NAME";
   else
     echo -n "$(date '+%H:%M:%S') "
     MFA_SECRET_KEY=$(security find-generic-password -ga "${MFA_DEVICE_NAME}" 2>&1 >/dev/null)
     oathtool --base32 --totp $(echo $MFA_SECRET_KEY | cut -f2 -d'"')
   fi
}

function mfa_add () {
   local MFA_DEVICE_NAME="${1}"
   local MFA_SECRET_KEY="${2}"
   if [[ -z "${MFA_DEVICE_NAME}" ]] || [[ -z "${MFA_SECRET_KEY}" ]]; then
     echo "USAGE: mfa_add MFA_DEVICE_NAME MFA_SECRET_KEY";
   else
     security add-generic-password -a "${MFA_DEVICE_NAME}" -s "${MFA_DEVICE_NAME}" -w "${MFA_SECRET_KEY}" && \
     echo "Secret added to keychain"
   fi
}

function mfa_delete () {
   local MFA_DEVICE_NAME="${1}"
   if [[ -z "${MFA_DEVICE_NAME}" ]]; then
     echo "USAGE: mfa_delete MFA_DEVICE_NAME";
   else
     security delete-generic-password -a "${MFA_DEVICE_NAME}" -s "${MFA_DEVICE_NAME}" 2>&1 >/dev/null
   fi
}
```

### Usage
Each command prints usage when given no options, but here's a demo....


In order to setup a virtual mfa device you need to get a base32 64 character string from the app
or site that requires MFA. In the case of AWS go to the IAM console section for MFA and begin the
process of adding a new device. Click "Show secret key for manual configuration", you’ll be presented
with a 64-character string. Use this string in the example below.

![AWS MFA Dialogue](images/aws_mfa_dialogue.png?raw=true "AWS MFA Dialogue")

1) add a new key
```
$ mfa_add aws XVWXDMXZZXJIXA3XIOHX46VXXKQEXXXI73D6SX7VXXXNVFXXIOYF5F74BYHTI7FK
Secret added to keychain
```
2) use the tool to generate codes
```
$ mfa aws
19:49:58 822269
# wait until the next minute for a new code
$ mfa aws
19:50:00 017036
```
3) Delete a virtual device
```
$ mfa_delete aws
password has been deleted.
```

PR's welcome. Enjoy!!
