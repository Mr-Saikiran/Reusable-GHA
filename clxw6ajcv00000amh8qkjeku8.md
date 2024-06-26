---
title: "Secure Password Storage in Python: A Guide to Keyring"
datePublished: Mon Oct 10 2022 18:30:00 GMT+0000 (Coordinated Universal Time)
cuid: clxw6ajcv00000amh8qkjeku8
slug: secure-password-storage-in-python-a-guide-to-keyring
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1719426251948/f62ef4af-bb37-49c7-91e7-a56933e461f3.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1719426707520/cc2006ca-cb74-46cc-8f9b-3d490dcc0710.jpeg
tags: python, security, passwords, keyring

---

```python
import os
import keyring


def set_password(service_name, username, password):
    try:
        # Set the password in the keyring
        keyring.set_password(service_name, username, password)
        print("Password set for service '{}' and username '{}'".format(service_name, username))
    except keyring.errors.KeyringError as e:
        # Handle any errors that occur during password setting
        print("Error setting password: '{}'".format(str(e)))


def get_password(service_name, username):
    try:
        # Retrieve the password from the keyring
        password = keyring.get_password(service_name, username)
        if password:
          	print("Password set for service '{}' and username '{}'".format(service_name, username))
        else:
            print("No password found '{}' and username '{}'".format(service_name, username))
    except keyring.errors.KeyringError as e:
        # Handle any errors that occur during password retrieval
        print("Error retrieving password: '{}'".format(str(e)))


# Usage example
service_name = 'test_svcaccount'
username = 'test-user'
pssword = os.environ['SECRET_PARAM']
#pssword = 'fhvjh'

# Set the password
set_password(service_name, username, pssword)

# Get the password
get_password(service_name, username)
```