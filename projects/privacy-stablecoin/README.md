# Privacy Vault for Stablecoins

**Status:** building  
**What:** Stablecoin vault with private payment flows, enabling compliant yet confidential transfers and lending primitives across Celo and Aztec

**Repos:**

- Aztec backend: [https://github.com/ArturVargas/aztec-starter](https://github.com/ArturVargas/aztec-starter) (branch: `vault-aztec`)
- Frontend: [https://github.com/self-zkResidency/vault-front](https://github.com/self-zkResidency/vault-front)

**Demo:** [https://nomimoney.vercel.app/](https://nomimoney.vercel.app/)  
**Team:** @0xVato  
**Contact:** @0xVato

**How Self fits:** Self proofs are used to gate vault deposits — users must verify jurisdiction and OFAC compliance via zero-knowledge attestations.
Once verified, users can deposit stablecoins (cUSD, USDC, ccCOP) into the vault on Celo, which triggers a private mint in Aztec via the relayer backend.
All flows remain private within Aztec, while Self guarantees compliant access without exposing user data on-chain.

## Quickstart

This project requires setting up several components. Follow these steps in order:

### 1. Backend Aztec (PrivacyVault Contract & Relayer)

```bash
# Clone the Aztec repository on the correct branch
git clone https://github.com/ArturVargas/aztec-starter.git
cd aztec-starter
git checkout vault-aztec

# Install dependencies
pnpm install

# Set up Aztec Sandbox
# (Follow the repository instructions to configure the sandbox)
```

#### Deploy accounts and PrivacyVault contract

```bash
# Deploy Aztec accounts
# (Run the account deployment commands according to Aztec documentation)

# Deploy the PrivacyVault contract
# (Run the contract deployment script)
```

#### Run the Simple Relayer

```bash
# In the same aztec-starter repository, run the relayer
# (Execute the simple_relayer script to connect Celo with Aztec)
```

### 2. Frontend (Self Validation & Celo Integration)

```bash
# Clone the frontend repository
git clone https://github.com/self-zkResidency/vault-front.git
cd vault-front

# Install dependencies
pnpm install

# Configure environment variables (see Env section)
# The frontend connects to the PrivacyVault contract on Celo Sepolia
```

### 3. Run the Frontend

```bash
# In the vault-front directory
pnpm dev
```

The frontend includes Self validation that allows users to verify their jurisdiction and OFAC compliance before depositing into the vault.

## Env

Create `.env` from `.env.example`:
SELF_API_KEY=...
SELF_ENV=sandbox

## Milestones

- [ ] v0 demo scope defined
- [ ] Self proof verified end-to-end
- [ ] Public demo link
- [ ] README quickstart complete

## Updates

- Week 1: <one-line + PR/thread link>
- Week 2: <one-line + PR/thread link>
- Week 3: <one-line + PR/thread link>
