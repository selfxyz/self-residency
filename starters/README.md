# Starters

Small public examples and quickstarts to get started with Self and related technologies.

## DSC Parser (Swift)
**Repo:** https://github.com/0x471/dsc-parser

Swift app for parsing Digital Signing Certificate (DSC) data from NFC-enabled national IDs and passports.

What it does:
- Reads NFC chip data from digital IDs and passports
- Extracts DSC certificate chain (CSCA -> DSC -> SOD)
- Parses ASN.1/DER encoded certificate structures
- Displays certificate metadata and public keys
---

## Celo ↔ Base Hyperlane Messaging
**Repo:** https://github.com/0x471/celo-base-interop

Cross-chain messaging example using Hyperlane to bridge Celo and Base.

What it does:
- Sends messages from Celo to Base
- Uses Hyperlane's interoperability protocol for cross-chain communication
- Demonstrates smart contract deployment and message passing