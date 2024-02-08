
# Mina Node Requirements

## Software:
- **Operating Systems:** macOS, Linux (Debian 9 and Ubuntu 18.04 LTS), or any host with Docker.
- *Windows* is not officially supported, but it can be set up using Windows Subsystem for Linux or Docker (community-created instructions available).

## Hardware:
- **CPU:** At least an 8-core processor.
- **RAM:** At least 16GB (more may be needed if running a snark worker node simultaneously).
- *GPUs* are not currently required but may be needed in future protocol upgrades for node operators.

## Network:
- Minimum 1 Mbps connection.

## VM Instances (Recommended for Basic Node Operator Needs):
- **AWS:** c5.2xlarge.
- **GCP:** c2-standard-8.
- **Azure:** Standard_F8s_v2.
- **Digital Ocean:** c-8-16gib.
    - Note: Custom requirements or different cost constraints may necessitate choosing a different instance type. For the latest updates, kindly check out Mina Protocol documentation.

# How to Deploy a Mina Node

## Step 1: Install Mina
- Add the Mina Debian repo and install (for Ubuntu 18.04 / Debian 9):
    ```
    echo "deb [trusted=yes] http://packages.o1test.net release main" | sudo tee /etc/apt/sources.list.d/mina.list sudo apt-get update sudo apt-get install -y curl unzip mina-mainnet=1.1.5-a42bdee
    ```
- Check the installation by running `mina version`. The expected output isCommit a42bdeef6b0c15ee34616e4df76c882b0c5c7c2a on branch master.
- If you're on another Linux distro or macOS version, you can try building Mina from source code. Note that other operating systems might have issues. Seek troubleshooting help on Discord

## Step 2: Create a keypair
### Generate a keypair
- Generate a keypair
- First, ensure you have a dedicated folder on your system to store the key files. We recommend using the `~/keys` folder.
    ```
    mkdir ~/keys
    ```
- Next, set the proper permissions for this folder to prevent unauthorized access:
    ```
    chmod 700 ~/keys
    ```
- Generate a key using one of the following methods based on your setup:
    - On Ubuntu/Debian: Use the `mina-generate-keypair` command.
        ```
        mina-generate-keypair --privkey-path ~/keys/my-wallet
        ```
    - On Docker (Windows/MacOS/Linux): Use the `minaprotocol/generate-keypair` Docker image.
        ```
        ~docker run --interactive --tty --rm --volume $(pwd)/keys:/keys minaprotocol/mina-generate-keypair:1.3.0-9b0369c --privkey-path /keys/my-wallet
        ```
    - This creates two files: `~/keys/my-wallet` (encrypted private key) and `~/keys/my-wallet.pub` (public key in plain text). Store the private key file and its password securely, such as in a password manager.
- Ensure Private Key File Permissions
    - For added security, set the proper permissions for the private key file:
        ```
        chmod 600 $(pwd)/keys/my-wallet
        ```
- Validate Your Private Key
    - Ensure your private key works by validating it. Use the `mina-validate-keypair` tool for this:
        - On Linux:
            ```
            mina-validate-keypair --privkey-path <path-to-the-private-key-file>
            ```
        - Using Docker:
            ```
            docker run --interactive --tty --rm --entrypoint=mina-validate-keypair --volume $(pwd)/keys:/keys minaprotocol/mina-generate-keypair:1.3.0-9b0369c --privkey-path /keys/my-wallet
            ```

## Step 3: Start your node
- **Update your Mina daemon**
    - To update your Mina daemon and connect to Mainnet, follow these steps based on your operating system:
        - For Ubuntu 18.04, 20.04, Debian 9, 10, 11:
            - Install the latest Stable Mina Release 1.3.1.2, or opt for pre-release (Beta) builds from GitHub Releases Page.
                ```
                echo "deb [trusted=yes] http://packages.o1test.net $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/mina.list sudo apt-get install --yes apt-transport-https sudo apt-get update sudo apt-get install --yes curl unzip mina-mainnet=1.3.1.2-25388a0
                ```
            - Verify that the daemon is installed correctly: `mina version`
                - The expected output is:
                    ```
                    Commit 25388a0fed9695e8e9d04f75f50c2bae1c9c80db on branch master
                    ```
- **Start a standalone mina node**
    - Note: If using Hetzner hosting, refer to Networking troubleshooting guidance before starting a node.
    - To start a Mina node instance and connect to the live network:
        ```
        mina daemon --peer-list-url https://storage.googleapis.com/seed-lists/mainnet_seeds.txt
        ```
    - Mina uses peer-to-peer protocol, connecting to seed peers for network initiation.
    - If you have a key with MINA stake and wish to produce blocks:
        ```
        mina daemon --peer-list-url https://storage.googleapis.com/seed-lists/mainnet_seeds.txt \ --block-producer-key ~/keys/my-wallet
        ```
    - Here, `~/keys/my-wallet` is the path to your private key.

- **Stop the standalone node**
    - Pause the existing mina daemon process by pressing `Ctrl+C`.
- **Start a mina node with auto-restart flows**
    - After confirming functionality with the standalone process, set up mina with auto-restart workflows for continued operation after logout and automatic restart on machine reboot.
    - **Create and customize the .mina-env**
        - For block production or configuration customization:
            - Create the `~/.mina-env` file.
            - For block production, add configuration (replace `<BLOCK_PRODUCER_KEY_PATH>`):
                ```
                MINA_PRIVKEY_PASS="My_V3ry_S3cure_Password" LOG_LEVEL=Info FILE_LOG_LEVEL=Debug EXTRA_FLAGS=" --block-producer-key "
                ```
            - If not producing blocks, keep `~/.mina-env` empty.
            - To change mina's configuration, specify flags with space-separated arguments in `EXTRA_FLAGS`.
    - **Start a Mina node instance and connect**
        - Run the command:
            ```
            systemctl --user daemon-reload systemctl --user start mina systemctl --user enable mina sudo loginctl enable-linger
            ```
        - This allows the node to run after logout and restart automatically on machine reboots.
# Check Connectivity and Monitoring

To ensure connectivity and monitor the Mina process, follow these steps:

## Check Connectivity

- **Monitor the Running Mina Process and Auto-Restarts**
    - To monitor the running Mina process and auto-restarts:
        ```
        systemctl --user status mina
        ```

- **Troubleshoot Mina Startup Issues**
    - If Mina encounters startup issues, you can troubleshoot by checking its status:
        ```
        systemctl --user status mina
        ```

## Management Commands

- **Stop Mina and Disable Automatic Restart**
    - To stop Mina and disable automatic restart:
        ```
        systemctl --user stop mina
        ```

- **Manually Restart Mina**
    - To manually restart Mina:
        ```
        systemctl --user restart mina
        ```

## View Logs

- **View Mina Logs**
    - To view Mina logs for debugging purposes:
        ```
        journalctl --user -u mina -n 1000 -f
        ```
    - In some cases, you might need to specify the unit:
        ```
        journalctl --user-unit mina -n 1000 -f
        ```

## Monitor Mina Client Status

- **Check Mina Client Status**
    - Open a different terminal and check the Mina client status:
        ```
        mina client status
        ```

    - Upon initial startup, it may take time for the Mina client status to connect. If you encounter "Error: daemon not running," wait for some time and try again.

    - The expected response includes details about peers and sync status, such as:
        ```
        Peers: Total: 4 (...) Sync Status: Bootstrap
        ```

    - Bootstrap indicates syncing, which can take time. Later, it moves to Sync Status: Catchup, gathering recent blocks. Upon reaching Synced, it indicates a successful connection.

## Troubleshooting

- **Troubleshoot Port Issues**
    - If the Bootstrap state persists for an extended period, troubleshoot port issues that might be causing delays.

## Advanced Daemon Features

- **Explore Advanced Daemon Features**
    - Once you have a synced node, explore advanced daemon features such as sending a payment, staking & delegating. Refer to the guide on staking MINA with validators to earn MINA tokens.
