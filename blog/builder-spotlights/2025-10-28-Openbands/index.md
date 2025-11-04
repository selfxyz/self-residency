---
title: "Builder Spotlight: Openbands"
date: "2025-10-31"
author: "Stratos"
project: "Openbands"
links:
  repo: "https://github.com/Openbandsxyz/Openbandsv2"
  demo: "https://openbandsv2.vercel.app/"
  contact: "X: @0xstratos @masaun2551"
self:
  proofs: ["age-18+", "nationality",] # keep what applies
  flow: "QR"
  verifier_learns: "which nationality and if age-18+ yes/no — never raw identity data"
  sdks: ["QRCode SDK"] # edit to match
stack: ["Noir", "Celo", "Base", "zkEmail", "Solidity", "Typescript"] # edit to match
---

![Hero](./assets/hero.png)

## What is Openbands in one line

Openbands is an anonymous proof-of-membership forum

## Why this matters

Todays digital communities force a trade-off. Trust or privacy, but never both. 

When users remain private, trust might collapse under the weight of bots, sybils, and misinformation.  
When platforms demand trust, privacy disappears through exposure, surveillance, and data leaks.

Considering the fast growth of AI, the scale of the problem is growing fast.

- In the first half of 2025 alone, Meta removed over 10 million fake profiles.  
  (Source: [Meta Creators Blog](https://creators.facebook.com/blog/combating-unoriginal-content))

- Over 16 billion credentials were exposed in data breaches in June 2025.  
  (Source: [Cybernews](https://cybernews.com/security/billions-credentials-exposed-infostealers-data-leak))

- Malicious bots now account for 37% of all internet traffic, marking the sixth consecutive year of rising automated activity. (Source: [Thales Group](https://cpl.thalesgroup.com/about-us/newsroom/2025-imperva-bad-bot-report-ai-internet-traffic))

These numbers highlight how fragile online authenticity has become and Openbands addresses this imbalance. We aim to be a Reddit style platform but where communities are verified and privacy is the default.

Users must prove real-world attributes, such as nationality, age, company affiliation, GitHub activity, bank balance, and more, to gain access to gated communities of people who share the same traits. This approach enables genuine, human interaction while minimizing bots, sybils, and misinformation. 

## Use Cases

For citizens:
Citizens can participate in discussions about political or cultural situations within their country. For example, in Germany, there was recently a broad public discussion about the EU’s proposal for chat control — essentially allowing authorities to scan all messaging platforms without any suspicion of illegal activity. Openbands can provide a space where such topics can be discussed by the people directly affected.

An EU citizen community could provide a space where EU legislation and laws are discussed by citizens themselves. It would be interesting to see how citizens vote on new legislation compared to how the EU Parliament votes.

For employees:
Employees can prove their company affiliation and share insights about their company — such as culture, policies, layoffs, and compensation — or ask for advice from industry peers. For example, if someone wants to transition from company A to company B, Openbands can provide a neutral space of verified industry employees to seek advice.

Proving company affiliation while preserving privacy and identity also creates a secure environment for whistleblowers.

More use cases:
- age gated communites for 18+ or 21+
- Student groups for professor and university reviews
- Web3 token gated communities

## Why identity (and why Self)

As mentioned above idenity protocols and passport credentials are very effective to minimize bots, sybils and misinformation. Furthermore, enabeling nationality or age gated communitites is a core feature of our project, therefore using Self Protocol is the perfect solution for our project. Users can prove their nationality or that they are 18+ years old while keeping personal and sensitive information private. This is how we can enable country communitites where only proven citizens can participate in discussions.

**Proofs used:** nationality, age 18+ 
**Verifier learns:** nationality and age if 18+ yes/no
**Flow:** User scans QR, proves the nationality and wether age is 18+ or not

## How it works (short flow)

- User scans QR code
- Verifies the request within the Self app
- After proof is generated it is verified onchain
- Users attributes (nationality, age) are then stored on our deployed contracts on Celo mainnet
Nationality Registriy Contract:
https://celoscan.io/address/0x5aca8d5c9f44d69fa48ccecb6b566475c2a5961a
Age Registriy Contract:
https://celoscan.io/address/0x72f1619824bcd499f4a27e28bf9f1aa913c2ef2a

- User can visit "My Badges" section and view all the badges that are associated with the connected users wallet.

If helpful, drop a quick diagram image in `./assets/Badges.png`.

## What shipped

- We integrated Self Protocol and enabled nationality and age 18+ proofs
- Profile is populated with valid badges within the "My Badges" section
- Updated frontend and UX
- Demo link: https://openbandsv2.vercel.app/

## What’s next

- Enable permissionless community creation. Every user should be able to create his own community based on the badges the user has.
- IPFS storage for censorship-resistant content
- Integrate Semaphore to anonymize wallet addresses for groups
- Explore zkPDF for selective PDF disclosures (like salary statements) and rate limiting nullifier for spam protection
- Launch closed beta by EOY

## Get involved

- Repo: https://github.com/Openbandsxyz
- Issues/Help wanted: link or short note
- Contact: X: @0xstratos @masaun2551

