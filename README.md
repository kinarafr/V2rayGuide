# The Complete Guide to Setting Up a V2Ray Server (VLESS + 3x-ui)
 
Setting up your own V2Ray server gives you total control over your proxy network. This guide will walk you through deploying a VPS, securing a domain, generating SSL certificates, and configuring the 3x-ui panel for a VLESS connection.
 
---
 
## Step 1: Choose a Cloud Provider & Deploy a VPS
 
First, you need a Virtual Private Server (VPS) to host your V2Ray setup.
 
- **Providers:** You can choose any major cloud service provider like DigitalOcean, AWS, Oracle, or smaller independent hosts depending on your budget and performance needs.
- **Recommendation:** **SpeedyPage** is a great choice — they offer affordable, high-performance servers that typically include 2TB of monthly bandwidth, which is plenty for personal use.
- **OS & Settings:** When creating your VPS, select **Ubuntu LTS** (e.g., 20.04 or 22.04) as your operating system. **Crucial:** Make sure you **do not enable IPv6** during setup, as it can complicate routing later.
---
 
## Step 2: Connect to Your Server via SSH (Using Termius)
 
Once your VPS is running, you need to access its command line. **Termius** is an excellent, user-friendly SSH client for Windows, Mac, iOS, and Android.
 
**How to connect using Termius:**
 
1. **Download and Open Termius.** Go to the "Hosts" tab and click **New Host**.
2. **Enter Server Details:**
   - **Alias:** Give your server a recognizable name (e.g., "V2Ray Singapore").
   - **Hostname/IP:** Paste the public IP address provided by your VPS host.
3. **Set Up Authentication:**
   - Under the **SSH** section, enter your **Username** (almost always `root` for a fresh VPS).
   - **If using a Password:** Enter the root password provided by your host.
   - **If using an SSH Key (Recommended):**
     - If your host gave you a private key file (`.pem` or `.ppk`), go to **Settings > Keychain** in Termius, click **+ Key**, and import it.
     - Go back to your Host setup, click the **Keys** dropdown, and select the key you just imported. Leave the password field blank.
4. **Connect.** Double-click your newly created host. If you see a warning about an "unrecognized host key," click **Continue** or **Add and Continue**. You are now in your server's terminal.
---
 
## Step 3: Purchase a Domain Name
 
To secure your connection and disguise your proxy traffic as standard web traffic, you need a domain name.
 
- Use **Cloudflare Registrar** to get a standard `.com` domain for about $10/year.
- Alternatively, use **Namecheap** to grab a heavily discounted domain (like `.online`, `.xyz`, or `.site`) for less than $1 for the first year.
---
 
## Step 4: Route Your Domain Through Cloudflare
 
To manage your DNS and keep everything smooth, you need to link your domain to Cloudflare.
 
**How to change Nameservers in Namecheap:**
 
1. **Add Site to Cloudflare.** Create a free Cloudflare account, click **Add a Site**, and enter your domain. Select the **Free** plan. Cloudflare will scan your records and provide two **Nameservers** (e.g., `ns1.cloudflare.com` and `ns2.cloudflare.com`).
2. **Update Namecheap.** Log into your Namecheap account and go to your **Domain List**.
3. Click the **Manage** button next to your domain.
4. Scroll down to the **Nameservers** section. Click the dropdown (currently showing "Namecheap BasicDNS") and change it to **Custom DNS**.
5. **Paste the Nameservers** into the lines provided.
6. Click the small **green checkmark** to save.
7. **Wait for Propagation.** DNS changes can take anywhere from 15 minutes to 24 hours to fully activate, though it is usually on the faster end.
8. **Point Domain to VPS.** Once active, go to the **DNS** tab in Cloudflare, add an `A Record`, enter `@` for the name, and paste your VPS IP address. **Turn the orange proxy cloud OFF (set to DNS Only)** for now.
---
 
## Step 5: Generate an SSL Certificate
 
Your proxy needs an SSL certificate to encrypt traffic. We will use Certbot to get a free one from Let's Encrypt.
 
Run the following commands in your SSH terminal, one by one:
 
```bash
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:certbot/certbot -y
sudo apt-get install certbot -y
```
 
Now generate the certificate. Replace the placeholder email and domain with your actual details:
 
```bash
sudo certbot certonly --standalone --preferred-challenges http --agree-tos --email youremail@gmail.com -d yourdomain.com
```
 
> **Important:** Once finished, Certbot will output two file paths — one for the **Certificate** and one for the **Private Key**. Copy and paste these paths into a secure notepad document. Do not leak these keys. You will need them for the 3x-ui panel.
 
---
 
## Step 6: Install the 3x-ui Panel
 
Run this installation script to install the graphical interface that will manage your V2Ray connections:
 
```bash
bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)
```
 
- The script will prompt you to set a **Username**, **Password**, and **Port** (choose any random port, e.g., `54321`).
- Once installed, type `x-ui` in the terminal to bring up the menu. You can view your current settings there, including the exact URL/IP and port to access the panel in a browser.
- Open your browser, navigate to that address, and log in with the credentials you just created.
---
 
## Step 7: Configure the VLESS Inbound
 
Inside the 3x-ui panel, go to the **Inbounds** tab and create a new inbound with the following settings:
 
### Basics
 
| Setting | Value |
|---|---|
| Enabled | ✅ Yes |
| Remark | Any name (e.g., "Main-VLESS") |
| Protocol | `vless` |
| Address | Your server's public IP address |
| Port | `443` |
| Total Flow | `0` (Unlimited) |
| Traffic Reset | Never |
| Duration | Empty |
 
### Protocol
 
| Setting | Value |
|---|---|
| Decryption | `none` |
| Encryption | `none` |
 
### Stream
 
- **Transmission:** `TCP`
- Turn off all other toggles in this section.
### Security
 
| Setting | Value |
|---|---|
| Security | `TLS` |
| SNI | Set based on your ISP requirements (e.g., `zoom.us`, `netflix.com`) |
| Public Key | Path from Certbot (Step 5) |
| Private Key | Path from Certbot (Step 5) |
 
Leave everything else at default.
 
### Sniffing
 
- **Enabled:** ✅ Yes
- Select **HTTP** and **TLS** only.
Click **Save** to lock in your inbound settings.
 
---
 
## Step 8: Create a Client Profile
 
1. Navigate to the **Clients** tab in the 3x-ui panel.
2. Click **Add Client**.
3. **Email:** Give the client a recognizable name (e.g., "My-Laptop" or "My-Phone").
4. **Attached Inbound:** Select the VLESS inbound you just created from the dropdown.
5. Click **Create**.
6. Once created, click the **Information (i) icon** next to the client and copy the VLESS configuration link to your clipboard.
---
 
## Step 9: Connect via VPN Client
 
Your server is ready! Now connect to it from your local device.
 
1. **Download a Client.** You can use **Netch** (Windows) or **Netmod**.
2. **Settings Check.** In your client's settings, make sure **TLS AllowInsecure** is enabled.
3. **Import Config.** Go to **Add Server > Add Server from Clipboard**. The app will automatically parse the VLESS link you copied.
4. **Start Connection.** Select your new profile and click **Start / Connect**.
5. **Verify.** Open a browser and search "What is my IP". If your location shows as your VPS location (e.g., Singapore), your V2Ray setup is working perfectly. ✅
