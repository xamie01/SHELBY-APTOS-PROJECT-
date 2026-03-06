# ShelbyAI: Verifiable Decentralized AI Pipeline

A high-performance AI training and inference pipeline built on **Shelby**, the decentralized hot-storage protocol by Aptos Labs and Jump Crypto.

## 🎯 Project Goal
To bridge the gap between decentralized storage and high-performance AI. ShelbyAI enables researchers to train models on verifiable, rights-gated data without the latency of traditional decentralized networks or the "black box" nature of centralized clouds.

## 🛠 Tech Stack
- **Storage Layer:** Shelby Protocol (Sub-second hot storage)
- **Control Plane:** Aptos Blockchain (Move Smart Contracts)
- **Compute Layer:** Python (PyTorch/Transformers)
- **Development:** GitHub Codespaces (32-core CPU) + [RunPod/Lambda] for GPU Burst
- **Communication:** Fiber-optic node networking

## 🚀 Key Features
- **Instant RAG:** Sub-second retrieval of decentralized context for LLMs.
- **On-Chain Attribution:** Every data 'read' is logged and monetized via Aptos.
- **Hybrid Compute:** Logic lives in Codespaces; heavy training bursts on rented Cloud GPUs.

## 🏗 Setup & Workflow
1. **Initialize Environment:** Run `aptos init` in the devcontainer.
2. **Data Sharding:** Use the Shelby SDK to shard datasets for the decentralized mesh.
3. **Training:** - Rent a temporary GPU (RunPod/Lambda).
    - Use the provided `checkpoint_manager.py` to save progress.
    - Sync weights back to Shelby or GitHub.
4. **Release:** Terminate GPU instances immediately after training to minimize costs.

---
*Built for the Shelby Early Access Program.*
