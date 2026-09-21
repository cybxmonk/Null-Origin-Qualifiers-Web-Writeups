# CTFception — Solution Steps

### Step 1: Reconnaissance & Telemetry Intercept
1. Open the CTFception website in your browser.
2. In the interactive terminal, run:
   ```bash
   files
   ```
   or inspect `/archive/evidence/telemetry.json` (or type `cat /archive/evidence/telemetry.json`).
3. Note the downloadable artifact path: `/downloads/kesari_vault_1908.zip` and the hint for the password:
   > *"The vault passphrase is the romanized title of Tilak's monumental philosophical work composed secretly by pencil inside Mandalay Jail between 1908 and 1914 (lowercase, single word, 11 letters)."*

---

### Step 2: Unlocking the Encrypted Vault
1. The philosophical magnum opus written by Lokmanya Tilak in Mandalay prison is **Gita Rahasya** (*Shrimadh Bhagavad Gita Rahasya*).
2. The 11-letter lowercase password is:
   ```text
   gitarahasya
   ```
3. Download and extract `/downloads/kesari_vault_1908.zip` using the password `gitarahasya`:
   ```bash
   # Using 7-Zip / Linux unzip:
   7z x -pgitarahasya kesari_vault_1908.zip

   # Or using the provided helper script:
   python downloads/extract_vault.py gitarahasya
   ```
4. The extracted archive contains:
   - `MANDALAY_DISPATCH_1908.txt`
   - `beacon_payload.enc`
   - `telegraph_decoder.py`

---

### Step 3: Decrypting the Telegraph Beacon
1. Read `MANDALAY_DISPATCH_1908.txt`. It states that the courier cipher is keyed using the launch date of the eternal chronicle: 25th Sept (`DDMM` format $\rightarrow$ `2509`).
2. Decode the Base64 payload in `beacon_payload.enc` (`fnp7dHN7aXhtcnFtenRvenpnf3d7dnx8bQQJCQo=`) using XOR with key `2509`:
   ```bash
   python telegraph_decoder.py 2509
   ```
   *(Alternatively, in CyberChef: `From Base64` $\rightarrow$ `XOR` with key `2509` in UTF-8)*.
3. The decoded beacon key is:
   ```text
   LOKMANYA_GATHA_CHRONICLE_1908
   ```

---

### Step 4: Reconstructing the Archive & Capturing the Flag
1. Return to the CTFception web terminal and run:
   ```
   reconstruct LOKMANYA_GATHA_CHRONICLE_1908
   ```
2. The client-side Web Crypto AES-256-GCM engine decrypts the sealed record and restores the transmission.
3. The flag is revealed:
   ```text
   NullOrigin{L0km4ny4_G4th4_25S3pt_Sw4r4jy4_PRX02_}
   ```
