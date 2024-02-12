# Mina Node Requirements

## Software:

- **Operating Systems:** macOS, Linux (Debian 9 and Ubuntu 18.04 LTS), or any host with Docker.
  - Windows is not officially supported, but it can be set up using Windows Subsystem for Linux or Docker (community-created instructions available).

## Hardware:

- **CPU:** At least an 8-core processor.
- **RAM:** At least 16GB (more may be needed if running a snark worker node simultaneously).
- **GPUs:** Not currently required but may be needed in future protocol upgrades for node operators.

## Network:

- Minimum 1 Mbps connection.

## VM Instances (Recommended for Basic Node Operator Needs):

- **AWS:** c5.2xlarge.
- **GCP:** c2-standard-8.
- **Azure:** Standard_F8s_v2.
- **Digital Ocean:** c-8-16gib.
  
  Note: Custom requirements or different cost constraints may necessitate choosing a different instance type. For the latest updates, kindly check out Mina Protocol documentation.

# How to Deploy a Mina Node

## Step 1: Install Mina

1. Add the Mina Debian repo and install (for Ubuntu 18.04 / Debian 9):

```bash
echo "deb [trusted=yes] http://packages.o1test.net release main" | sudo tee /etc/apt/sources.list.d/mina.list 
sudo apt-get update 
sudo apt-get install -y curl unzip mina-mainnet=1.1.5-a42bdee
```

2. Check the installation by running `mina version`. The expected output is Commit a42bdeef6b0c15ee34616e4df76c882b0c5c7c2a on branch master.

3. If you're on another Linux distro or macOS version, you can try building Mina from source code. Note that other operating systems might have issues. Seek troubleshooting help on Discord.

## Step 2: Create a keypair

1. Generate a keypair.

- First, ensure you have a dedicated folder on your system to store the key files. We recommend using the `~/keys` folder.

```bash
mkdir ~/keys
```

- Next, set the proper permissions for this folder to prevent unauthorized access:

```bash
chmod 700 ~/keys
```

- Generate a key using one of the following methods based on your setup:

On Ubuntu/Debian:

```bash
mina-generate-keypair --privkey-path ~/keys/my-wallet
```

On Docker (Windows/MacOS/Linux):

```bash
docker run --interactive --tty --rm --volume $(pwd)/keys:/keys minaprotocol/mina-generate-keypair:1.3.0-9b0369c --privkey-path /keys/my-wallet
```

2. Ensure Private Key File Permissions:

```bash
chmod 600 $(pwd)/keys/my-wallet
```

3. Validate Your Private Key:

On Linux:

```bash
mina-validate-keypair --privkey-path <path-to-the-private-key-file>
```

Using Docker:

```bash
docker run --interactive --tty --rm --entrypoint=mina-validate-keypair --volume $(pwd)/keys:/keys minaprotocol/mina-generate-keypair:1.3.0-9b0369c --privkey-path /keys/my-wallet
```

## Step 3: Start your node

1. Update your Mina daemon.

- To update your Mina daemon and connect to Mainnet, follow these steps based on your operating system:

For Ubuntu 18.04, 20.04, Debian 9, 10, 11:

```bash
echo "deb [trusted=yes] http://packages.o1test.net $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/mina.list 
sudo apt-get install --yes apt-transport-https 
sudo apt-get update 
sudo apt-get install --yes curl unzip mina-mainnet=1.3.1.2-25388a0
```

2. Verify that the daemon is installed correctly: `mina version`

3. Start a standalone mina node.

- To start a Mina node instance and connect to the live network:

```bash
mina daemon --peer-list-url https://storage.googleapis.com/seed-lists/mainnet_seeds.txt
```

- If you have a key with MINA stake and wish to produce blocks:

```bash
mina daemon --peer-list-url https://storage.googleapis.com/seed-lists/mainnet_seeds.txt \
--block-producer-key ~/keys/my-wallet
```

Here, `~/keys/my-wallet` is the path to your private key.

4. Stop the standalone node.

- Pause the existing mina daemon process by pressing Ctrl+C.

5. Start a mina node with auto-restart flows.

- After confirming functionality with the standalone process, set up mina with auto-restart workflows for continued operation after logout and automatic restart on machine reboot.

6. Create and customize the `.mina-env`.

- For block production or configuration customization:

- Create the `~/.mina-env` file.

- For block production, add configuration (replace `<BLOCK_PRODUCER_KEY_PATH>`):

```bash
MINA_PRIVKEY_PASS="My_V3ry_S3cure_Password" 
LOG_LEVEL=Info 
FILE_LOG_LEVEL=Debug 
EXTRA_FLAGS=" --block-producer-key "
```

- If not producing blocks, keep `~/.mina-env` empty.

- To change mina's configuration, specify flags with space-separated arguments in `EXTRA_FLAGS`.

7. Start a Mina node instance and connect.

- Run the command:

```bash
systemctl --user daemon-reload 
systemctl --user start mina 
systemctl --user enable mina 
sudo loginctl enable-linger
```

This allows the node to run after logout and restart automatically on machine reboots.

8. Check connectivity.

- Monitor the running mina process and auto-restarts.

- Check if Mina had trouble starting:

```bash
systemctl --user status mina
```

- Stop Mina and disable automatic restart:

```bash
systemctl --user stop mina
```

- Manually restart Mina:

```bash
systemctl --user restart mina
```

- View logs:

```bash
journalctl --user -u mina -n 1000 -f
```

- In some cases:

```bash
journalctl --user-unit mina -n 1000 -f
```

9. Monitor mina client status.

- In a different terminal:

```bash
mina client status
```

- Upon initial startup, it may take time for mina client status to connect. If you see "Error: daemon not running," wait and try again.

- The expected response includes:

```
...Peers: Total: 4 (...) 
...Sync Status: Bootstrap
```

- `Bootstrap` indicates syncing, which can take time. Later, it moves to `Sync Status: Catchup`, gathering recent blocks. Upon reaching `Synced`, it's successfully connected.

10. Troubleshoot port issues causing prolonged Bootstrap state.

11. With a synced node, explore advanced daemon features like Sending a payment, Staking & Delegating.
