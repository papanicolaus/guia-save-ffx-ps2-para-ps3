# How to Convert a Final Fantasy X Save from PS2 to Final Fantasy X HD Remaster on PS3

[**English**](README.md) | [Português (Brasil)](README.pt-BR.md)

## A detailed, public, and safety-conscious guide

**Last revised:** September 14, 2026  
**Game:** Final Fantasy X  
**Source:** PlayStation 2  
**Destination:** Final Fantasy X HD Remaster on PlayStation 3  
**Computer operating system used:** Windows

> This procedure modifies save data and is not an official Square Enix or Sony feature. Proceed at your own risk. Never work on the only copy of your save. This guide is based on a real conversion that ended with the save being recognized and successfully loaded on a PS3.

---

## 1. The working process in one diagram

```text
PS2 Memory Card
        ↓
.psu backup with wLaunchELF/uLaunchELF
        ↓
raw file extraction with PS2 Save Builder
        ↓
conversion with FFX HD Cross-platform Save Converter
        ↓
decrypted PS3 SAVES file
        ↓
replace SAVES inside a genuine PS3 base save
        ↓
encryption and integrity rebuilding with Bruteforce Save Data
        ↓
copy through the XMB and load in FFX HD Remaster
```

The most important point is this: **do not create a complete PS3 save from scratch**. It is much safer to create a genuine base save on the destination PS3 and preserve its structure, `PARAM.SFO`, `PARAM.PFD`, icons, and ownership data. Replace only the game-data file named `SAVES` inside that structure.

---

## 2. What was confirmed in the real conversion

In the case that produced this guide:

- the raw PS2 file was `BISLPS-25088FF090600`;
- its name indicated a **Final Fantasy X International** save associated with the Japanese ID `SLPS-25088`;
- the original save had approximately 99 hours of playtime and was at Bevelle/Via Purifico;
- the destination was the European PS3 release of HD Remaster, with a folder beginning with `BLES01880`;
- a short base save of approximately 23 minutes was created on the PS3;
- the converter produced a new file named `SAVES`;
- an attempt to finish the process with Apollo Save Tool failed with error `8002920E`;
- completing the process with Bruteforce Save Data worked;
- the converted save initially appeared in the load menu with the incorrect time `24:17:12`, but the game recognized and loaded it;
- incorrect information in the slot preview is a known limitation of this kind of conversion and usually corrects itself after loading the game and saving again at a save point.

The names and times above are diagnostic examples only. Your file, regional ID, playtime, and folder name will probably be different.

---

## 3. What you will need

### On the PS2

- a PS2 capable of running homebrew, usually through Free McBoot;
- `wLaunchELF` or `uLaunchELF`, commonly launched through a `BOOT.ELF` file;
- the Memory Card containing the save;
- a PS2-compatible USB drive, preferably a simple one formatted as FAT32.

### On the computer

- Windows;
- Python 3;
- [Final Fantasy X HD Cross-platform Save Converter](https://github.com/mrhappyasthma/Final-Fantasy-X-HD-Cross-platform-Save-Converter);
- PS2 Save Builder;
- Bruteforce Save Data, also called BSD;
- enough storage for several backup copies.

### On the PS3

- a working installation of Final Fantasy X HD Remaster;
- access to the same PS3 user account on which the final save will be played;
- a base save created by the game under that user;
- a FAT32 USB drive or external drive using the standard `PS3/SAVEDATA` structure.

### Optional

- [Apollo Save Tool for PS3](https://github.com/bucanero/apollo-ps3), useful for inspecting, copying, and backing up saves;
- an FTP client and a manager such as multiMAN if you need to move files inside the PS3. This route is not required for the main method and was where error `8002920E` occurred in the real case.

> You do not need to download a game ISO for this conversion. An ISO would only be useful for testing the save in an emulator or checking the region of the PS2 release. That is an optional detour, not part of the required process.

---

## 4. Safety rules before you begin

Create at least these four copies:

1. **Backup A:** the untouched original export from the PS2 Memory Card.
2. **Backup B:** the raw file extracted from the `.psu` package.
3. **Backup C:** the original base-save folder created by the PS3.
4. **Backup D:** the assembled and encrypted final folder before copying it to the PS3.

Suggested organization:

```text
FFX_CONVERSION
├── 01_PS2_ORIGINAL_PSU
├── 02_PS2_RAW_FILE
├── 03_PS3_ORIGINAL_BASE_SAVE
├── 04_CONVERTED_SAVES
└── 05_PS3_FINAL_ENCRYPTED
```

Never move your only original. Always make a copy.

### Information you should not publish

When asking for help on a forum, Reddit, Discord, or GitHub, do not publish:

- your PSN Account ID;
- your PS3 User ID;
- your Console ID, PSID, or IDPS;
- codes shown by resigning tools;
- your PS3 local IP address;
- your Windows account name;
- full paths to personal folders on your computer;
- `PARAM.SFO` or `PARAM.PFD` from a personal save unless there is a clear technical reason and you understand the risk;
- another person's complete save folder.

Cover these fields in screenshots. Use generic paths in public guides, such as:

```text
C:\FFX_CONVERSION\...
X:\PS3\SAVEDATA\...
```

This method reduces the need to enter IDs manually because it uses a base save created by the destination user on the destination PS3.

---

## 5. Understanding the files

| File or folder | Source | Purpose |
| --- | --- | --- |
| game folder inside `mc0:/` or `mc1:/` | PS2 Memory Card | Contains the original save and its metadata |
| `.psu` file | wLaunchELF/uLaunchELF | Backup package that preserves Memory Card attributes |
| file such as `BISLPS-25088FF090600` | extracted from `.psu` | Raw FFX save data from the PS2; input for the converter |
| `SAVES` | produced by the converter | FFX data in the internal format expected by the PS3 HD Remaster, still decrypted |
| `PARAM.SFO` | PS3 base save | Save metadata, title, folder, and ownership-related data |
| `PARAM.PFD` | PS3 base save | File integrity, hashes, and protection data |
| `ICON0.PNG` and other resources | PS3 base save | Icon and other possible XMB images |

Do not confuse these two layers:

- the converter fixes the internal **Final Fantasy X** structure and checksum;
- Bruteforce Save Data handles encryption and integrity for the **PlayStation 3** save folder.

Both layers are required.

---

## 6. Stage A: export the save from the PS2 Memory Card

### 6.1 Prepare the USB drive

1. Format the USB drive as FAT32.
2. Place the `BOOT.ELF` for wLaunchELF/uLaunchELF on it if Free McBoot is not already configured to launch the program.
3. Insert the Memory Card containing the save into the PS2.
4. Insert the USB drive into the PS2.
5. Start wLaunchELF/uLaunchELF.

wLaunchELF is the successor to uLaunchELF, so both names appear in older guides. It works as a PS2 file manager.

### 6.2 Locate the save

1. Open `FileBrowser`.
2. Enter `mc0:/` if the Memory Card is in slot 1, or `mc1:/` if it is in slot 2.
3. Locate the Final Fantasy X folder.

The folder name depends on the region and release. Examples include:

- `SLPS-25088`: Japanese Final Fantasy X International;
- `SCES-50490`: European/PAL release;
- `SLUS` identifiers or names beginning with `BASLUS`: North American releases.

In the real case, the identifying file was `BISLPS-25088FF090600`.

If you have multiple FFX saves, check the correct slot in the game before exporting. Write down the location, playtime, character name, and save date.

### 6.3 Create the `.psu` backup

1. Highlight or mark the entire save folder on the Memory Card.
2. Open the operations menu, usually with `R1`.
3. Choose `Copy`.
4. Go back and enter `mass:/`, which represents the USB drive.
5. Open the menu again with `R1`.
6. Choose **`psuPaste`**, not the regular `Paste` command.
7. Wait for completion without turning off the console or removing the USB drive.

### Why use `psuPaste`?

The PSU format preserves attributes, timestamps, and metadata specific to the PS2 Memory Card. Copying the folder as ordinary files can produce an incomplete backup that is difficult to import again. For an export intended for a computer, `psuPaste` is the correct option.

### 6.4 Check the backup on the computer

1. Safely eject the USB drive.
2. Connect it to the computer.
3. Copy the `.psu` file to `01_PS2_ORIGINAL_PSU`.
4. Duplicate it and keep one copy untouched.
5. Confirm that its file size is greater than zero.

---

## 7. Stage B: extract the raw file from `.psu`

1. Open PS2 Save Builder.
2. Select `File > Open` and open the `.psu` file.
3. The program will display the files stored inside the package.
4. Find the main file whose name includes the game release ID.
5. Do not select `icon.sys`, icons, or image files.
6. Right-click the main file and choose `Extract`.
7. Save it in `02_PS2_RAW_FILE`.

In the real case, the correct file was:

```text
BISLPS-25088FF090600
```

The converter repository generally refers to files beginning with `BISLPS` or `BASLUS`. The prefix can differ in other regions. The important point is to select the large game-data file rather than a visual resource.

### How to check that you extracted the correct file

- its name should resemble the regional ID of the PS2 release;
- it should not be a PNG, an icon, or `icon.sys`;
- the converter should accept it as PS2 input;
- when tested with the same PS2 game version, it should contain the expected progress.

---

## 8. Stage C: resolve the PS2 save region

This caused real confusion during the conversion.

`SCES-50490` and `SLPS-25088` are different releases:

- `SCES-50490` identifies a European/PAL release;
- `SLPS-25088` identifies the Japanese Final Fantasy X International release;
- fan translations may be based on either release;
- a translated ISO being described as “European” does not turn an `SLPS-25088` save into an `SCES-50490` save.

If a PS2 game does not recognize the save during testing, first suspect a region mismatch. Do not blindly rename the folder or main file. Renaming can make a save appear in a list without making its internal data compatible.

For the conversion documented here, the raw `BISLPS-25088FF090600` file was accepted by the converter and converted directly into a PS3 `SAVES` file. No PS2-to-PS2 region conversion was needed first.

---

## 9. Stage D: convert the raw PS2 file

### 9.1 Prepare the converter

1. Open the [FFX HD Cross-platform Save Converter repository](https://github.com/mrhappyasthma/Final-Fantasy-X-HD-Cross-platform-Save-Converter).
2. Download the complete project, for example through `Code > Download ZIP`.
3. Extract the entire ZIP into one folder.
4. Do not move only `ffx.pyw`; it depends on the `Modules` folder and other project files.
5. Install Python 3 if it is not already installed.

The converter code requires Python 3.

### 9.2 Run the converter

First try opening `ffx.pyw` by double-clicking it. If nothing happens:

1. open Terminal or Command Prompt inside the project folder;
2. run:

```bat
py -3 ffx.pyw
```

If the `py` command is unavailable, try:

```bat
python ffx.pyw
```

### 9.3 Configure the conversion

In the program window:

1. under `Game`, choose `Final Fantasy X`;
2. under `Save File Type`, choose `PS2`;
3. under `Target Console`, choose `PS3 (decrypted)`;
4. click `Convert save file`;
5. select the raw file extracted with PS2 Save Builder;
6. confirm the conversion;
7. locate the new file named `SAVES`.

Copy this file to `04_CONVERTED_SAVES` and do not edit it manually.

### Important note

The resulting `SAVES` file is **decrypted**. It cannot be placed on the PS3 by itself. It must first be inserted into a valid base save and encrypted while rebuilding `PARAM.PFD`.

---

## 10. Stage E: create a genuine base save on the PS3

This avoids inventing metadata or using another person's account codes.

1. On the PS3 user account where you intend to play, launch the exact Final Fantasy X HD Remaster release that will receive the save.
2. Start a new game or use disposable progress.
3. Save normally at a save point.
4. Exit the game.
5. On the XMB, open `Game > Saved Data Utility (PS3)`.
6. Locate the Final Fantasy X HD Remaster save.
7. Press `Triangle > Copy` and copy it to a FAT32 device.
8. On the computer, open:

```text
X:\PS3\SAVEDATA\
```

9. Copy the entire save folder to `03_PS3_ORIGINAL_BASE_SAVE`.
10. Make a second copy to use as your working copy.

In the European release used for the real conversion, the folder name was similar to:

```text
BLES01880SAVEDIR---------------
```

Do not assume that this name applies to every release. Use the folder created by your own copy of the game.

A minimum observed structure was:

```text
PS3
└── SAVEDATA
    └── BLES01880SAVEDIR---------------
        ├── ICON0.PNG
        ├── PARAM.PFD
        ├── PARAM.SFO
        └── SAVES
```

Your folder may contain additional visual resources. Preserve all of them.

### Why must the base save come from your own PS3?

It already contains:

- the correct destination release ID;
- valid metadata;
- the structure expected by the XMB;
- data belonging to the correct user;
- a legitimate `PARAM.PFD` that can be rebuilt;
- icons and resources for that specific release.

Downloading a random base save adds unnecessary region, ownership, and resigning problems.

---

## 11. Stage F: decrypt the base save with Bruteforce Save Data

> Bruteforce Save Data is old software. Obtain it from a reputable source, scan the installer with antivirus software, and, if possible, run it offline or inside a virtual machine. Do not permanently disable system protections merely to run it.

### 11.1 If the program requests a PS3 profile

Some installations already have a profile; others ask the user to create one. Use only information extracted from a save created by **your own PS3 and your own user account**.

The usual process in older versions is:

1. open `Settings > Global Settings`;
2. create or select a local profile;
3. point the tool to the base save's `PARAM.SFO` when requested;
4. let the tool read the necessary identifiers, or locally enter requested fields such as User ID and Console ID/PSID;
5. save this configuration only on the computer used for the conversion.

Never include these values in screenshots, tutorials, ZIP files, or public support requests. Do not copy another user's values or reuse example values from old guides. Every console and user can have different identifiers.

Because this method uses a base save created by the destination user, you do not need to turn it into a save belonging to another account. The configuration simply allows the tool to open and correctly rebuild protection for the local save.

### 11.2 Decrypt the save

1. Open Bruteforce Save Data.
2. Set `Path for SAVEDATA folders` to the directory containing the save folders, for example:

```text
C:\FFX_CONVERSION\WORK\PS3\SAVEDATA
```

Do not point it directly to the `SAVES` file.

3. Wait for the program to list the Final Fantasy X HD Remaster save.
4. Select the correct save.
5. Use the file-decryption option, usually:

```text
Decrypt PFD > Decrypt All Files
```

In some versions it is shown simply as `Decrypt all files`.

6. Confirm the operation.
7. Check that the save status has turned green.
8. Check whether the tool created a marker file such as:

```text
~files_decrypted_by_pfdtool.txt
```

This marker records which files are decrypted; it is not a game file.

---

## 12. Stage G: replace only `SAVES`

With the base save decrypted:

1. open the working copy of the base-save folder;
2. temporarily rename the original `SAVES` to something such as `SAVES.base.bak`, or keep it outside the working folder in your backup;
3. copy the converter-produced `SAVES` into the folder;
4. confirm that its final name is exactly `SAVES`, with no extension;
5. do not replace `PARAM.SFO`;
6. do not manually replace `PARAM.PFD`;
7. do not replace the icons;
8. do not mix in files from another PS3 save.

The folder must remain the genuine base save, with only its game-data file replaced.

### Useful verification

Before and after replacement, check:

- file size;
- modification date;
- exact filename;
- absence of a hidden extension such as `SAVES.bin` or `SAVES.txt`.

For a stricter check, calculate the SHA-256 hash of the converted `SAVES` before copying it and again inside the folder. The hashes must match.

In PowerShell:

```powershell
Get-FileHash .\SAVES -Algorithm SHA256
```

The hash is safe to share; the personal save file is not.

---

## 13. Stage H: encrypt and rebuild the save

Return to Bruteforce Save Data with the same save selected.

Use:

```text
Encrypt PFD > Encrypt Decrypted Files
```

This detail was decisive in the real conversion:

- use `Encrypt Decrypted Files`;
- do not run `Update PFD` first;
- do not use `Encrypt All Files` as your first attempt;
- do not apply a second checksum with Apollo afterward.

The converter has already prepared the internal FFX checksum. At this stage, Bruteforce should encrypt the replaced file and update protection for the PS3 save set.

After completion:

1. confirm that no error message appeared;
2. confirm that the save is no longer shown as decrypted;
3. remove any auxiliary `~files_decrypted_by_pfdtool.txt` file if the tool did not remove it automatically;
4. do not remove legitimate base-save files;
5. copy the completed folder to `05_PS3_FINAL_ENCRYPTED`.

---

## 14. Stage I: return the save to the PS3

On the FAT32 device, create this exact structure:

```text
X:\PS3\SAVEDATA\SAVE_FOLDER\
```

Example from the European release used in the test:

```text
X:\PS3\SAVEDATA\BLES01880SAVEDIR---------------\
```

The folder must contain `PARAM.SFO`, `PARAM.PFD`, the encrypted `SAVES`, and all legitimate base-save files.

Then:

1. safely eject the device;
2. connect it to the PS3;
3. on the XMB, open `Game > Saved Data Utility (PS3)`;
4. open the USB device;
5. select the Final Fantasy X HD Remaster save;
6. press `Triangle > Copy`;
7. if a save already exists in internal storage, confirm replacement only after verifying that your original backup is safe;
8. launch the game normally;
9. choose **`Load`**, not `Continue`;
10. attempt to load the converted slot.

---

## 15. The slot shows the wrong playtime or party. What now?

This can be normal.

The converter project documents two known effects:

- a custom protagonist name may be reset to Tidus;
- the load-menu preview may show incorrect playtime and party members.

In the real conversion, the original save had approximately 99 hours, while HD Remaster initially displayed `24:17:12`. The game still recognized and loaded it.

Use the correct test:

1. try loading the slot even if the preview looks wrong;
2. check the location, characters, items, equipment, and progress inside the game;
3. go to a save point;
4. save to another slot, preserving the first converted slot;
5. return to the title screen;
6. check the preview again and load the new slot.

A new save made by HD Remaster itself usually rebuilds the preview metadata correctly.

---

## 16. Errors encountered in the real conversion and their solutions

### Error 1: testing with the wrong PS2 game release

**Symptom:** the save does not appear or is not recognized when testing an ISO.

**Likely cause:** confusion between `SCES-50490` and `SLPS-25088`.

**Solution:** identify the region from the save filename and use the matching release for testing. In the real case, `BISLPS-25088FF090600` indicated FFX International/`SLPS-25088`, not European `SCES-50490`.

**Lesson:** ISO testing is not required for conversion to PS3. If you test, the regions must match.

### Error 2: extracting or selecting the wrong file inside `.psu`

**Symptom:** the converter rejects the file, closes, or creates invalid output.

**Likely cause:** `icon.sys`, an image, or another resource was selected.

**Solution:** open `.psu` in PS2 Save Builder and extract the large file whose name contains the game ID, such as `BISLPS-25088FF090600`.

### Error 3: running `ffx.pyw` by itself

**Symptom:** the window does not open or a missing-module error appears.

**Likely cause:** the script was moved away from the project's `Modules` folder.

**Solution:** extract and keep the entire repository together, then run `ffx.pyw` with Python 3 from inside it.

### Error 4: the PS3 still shows the short base-save progress

**Symptom:** after the supposed import, the game still shows the disposable progress, such as 23 minutes.

**Cause:** the converted `SAVES` did not reach the final folder, was placed in the wrong directory, or was not encrypted/imported correctly.

**Solution:** stop, return to the backup, and compare the size, date, and hash of `SAVES` at every stage. If the short progress remains, you are still loading the base save's original `SAVES`.

### Error 5: the Apollo temporary-directory route fails

**Attempted procedure:** copy the `BLES01880` base save with Apollo; export or decrypt `SAVES` to a temporary path such as `/dev_hdd0/tmp/apollo/SAVE_FOLDER/SAVES`; replace it over FTP using a manager such as multiMAN; then return to Apollo and attempt checksum, patching, and resign operations.

**Result:** error `8002920E`.

**Practical diagnosis:** replacing a file inside Apollo's temporary directory did not guarantee that Apollo would produce a final folder with valid encryption and `PARAM.PFD`. The temporary directory must not be mistaken for an XMB-ready save.

**Working solution:** abandon that working copy, restore the untouched base save, and complete decryption, `SAVES` replacement, and `Encrypt Decrypted Files` with Bruteforce Save Data.

### Error 6: Bruteforce does not list the save

**Likely causes:**

- the configured path points to an individual save folder instead of the `SAVEDATA` directory;
- the `PS3/SAVEDATA` structure is broken;
- `PARAM.SFO` is missing;
- the base save was copied incompletely;
- the device or folder is not writable.

**Solution:** keep the complete PS3-created folder and set `Path for SAVEDATA folders` to the directory containing the save folder.

### Error 7: the save turns green in Bruteforce

**This is not necessarily an error.** Green usually means the files have been decrypted and are available for editing.

After replacing `SAVES`, finish with:

```text
Encrypt PFD > Encrypt Decrypted Files
```

### Error 8: “Corrupted Data” appears on the XMB

**Most common causes:**

- `SAVES` is still decrypted;
- `PARAM.PFD` was not updated;
- the regional folder is wrong;
- `PARAM.SFO` came from another release or user;
- the directory structure is wrong;
- a hidden extension was added to the filename;
- copying was interrupted.

**Solution:** do not repeatedly attempt to repair the same corrupted folder. Return to an untouched base save from your own PS3, decrypt it again, replace only `SAVES`, and run `Encrypt Decrypted Files`.

### Error 9: the save appears, but the preview shows the wrong time

**Real example:** `24:17:12` instead of approximately 99 hours.

**Solution:** load the save. If the internal progress is correct, save again from inside the game to another slot. An incorrect preview alone does not prove corruption.

### Error 10: `Continue` opens the wrong location

**Cause:** `Continue` may open the last previously used slot rather than the converted one.

**Solution:** choose `Load` and manually select the converted slot.

### Error 11: `mass:/` does not appear in wLaunchELF

**Likely causes:** an incompatible USB drive, unsupported filesystem, or the drive being connected too late.

**Solutions:** use FAT32; test a different, preferably smaller and older USB drive; connect it before launching wLaunchELF; restart FileBrowser; avoid adapters and complex partition layouts.

### Error 12: the backup was copied with ordinary `Paste`

**Risk:** loss of Memory Card-specific attributes.

**Solution:** export again using `Copy` at the source and `psuPaste` inside `mass:/`.

---

## 17. Why the Bruteforce route worked better

The successful method kept three responsibilities separate:

1. **PS2 Save Builder:** extract the correct raw file from the `.psu` package.
2. **FFX HD Cross-platform Save Converter:** transform the internal FFX data and recalculate the game checksum.
3. **Bruteforce Save Data:** place those data inside a legitimate PS3 structure, encrypt them, and rebuild PFD protection.

The Apollo attempt mixed temporary-directory operations, FTP replacement, checksums, and resigning. Once `8002920E` appeared, there was no guarantee that every protection layer was synchronized.

Apollo remains useful. For this specific case and tested sequence, however, the PC method made it clearer which file was decrypted and exactly when encryption would be rebuilt.

---

## 18. Short checklist for the proven route

- [ ] Back up the Memory Card.
- [ ] Export the FFX folder with `psuPaste` to create `.psu`.
- [ ] Open `.psu` in PS2 Save Builder.
- [ ] Extract the main game file, not an icon.
- [ ] Open `ffx.pyw` with Python 3.
- [ ] Choose `PS2` input and `PS3 (decrypted)` output.
- [ ] Generate `SAVES`.
- [ ] Create a base save in FFX HD Remaster on the destination PS3.
- [ ] Copy and duplicate the complete base-save folder.
- [ ] Decrypt the base save with Bruteforce Save Data.
- [ ] Replace only `SAVES`.
- [ ] Run `Encrypt PFD > Encrypt Decrypted Files`.
- [ ] Place the final folder in `PS3/SAVEDATA` on the FAT32 device.
- [ ] Copy it through the XMB Saved Data Utility.
- [ ] Open the game and choose `Load`.
- [ ] Load the slot even if its preview initially looks wrong.
- [ ] Check the progress inside the game.
- [ ] Save again, preferably to another slot.

---

## 19. Troubleshooting decision tree

### The converter rejects the input

Return to `.psu` and confirm that you extracted the main game-data file rather than an icon or the `.psu` package itself.

### Bruteforce cannot find the save

Check the `PS3/SAVEDATA/SAVE_FOLDER` structure and point the program to `SAVEDATA`.

### The XMB shows “Corrupted Data”

The problem is in the PS3 layer: encryption, `PARAM.PFD`, `PARAM.SFO`, region, or folder structure. Return to the untouched base save.

### The XMB accepts it, but the game does not show the slot

Confirm that the folder matches the same region/Title ID as the destination installation and that the final file is named exactly `SAVES`.

### The game shows strange information for the slot

Try loading it. The preview can be incorrect after conversion.

### The game loads the short base-save progress

The new `SAVES` was not actually incorporated. Compare date, size, and hash, then repeat the replacement stage.

### The game loads the correct old progress

Go to a save point, save to another slot, restart the game, and test again. The conversion is complete.

---

## 20. What not to do

- Do not work on the only copy of the Memory Card save.
- Do not use an ISO from a different region to conclude that the save is lost.
- Do not select `icon.sys` as converter input.
- Do not move `ffx.pyw` away from `Modules`.
- Do not place a decrypted `SAVES` directly on the PS3.
- Do not replace `PARAM.SFO` or `PARAM.PFD` with arbitrary files.
- Do not use another person's base save when you can create your own.
- Do not repeatedly apply multiple checksum routines “just to be safe.”
- Do not continue using a folder that already produced `8002920E`; return to a clean backup.
- Do not trust only the time shown in the slot preview.
- Do not use `Continue` as the definitive test.
- Do not publish account IDs, console IDs, IP addresses, or personal paths.
- Do not distribute an ISO, game files, or personal saves with this guide.

---

## 21. How to ask for help without exposing personal data

A useful support request includes:

```text
Source: PS2 FFX, region/ID [PROVIDE]
Generic raw filename: [PROVIDE]
Export format: .psu
Destination: PS3 FFX HD Remaster, Title ID [PROVIDE]
Converter commit/version: [PROVIDE]
Failed stage: [PROVIDE]
Error message or code: [PROVIDE]
SAVES file size before/after: [PROVIDE]
SAVES SHA-256: [PROVIDE]
Was the base save created on the destination PS3? yes/no
Was SAVES re-encrypted? yes/no
```

You may publish hashes, file sizes, and commercial release IDs. Do not publish identifiers unique to your account or console.

---

## 22. Technical references

- [FFX HD Cross-platform Save Converter, official repository](https://github.com/mrhappyasthma/Final-Fantasy-X-HD-Cross-platform-Save-Converter)
- [Converter wiki: extracting PS2 saves](https://github.com/mrhappyasthma/Final-Fantasy-X-HD-Cross-platform-Save-Converter/wiki/Playstation-2-(PS2)-Guide)
- [Converter wiki: PS3 saves](https://github.com/mrhappyasthma/Final-Fantasy-X-HD-Cross-platform-Save-Converter/wiki/Playstation-3-(PS3)-Guide)
- [Converter wiki: known issues](https://github.com/mrhappyasthma/Final-Fantasy-X-HD-Cross-platform-Save-Converter/wiki/Known-Issues)
- [Apollo Save Tool, official repository](https://github.com/bucanero/apollo-ps3)
- [wLaunchELF, official repository](https://github.com/ps2homebrew/wLaunchELF)
- [Apollo save decrypters and checksum fixers](https://github.com/bucanero/save-decrypters)

### Source limitations

The converter author lists PS2 → PS3 as implemented but not personally verified in the project's compatibility table. The PS3 wiki also leaves the re-encryption section for saves imported from another platform incomplete. This guide therefore emphasizes the route that was actually completed in the real case: use a legitimate base save from the destination PS3, replace only `SAVES`, and run `Encrypt Decrypted Files` in Bruteforce Save Data.

---

## 23. Conclusion

The conversion is possible, but it involves three formats and two different integrity layers. The most common mistake is assuming that generating `SAVES` completes the process. It does not: that file must still be inserted into a legitimate PS3 save folder and re-encrypted.

The safest route is conservative:

- preserve the original PS2 save;
- extract only the correct data;
- let the converter handle FFX's internal transformation;
- let a genuine base save provide the PS3 identity and structure;
- let Bruteforce encrypt only the decrypted files;
- test through `Load`;
- save again from inside the game.

If the preview looks wrong but the game loads and the internal progress is correct, the conversion succeeded.
