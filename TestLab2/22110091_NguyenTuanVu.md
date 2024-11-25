# Lab#2: File Transfer and Encryption
**Student ID**: 22110091  
**Name**: Nguyen Tuan Vu  
**Course**: INSE33030E_01FIE

---
# Task 1: Transfer files between computers  
**Question 1**: 
Conduct transfering a single plaintext file between 2 computers, 
Using openssl to implementing measures manually to ensure file integerity and authenticity at sending side, 
then veryfing at receiving side. 

**Answer 1**:


#### Setup
We have a network with two computers:  
- **Alice**: IP 10.9.0.5  
- **Bob**: IP 10.9.0.6

### Steps on Alice's Side

1. **Create a file**: On Alice's machine, create a plaintext file `file.txt` with the following content:
    ```bash
    echo "This file is used for transferring" > file.txt
    ```
   ![File Created](https://github.com/user-attachments/assets/0f0cdc8b-4802-486e-9021-7123ae05434d)

2. **Encrypt the file**: Encrypt the file using AES-256-ECB symmetric encryption with a shared secret key `11223344551122334455112233445511`:
    ```bash
    openssl enc -aes-256-ecb -in file.txt -out file.enc -pass pass:11223344551122334455112233445511
    ```
    This command does the following:
    - **`openssl enc`**: Invokes OpenSSL's encryption functionality to perform encryption or decryption.
    - **`-aes-256-ecb`**: Specifies AES encryption with a 256-bit key in ECB (Electronic Codebook) mode.
    - **`-in file.txt`**: Specifies the input file (`file.txt`) to encrypt.
    - **`-out file.enc`**: Specifies the output file (`file.enc`) where the encrypted data will be stored.
    - **`-pass pass:11223344551122334455112233445511`**: Provides the secret key for encryption using the passphrase         `11223344551122334455112233445511`.

    ![Encryption Process](https://github.com/user-attachments/assets/4c87e603-a7bd-457f-a5c2-7404e3fe11e3)


3. **Generate HMAC**: Create a Hash-based Message Authentication Code (HMAC) to ensure authenticity using the same shared secret key:
    ```bash
    openssl dgst -sha256 -hmac "11223344551122334455112233445511" -binary file.enc > file.enc.hmac
    ```
    This command does the following:

    - **`openssl dgst`**: Invokes the `openssl dgst` command, which is used to compute message digests (hashes) of files or strings.
    - **`-sha256`**: Specifies that the SHA-256 hashing algorithm should be used to compute the digest.
    - **`-hmac "11223344551122334455112233445511"`**: Uses the specified key (`11223344551122334455112233445511`) to compute the HMAC         (Hash-based Message Authentication Code) of the file.
    - **`-binary`**: Outputs the hash in binary format, rather than hexadecimal.
    - **`file.enc`**: The input file (`file.enc`) whose HMAC hash is being computed.
    - **`> file.enc.hmac`**: Redirects the output (the HMAC hash) to the file `file.enc.hmac`.
      
    ![image](https://github.com/user-attachments/assets/abeef4bc-04e0-4742-b59f-58a7e96e15ec)

Now, Alice has two files:
- `file.enc`: The encrypted file.
- `file.enc.hmac`: The HMAC for authenticity and integrity.

4. **Setup SSH on Bob**: Install OpenSSH on Bob's machine and start the SSH service:
    ```bash
    apt update -y
    apt install openssh-server
    service ssh start
    ```
    ![SSH Start](https://github.com/user-attachments/assets/2eb2f516-c425-4d96-8ac4-88f77522f19f)

5. **Setup SSH on Alice**: Install OpenSSH on Alice's machine and start the service:
    ```bash
    apt update -y
    apt install openssh-server
    service ssh start
    ```
    ![SSH Setup](https://github.com/user-attachments/assets/71358c7d-35e0-4739-98fb-59cc31d7e881)

6. **Change Folder Permissions**: Modify file permissions on Bob’s home directory for convenience:
    ```bash
    chmod 777 /home
    ```
    ![Permission Change](https://github.com/user-attachments/assets/5c23c076-4129-4e1a-8c7a-6df7ff30eb17)

7. **Transfer the Files Using SCP**: Use `scp` to transfer the files from Alice to Bob:
    ```bash
    scp file.enc file.enc.hmac bob@10.9.0.6:/home/
    ```
    This command does the following:

    - **`scp`**: Uses the `scp` (secure copy) command to securely transfer files between hosts over SSH.
    - **`file.enc`**: Specifies the first file (`file.enc`) to be copied from the local machine.
    - **`file.enc.hmac`**: Specifies the second file (`file.enc.hmac`) to be copied from the local machine.
    - **`bob@10.9.0.6:/home/`**: Specifies the remote destination, where the files will be copied. The files will be transferred to         the `/home/` directory of the remote user `bob` on the machine with IP address `10.9.0.6`.
      
    ![image](https://github.com/user-attachments/assets/faff5816-b874-4bce-afa6-3c6a4b89d8ea)

   Bob has successfully received the file
   
    ![image](https://github.com/user-attachments/assets/ec47579e-e725-44ef-8839-ea7982015bf7)

---

### Steps on Bob's Side

1. **Verify the HMAC**: Bob verifies the authenticity of the file by checking the HMAC:
    ```bash
    openssl dgst -sha256 -hmac "11223344551122334455112233445511" -binary file.enc > file.enc.hmac.verify
    diff file.enc.hmac file.enc.hmac.verify
    ```
    This set of commands does the following:

    1. **`openssl dgst -sha256 -hmac "11223344551122334455112233445511" -binary file.enc > file.enc.hmac.verify`**:
       - **`openssl dgst`**: Invokes the `openssl dgst` command to compute a message digest.
       - **`-sha256`**: Specifies the SHA-256 hashing algorithm.
       - **`-hmac "11223344551122334455112233445511"`**: Uses the given key (`11223344551122334455112233445511`) to compute the HMAC            for the file.
       - **`-binary`**: Outputs the hash in binary format.
       - **`file.enc`**: The input file (`file.enc`) whose HMAC hash is being computed.
       - **`> file.enc.hmac.verify`**: Redirects the resulting hash to the file `file.enc.hmac.verify`.
    
    2. **`diff file.enc.hmac file.enc.hmac.verify`**:
       - **`diff`**: Compares two files line by line.
       - **`file.enc.hmac`**: The original HMAC file that was transferred or created earlier.
       - **`file.enc.hmac.verify`**: The newly computed HMAC file.
       - This command compares the two files and shows any differences. If the files match, there will be no output, indicating that         the HMACs are identical.

   > **Conclusion**: This is used to compare the HMACs to confirm authenticity and integrity. If diff show no difference, that means the authenticity        and integrity has been confirmed that it is truely from Alice
   
    ![image](https://github.com/user-attachments/assets/ac1e6213-7a04-4fa7-a57f-e63e63ee6692)

2. **Decrypt the File**: Finally, Bob decrypts the file using the shared secret key:
    ```bash
    openssl enc -aes-256-ecb -d -in file.enc -out file_decrypted.txt -pass pass:11223344551122334455112233445511
    cat file_decrypted.txt
    ```
    
    This sequence of commands does the following:

    1. **`openssl enc -aes-256-ecb -d -in file.enc -out file_decrypted.txt -pass pass:11223344551122334455112233445511`**:
       - **`openssl enc`**: Invokes OpenSSL's encryption functionality to perform encryption or decryption.
       - **`-aes-256-ecb`**: Specifies AES encryption with a 256-bit key in ECB (Electronic Codebook) mode.
       - **`-d`**: Tells OpenSSL to perform decryption (as opposed to encryption).
       - **`-in file.enc`**: Specifies the encrypted input file (`file.enc`) to decrypt.
       - **`-out file_decrypted.txt`**: Specifies the output file (`file_decrypted.txt`) where the decrypted data will be stored.
       - **`-pass pass:11223344551122334455112233445511`**: Provides the secret key for decryption using the passphrase `11223344551122334455112233445511`.
    
    2. **`cat file_decrypted.txt`**:
       - **`cat`**: Displays the contents of the specified file (`file_decrypted.txt`).
     
    > **Conclusion**: This will display the original content of the file.
    ![File Decryption](https://github.com/user-attachments/assets/c135d392-ffa8-42c9-b906-26b66a294601)

---

# Task 2: Transfering encrypted file and decrypt it with hybrid encryption. 
**Question 1**:
Conduct transfering a file (deliberately choosen by you) between 2 computers. 
The file is symmetrically encrypted/decrypted by exchanging secret key which is encrypted using RSA. 
All steps are made manually with openssl at the terminal of each computer.

**Answer 1**:
---

### Step 1: Generate RSA Key Pairs


1. **On Bob's Machine**: Generate a public-private RSA key pair:
    ```bash
    openssl genrsa -out keypair.pem 2048
    ```
   
This command does the following:

- **`openssl genrsa`**: Generates an RSA private key.
- **`-out keypair.pem`**: Specifies the output file (`keypair.pem`) where the generated private key will be saved.
- **`2048`**: Specifies the size of the RSA key in bits (2048 bits).
  
    ![RSA Key Pair Generation](https://github.com/user-attachments/assets/6d21fe9b-91ed-4209-b40b-80d23b3ba203)
    ![image](https://github.com/user-attachments/assets/c936ac26-cb90-4331-8e5d-f9b45d455803)


2. **Extract Bob's Public Key**: Extract the public key from the generated private key:
    ```bash
    openssl rsa -in keypair.pem -pubout -out publickey.crt
    ```
    This command does the following:

    - **`openssl rsa`**: Invokes OpenSSL's RSA utility to process RSA private and public keys.
    - **`-in keypair.pem`**: Specifies the input file (`keypair.pem`) that contains the RSA private key.
    - **`-pubout`**: Tells OpenSSL to output the public key corresponding to the private key.
    - **`-out publickey.crt`**: Specifies the output file (`publickey.crt`) where the public key will be saved.
      
    ![Public Key Extraction](https://github.com/user-attachments/assets/63b2d604-15f6-448a-b8d5-9eacd23e717d)
   
   And then we will use command
   
   ```
   cat publickey.crt
   ```

   To see the public key
   
   ![image](https://github.com/user-attachments/assets/d93259b1-6c5a-40d5-86a8-bb81b032aa0d)

4. **Send Public Key to Alice**: Transfer the public key file to Alice via SCP:
    ```bash
    scp publickey.crt alice@10.9.0.5:/home
    ```
    This command does the following:

    - **`scp`**: Invokes the secure copy protocol to transfer files between a local machine and a remote machine over SSH.
    - **`publickey.crt`**: Specifies the file (`publickey.crt`) to be transferred.
    - **`alice@10.9.0.5:/home`**: Specifies the remote server's username (`alice`), IP address (`10.9.0.5`), and the destination directory (`/home/`) where the file     will be copied.
      
    ![Send Public Key](https://github.com/user-attachments/assets/a6538150-b222-4475-8184-8eb8fc078e44)

---

### Step 2: File Encryption and Transfer from Alice

1. **Generate a Random Password**: Alice generates a random 256-bit password to use for symmetric encryption:
    ```bash
    openssl rand -hex 32 > randompassword
    ```
    This command does the following:

    - **`openssl rand`**: Invokes OpenSSL's random number generation utility.
    - **`-hex`**: Specifies that the output should be in hexadecimal format.
    - **`32`**: Specifies the number of random bytes to generate (32 bytes = 256 bits).
    - **`> randompassword`**: Redirects the generated random hex string to a file named `randompassword`.

    ![Random Password Generation](https://github.com/user-attachments/assets/dcc202ff-f783-4790-96cd-2f294d0358ba)

2. **Encrypt the File Using AES**: Alice encrypts the file using the random password:
    ```bash
    openssl enc -aes-256-ecb -in file.txt -out file.enc -pass file:/home/randompassword
    ```
    This command does the following:

    - **`openssl enc`**: Invokes OpenSSL's encryption functionality to perform encryption or decryption.
    - **`-aes-256-ecb`**: Specifies AES encryption with a 256-bit key in ECB (Electronic Codebook) mode.
    - **`-in file.txt`**: Specifies the input file (`file.txt`) to encrypt.
    - **`-out file.enc`**: Specifies the output file (`file.enc`) where the encrypted data will be stored.
    - **`-pass file:/home/randompassword`**: Specifies the passphrase for encryption by reading it from the file `/home/randompassword`.
      
    ![File Encryption](https://github.com/user-attachments/assets/5e0eee40-13ba-4b11-9f3d-1f2e9469563d)

3. **Encrypt the Random Password Using RSA**: Alice encrypts the random password using Bob's public key:
    ```bash
    openssl rsautl -encrypt -inkey publickey.crt -pubin -in randompassword -out randompassword.encrypted
    ```
   This command does the following:
    
    - **`openssl rsautl`**: Invokes OpenSSL's utility for RSA encryption and decryption.
    - **`-encrypt`**: Specifies that the operation is encryption.
    - **`-inkey publickey.crt`**: Specifies the input file (`publickey.crt`) containing the RSA public key to be used for encryption.
    - **`-pubin`**: Indicates that the input key is a public key.
    - **`-in randompassword`**: Specifies the input file (`randompassword`) that contains the data to be encrypted (in this case, the randomly generated password).
    - **`-out randompassword.encrypted`**: Specifies the output file (`randompassword.encrypted`) where the encrypted data will be stored.
  
    ![Encrypt Random Password](https://github.com/user-attachments/assets/91e32951-ad2c-4ebc-8acb-37f705bc407d)

  
5. **Transfer Files to Bob**: Alice sends the encrypted file and encrypted password to Bob:
    ```bash
    scp file.enc randompassword.encrypted bob@10.9.0.6:/home
    ```
    This command does the following:

    - **`scp`**: Invokes the secure copy protocol to transfer files between a local machine and a remote machine over SSH.
    - **`file.enc randompassword.encrypted`**: Specifies the files (`file.enc` and `randompassword.encrypted`) to be transferred.
    - **`bob@10.9.0.6:/home`**: Specifies the remote server's username (`bob`), IP address (`10.9.0.6`), and the destination directory (`/home/`) where the files  will be copied.
  
    ![image](https://github.com/user-attachments/assets/a40834f3-f5db-4ac1-86e6-1f4648b20b5c)

    ![image](https://github.com/user-attachments/assets/831b8495-5c7c-4ba2-ae11-7277775f0ab3)

---

### Step 3: Decrypt the Files on Bob's Machine

1. **Decrypt the Random Password**: Bob decrypts the random password using his private key:
    ```bash
    openssl rsautl -decrypt -inkey keypair.pem -in randompassword.encrypted -out randompassword.decrypted
    ```
    This command does the following:

    - **`openssl rsautl`**: Invokes OpenSSL's utility for RSA encryption and decryption.
    - **`-decrypt`**: Specifies that the operation is decryption.
    - **`-inkey keypair.pem`**: Specifies the input file (`keypair.pem`) containing the RSA private key to be used for decryption.
    - **`-in randompassword.encrypted`**: Specifies the input file (`randompassword.encrypted`) that contains the encrypted data.
    - **`-out randompassword.decrypted`**: Specifies the output file (`randompassword.decrypted`) where the decrypted data will be saved.

   ![image](https://github.com/user-attachments/assets/5809f206-1492-4dae-876f-f91ff4a15961)

2. **Decrypt the File Using the Decrypted Password**: Bob decrypts the file using the decrypted password:
    ```bash
    openssl enc -aes-256-ecb -d -in file.enc -out file_decrypted.txt -pass file:/home/randompassword.decrypted
    cat file_decrypted.txt
    ```
    This sequence of commands does the following:

    1. **`openssl enc -aes-256-ecb -d -in file.enc -out file_decrypted.txt -pass file:/home/randompassword.decrypted`**:
       - **`openssl enc`**: Invokes OpenSSL's encryption functionality to perform encryption or decryption.
       - **`-aes-256-ecb`**: Specifies AES encryption with a 256-bit key in ECB (Electronic Codebook) mode.
       - **`-d`**: Tells OpenSSL to perform decryption (as opposed to encryption).
       - **`-in file.enc`**: Specifies the encrypted input file (`file.enc`) to decrypt.
       - **`-out file_decrypted.txt`**: Specifies the output file (`file_decrypted.txt`) where the decrypted data will be stored.
       - **`-pass file:/home/randompassword.decrypted`**: Specifies the passphrase for decryption by reading it from the file `/home/randompassword.decrypted`.
    
    2. **`cat file_decrypted.txt`**:
       - **`cat`**: Displays the contents of the specified file (`file_decrypted.txt`).
         
   ![image](https://github.com/user-attachments/assets/f9425a76-3043-4b45-aa2e-0a1df522f65d)

# Task 3: Firewall configuration
**Question 1**:
From VMs of previous tasks, install iptables and configure one of the 2 VMs as a web and ssh server. Demonstrate your ability to block/unblock http, icmp, ssh requests from the other host.

**Answer 1**:

In in task, i will setup Bob as a a web and ssh server. First, we will need to go into Bob container docker with command 

```
docker exec --privileged -it bob-10.9.0.6 /bin/bash
```

This command does the following:

- **`docker exec`**: Used to run commands inside an already running container.
- **`--privileged`**: Grants extended privileges to the container (if needed for specific operations).
- **`-it`**: Stands for interactive terminal, allowing you to interact with the container’s shell.
- **`bob-10.9.0.6`**: Refers to the container's name or ID, identifying which container the command should target.
- **`/bin/bash`**: Specifies the command to run inside the container, opening a bash shell.


![image](https://github.com/user-attachments/assets/de226d85-8459-4a3c-acf2-7683ac5cbccc)


I will install iptables on Bob machine using command

```
apt install iptables
```

![image](https://github.com/user-attachments/assets/57da898c-a308-489d-a143-be6488413728)


Then, i will use command to list all rules of iptables

```
sudo iptables -L
```

![image](https://github.com/user-attachments/assets/ce91ad00-a3c0-4b9f-9ddd-bc62593843c8)

Then, i will install openssh-server on Bob Machine

```
apt install openssh-server
```

![image](https://github.com/user-attachments/assets/60a35484-5389-45a7-954e-b8747319f2ce)


Then, i will use 2 command to start and check status of ssh server

```
service ssh start
service ssh status
```

![image](https://github.com/user-attachments/assets/d132f91e-a589-424e-86cc-bc43fe779b1c)


Then, i will install apache2 server on Bob machine

```
apt install apache2
```

![image](https://github.com/user-attachments/assets/490ae6d2-2ae7-44be-a177-402412c2d105)

Then, i will use 2 command to start and check status of apache2 server

```
service apache2 start
service apache2 status
```

![image](https://github.com/user-attachments/assets/f9f0b96f-4838-4370-a8e1-1179c2456c31)


As we know, the IP of Alice will be `10.9.0.5`

The IP of Bob will be `10.9.0.6`


### Block http, icmp, ssh requests from the other host.

#### Block http using command

```
sudo iptables -A INPUT -p tcp --dport 80 -s 10.9.0.5 -j DROP
```

This command does the following:

- **`iptables`**: Invokes the `iptables` utility to manage firewall rules on a Linux-based system.
- **`-A INPUT`**: Appends a rule to the `INPUT` chain, which controls incoming network traffic.
- **`-p tcp`**: Specifies the protocol as TCP (Transmission Control Protocol).
- **`--dport 80`**: Specifies that the rule applies to traffic destined for port 80, which is commonly used for HTTP.
- **`-s 10.9.0.5`**: Specifies that the rule applies to traffic originating from the IP address `10.9.0.5`.
- **`-j DROP`**: Specifies the action to take, which is to drop the packet, effectively blocking the connection.
  
![image](https://github.com/user-attachments/assets/3a7efc6a-0ce0-43b5-94de-db97f38f2059)

and then next, we can try to access http from Alice using command

```
curl http://10.9.0.6
```

This command does the following:

- **`curl`**: Uses the `curl` command to transfer data from or to a server, supporting various protocols (in this case, HTTP).
- **`http://10.9.0.6`**: Specifies the URL (in this case, the IP address `10.9.0.6`) of the server to which the request will be sent.
  - The default HTTP method used by `curl` is `GET`, so this will send a request to retrieve the content from the server at `10.9.0.6`.


![image](https://github.com/user-attachments/assets/49d2d8b8-a2b9-4c83-ab93-ca9f9eb10de3)

> **Conclusion**: As we can see on the picture, there is no html sent back to Alice

#### Block both incoming and outoging icmp using command

```
sudo iptables -A INPUT -p icmp --icmp-type echo-request -s 10.9.0.5 -j DROP
sudo iptables -A OUTPUT -p icmp --icmp-type echo-request -d 10.9.0.5 -j DROP
```
These commands do the following:

1. **`sudo iptables -A INPUT -p icmp --icmp-type echo-request -s 10.9.0.5 -j DROP`**:
   - **`sudo`**: Executes the command with superuser privileges, which are required for modifying firewall rules.
   - **`iptables`**: Invokes the `iptables` utility to manage firewall rules on a Linux-based system.
   - **`-A INPUT`**: Appends a rule to the `INPUT` chain, which controls incoming network traffic.
   - **`-p icmp`**: Specifies the protocol as ICMP (Internet Control Message Protocol), which is used for ping requests and replies.
   - **`--icmp-type echo-request`**: Specifies that the rule applies to ICMP echo requests (ping requests).
   - **`-s 10.9.0.5`**: Specifies that the rule applies to traffic originating from the IP address `10.9.0.5`.
   - **`-j DROP`**: Specifies the action to take, which is to drop the packet, effectively blocking the connection.

2. **`sudo iptables -A OUTPUT -p icmp --icmp-type echo-request -d 10.9.0.5 -j DROP`**:
   - **`sudo`**: Executes the command with superuser privileges.
   - **`iptables`**: Invokes the `iptables` utility.
   - **`-A OUTPUT`**: Appends a rule to the `OUTPUT` chain, which controls outgoing network traffic.
   - **`-p icmp`**: Specifies the protocol as ICMP.
   - **`--icmp-type echo-request`**: Specifies that the rule applies to ICMP echo requests (ping requests).
   - **`-d 10.9.0.5`**: Specifies that the rule applies to traffic destined for the IP address `10.9.0.5`.
   - **`-j DROP`**: Specifies the action to take, which is to drop the packet, effectively blocking the outgoing ping request.

![image](https://github.com/user-attachments/assets/cad37f10-13ad-4eb4-93ab-266b70f8c2ce)

and then next, we can try to ping from Alice using command

```
ping 10.0.9.6
```
This command does the following:

- **`ping`**: Sends ICMP echo requests to a specified network host to check its availability and measure round-trip time.
- **`10.0.9.6`**: Specifies the IP address of the host you want to ping. The command will send a series of packets to `10.0.9.6` and wait for a reply.

![image](https://github.com/user-attachments/assets/421dc35c-2246-473f-a70f-7ef875fdea78)

> **Conclusion**: As we can see, they cannot comunate each other through icmp because after the first packet sent, there is no reply


#### Block ssh request

We can block all SSH requests from other hosts using command

```
sudo iptables -A INPUT -p tcp --dport 22 -s 10.9.0.5 -j DROP
```
This command does the following:

- **`sudo`**: Executes the command with superuser privileges, which are required for modifying firewall rules.
- **`iptables`**: Invokes the `iptables` utility to manage firewall rules on a Linux-based system.
- **`-A INPUT`**: Appends a rule to the `INPUT` chain, which controls incoming network traffic.
- **`-p tcp`**: Specifies the protocol as TCP (Transmission Control Protocol).
- **`--dport 22`**: Specifies that the rule applies to traffic destined for port 22, which is commonly used for SSH (Secure Shell) connections.
- **`-s 10.9.0.5`**: Specifies that the rule applies to traffic originating from the IP address `10.9.0.5`.
- **`-j DROP`**: Specifies the action to take, which is to drop the packet, effectively blocking the connection.

![image](https://github.com/user-attachments/assets/a4b187bc-792f-41ef-be4f-7197d2b34cb8)

On Bob, we can confirm that openssh-server is still running using command

```
service ssh status
```

![image](https://github.com/user-attachments/assets/5a6bff67-4a46-4f86-bb79-d40cee747a00)

On Alice, we can confirm that openssh-server is still running using command 

```
service ssh status
```

![image](https://github.com/user-attachments/assets/0095f9fe-e3fc-4d3a-b0f1-dbea3dc689ef)

Try to send file.txt from Alice using command
```
scp file.txt bob@10.9.0.6:/home
```

This command does the following:

- **`scp`**: Invokes the secure copy protocol to transfer files between a local machine and a remote machine over SSH.
- **`test.txt`**: Specifies the file (`test.txt`) to be transferred.
- **`bob@10.9.0.6:/home`**: Specifies the remote server's username (`bob`), IP address (`10.9.0.6`), and the destination directory (`/home/`) where the file will be copied.

![image](https://github.com/user-attachments/assets/8b825df5-a616-418b-831d-11ee92dfefa8)

> **Conclusion**: The picture show that we cant connect to Bob's ssh channel

### Unblock http, icmp, ssh requests from the other host.

#### Unblock HTTP:

Number-list iptables
![image](https://github.com/user-attachments/assets/c7238da1-a5dd-444c-a9a9-6822ae7abb94)

On Bob machine, we can unblock http request using command
```
sudo iptables -D INPUT 1
```
This command does the following:

- **`sudo`**: Executes the command with superuser privileges, which are required for modifying firewall rules.
- **`iptables`**: Invokes the `iptables` utility to manage firewall rules on a Linux-based system.
- **`-D INPUT`**: Deletes a rule from the `INPUT` chain, which controls incoming network traffic.
- **`1`**: Specifies the rule number to delete from the `INPUT` chain. In this case, `1` refers to the first rule in the list.

  ![image](https://github.com/user-attachments/assets/4f5eddba-3362-48aa-aae5-c1d4b2f32c42)


On Alice using command to access http:
```
curl http://10.9.0.6
```

This command does the following:

- **`curl`**: Uses the `curl` command to transfer data from or to a server, supporting various protocols (in this case, HTTP).
- **`http://10.9.0.6`**: Specifies the URL (in this case, the IP address `10.9.0.6`) of the server to which the request will be sent.
  - The default HTTP method used by `curl` is `GET`, so this will send a request to retrieve the content from the server at `10.9.0.6`.


![image](https://github.com/user-attachments/assets/88f8be16-db3f-4b93-ac45-bce2ce4e9a87)

> **Conclusion**: We can see on the picture that there is html sent back to Alice

#### Unblock ICMP:

![image](https://github.com/user-attachments/assets/b80e2e81-757e-437b-aa13-6d15b5ddc878)

On Bob, we can unblock icmp request using command
```
sudo iptables -D INPUT 1
sudo iptables -D OUTPUT 1
```
These commands do the following:

1. **`sudo iptables -D INPUT 1`**:
   - **`sudo`**: Executes the command with superuser privileges, which are required for modifying firewall rules.
   - **`iptables`**: Invokes the `iptables` utility to manage firewall rules on a Linux-based system.
   - **`-D INPUT`**: Deletes a rule from the `INPUT` chain, which controls incoming network traffic.
   - **`1`**: Specifies the rule number to delete from the `INPUT` chain. In this case, `1` refers to the first rule in the list.

2. **`sudo iptables -D OUTPUT 1`**:
   - **`sudo`**: Executes the command with superuser privileges.
   - **`iptables`**: Invokes the `iptables` utility.
   - **`-D OUTPUT`**: Deletes a rule from the `OUTPUT` chain, which controls outgoing network traffic.
   - **`1`**: Specifies the rule number to delete from the `OUTPUT` chain. In this case, `1` refers to the first rule in the list.

the number based on the picture number-list iptables

![image](https://github.com/user-attachments/assets/24c1e893-49aa-47c3-beef-55fb87363836)

On Alice, we can try to ping to Bob using command
```
ping 10.9.0.6
```

This command does the following:

- **`ping`**: Sends ICMP echo requests to a specified network host to check its availability and measure round-trip time.
- **`10.0.9.6`**: Specifies the IP address of the host you want to ping. The command will send a series of packets to `10.0.9.6` and wait for a reply.

![image](https://github.com/user-attachments/assets/6c25964f-b2b3-416b-95a3-6fb3d11a8731)


> **Conclusion**: We can see on the picture that, there is request and reply between 2 hosts

#### Unblock SSH

![image](https://github.com/user-attachments/assets/0e031d02-ccbe-4a58-8b37-ac815a924906)

On Bob, we can unblock ssh using command 
```
sudo iptables -D INPUT 1
```
This command does the following:

- **`sudo`**: Executes the command with superuser privileges, which are required for modifying firewall rules.
- **`iptables`**: Invokes the `iptables` utility to manage firewall rules on a Linux-based system.
- **`-D INPUT`**: Deletes a rule from the `INPUT` chain, which controls incoming network traffic.
- **`1`**: Specifies the rule number to delete from the `INPUT` chain. In this case, `1` refers to the first rule in the list.

![image](https://github.com/user-attachments/assets/37131936-28d4-4486-bc2f-6442e3d3a8d5)


On Alice, we can try to send test.txt file to Bob using command
```
scp file.txt bob@10.9.0.6:/home
```

This command does the following:

- **`scp`**: Invokes the secure copy protocol to transfer files between a local machine and a remote machine over SSH.
- **`test.txt`**: Specifies the file (`test.txt`) to be transferred.
- **`bob@10.9.0.6:/home`**: Specifies the remote server's username (`bob`), IP address (`10.9.0.6`), and the destination directory (`/home/`) where the file will be copied.
  
![image](https://github.com/user-attachments/assets/cb647c05-96b0-4c92-88ae-0adb7a96d6c8)

> **Conclusion**: As the image shown, we can send file in SSH channel






