# Licensing

The L2 Telegram Bot Hub requires a valid license to operate. The license is tied to your server's hardware ID (Machine Code).

### Licensing Process

1. **Start the Hub**: When you start the Hub for the first time without a license file, or if the license is invalid, it will stop and display an error message in the console.
2. **Obtain Machine Code**: Look for a line in the console that says:
   `Your machine code: ABC123XYZ...`
3. **Send to Maintainer**: Copy this code and send it to the bot maintainer.
4. **Receive License Key**: The maintainer will generate a license key specifically for your machine.
5. **Create License File**: 
    - Create a new text file named `license.txt` inside the `config/` directory of your Hub installation.
    - Paste the license key you received into this file.
    - Ensure there are no extra spaces or line breaks.
6. **Restart Hub**: Start the Hub again. It should now verify the license and start successfully.

### Troubleshooting
- **Verification Failed**: If you see "LICENSE VERIFICATION FAILED!", double-check that the `license.txt` file is in the `config/` folder and contains the exact string provided by the maintainer.
- **Hardware Changes**: If you move the Hub to a different server or make significant hardware changes, your Machine Code may change, and you will need a new license key.
