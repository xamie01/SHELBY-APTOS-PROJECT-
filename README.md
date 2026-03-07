

# ShelbyTrain 🧠⛓️
### Verifiable ML Dataset Registry for the Shelby Protocol

> **The "GitHub for AI training data" — where every epoch pays creators and every sample is cryptographically provable.**

Built for the **Shelby Early Access Program** by Aptos Labs × Jump Crypto.

---

## 🎯 Vision

Machine learning is eating the world, but training data remains a black box:
- **Provenance unknown**: Did this dataset include copyrighted material? Poisoned data?
- **Creators unpaid**: OpenAI trained on your Flickr photos. You got $0.
- **Access fragmented**: 50 different cloud buckets, 50 different auth systems.

**ShelbyTrain** fixes this with **verifiable, rights-gated, monetized** datasets on Shelby's decentralized hot-storage.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      USER LAYER                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │ Data        │  │ Model       │  │ Registry            │  │
│  │ Providers   │  │ Trainers    │  │ Explorer            │  │
│  │ (Upload)    │  │ (Train)     │  │ (Discover)          │  │
│  └──────┬──────┘  └──────┬──────┘  └─────────────────────┘  │
└─────────┼────────────────┼──────────────────────────────────┘
          │                │
          ▼                ▼
┌─────────────────────────────────────────────────────────────┐
│                   SHELBY PROTOCOL LAYER                      │
│  ┌─────────────────┐  ┌──────────────────────────────────┐  │
│  │ Aptos Blockchain│  │ Shelby Storage Network           │  │
│  │ (Control Plane) │  │ (Data Plane)                     │  │
│  │                 │  │                                  │  │
│  │ • Dataset       │  │ • Erasure-coded blobs            │  │
│  │   Registry      │  │   (Clay codes, 2x replication)   │  │
│  │ • Rights        │  │ • Sub-second reads via fiber     │  │
│  │   Management    │  │ • Byte-range queries             │  │
│  │ • Micropayments │  │ • Sampling-based verification    │  │
│  │   (per read)    │  │                                  │  │
│  └─────────────────┘  └──────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## ✨ Key Features

| Feature | Description | Shelby Tech Used |
|---------|-------------|------------------|
| **🔍 Verifiable Lineage** | Every dataset commit links to on-chain blob IDs with Merkle proofs | Shelby sampling audits + Aptos finality |
| **💰 Automatic Attribution** | Trainers pay creators per batch via Shelby's "paid reads" | Off-chain payment channels, on-chain settlement |
| **⚡ Streaming Training** | No download delays — PyTorch streams chunks sub-second | Shelby RPC nodes + fiber backbone |
| **🛡️ Rights-Gated Access** | Smart contracts enforce training vs. inference licenses | Move-based access control |
| **📊 Dataset Provenance** | Full graph: derived datasets, transformations, model lineage | Aptos event logs |

---

## 🚀 Quick Start

### Prerequisites
- Python 3.10+
- [Aptos CLI](https://aptos.dev/tools/aptos-cli/) (for devnet/testnet)
- Shelby SDK (provided in Early Access Program)

### 1. Initialize Environment

```bash
# Clone and setup
git clone https://github.com/yourname/shelbytrain.git
cd shelbytrain
pip install -e .

# Initialize Aptos profile (testnet for development)
aptos init --network testnet

# Configure Shelby endpoints (devnet access required)
export SHELBY_RPC_URL="https://rpc.devnet.shelby.xyz"
export SHELBY_STORAGE_ENDPOINT="https://storage.devnet.shelby.xyz"
```

### 2. Upload a Dataset (Data Provider)

```python
from shelbytrain import DatasetUploader
from shelby import ShelbyClient

# Initialize Shelby client
client = ShelbyClient(
    rpc_url=os.getenv("SHELBY_RPC_URL"),
    wallet="0x..."  # Your Aptos address
)

# Upload with metadata and rights configuration
uploader = DatasetUploader(client)
dataset_id = uploader.publish(
    local_path="./imagenet-subset/",
    name="ImageNet-1K-Subset-Cleaned",
    description="Curated, deduplicated ImageNet subset for computer vision research",
    license="CC-BY-SA-4.0",
    rights_config={
        "training": {"price_per_gb": 0.05, "currency": "APT"},
        "inference": {"price_per_gb": 0.01, "currency": "APT"},
        "derivatives": {"allowed": True, "attribution_required": True}
    },
    tags=["computer-vision", "imagenet", "classification", "resnet"]
)

print(f"Dataset published! ID: {dataset_id}")
# Output: 0x7a8b9c...d2e3f4 (Aptos resource address)
```

**What happens under the hood:**
1. Dataset sharded into ~10MB chunksets using Shelby's erasure coding (Clay codes)
2. Chunksets distributed to Storage Providers with ~2x replication factor
3. Blob commitments registered on Aptos blockchain
4. Rights configuration compiled into Move resource

### 3. Discover & Stream (Model Trainer)

```python
from shelbytrain import DatasetRegistry, StreamingDataset
from torch.utils.data import DataLoader

# Search registry
registry = DatasetRegistry(client)
results = registry.search(
    tags=["computer-vision", "classification"],
    min_size_gb=10,
    max_price_per_gb=0.10,
    license_type="commercial_allowed"
)

# Select dataset
dataset_info = results[0]  # ImageNet-1K-Subset-Cleaned
print(f"Found: {dataset_info.name} ({dataset_info.size_gb} GB)")
print(f"Cost to train: ~${dataset_info.estimated_training_cost(epochs=10)}")

# Stream directly into PyTorch — no local download
streaming_dataset = StreamingDataset(
    client=client,
    dataset_id=dataset_info.id,
    transform=your_transforms,
    verify_chunks=True  # Enables Merkle proof verification
)

# Standard PyTorch DataLoader
loader = DataLoader(
    streaming_dataset, 
    batch_size=64, 
    num_workers=4,
    pin_memory=True
)

# Train — every batch triggers micropayment to creator
for epoch in range(10):
    for batch in loader:
        loss = model(batch)
        loss.backward()
        # ShelbyTrain automatically logs reads & routes payments
```

### 4. Verify Provenance

```python
from shelbytrain import ProvenanceVerifier

verifier = ProvenanceVerifier(client)
lineage = verifier.trace_dataset("0x7a8b9c...d2e3f4")

print(lineage.tree())
# Dataset: ImageNet-1K-Subset-Cleaned
# ├── Source: ImageNet (ILSVRC2012)
# │   └── License: Custom research license
# ├── Transformation: Deduplication (SHA-256)
# │   └── Tool: shelbytrain-dedup v1.2
# └── Parent Datasets: None
#
# Verification: ✅ All blob commitments valid on Aptos
#               ✅ All chunks verified via Shelby sampling
#               ✅ Creator payments: 1,247 reads, $6.23 paid
```

---

## 📋 Project Structure

```
shelbytrain/
├── shelbytrain/
│   ├── __init__.py
│   ├── client/                 # Shelby protocol interface
│   │   ├── storage.py          # Blob upload/download
│   │   └── payments.py         # Micropayment handling
│   ├── dataset/
│   │   ├── uploader.py         # Shard & upload logic
│   │   ├── streaming.py        # PyTorch Dataset implementation
│   │   └── verifier.py         # Merkle proof verification
│   ├── registry/
│   │   ├── search.py           # Dataset discovery
│   │   └── metadata.py         # Schema validation
│   └── contracts/              # Move smart contracts
│       ├── dataset_registry.move
│       └── rights_manager.move
├── examples/
│   ├── upload_cifar10.py
│   ├── train_resnet50.py
│   └── verify_lineage.py
├── tests/
└── README.md
```

---

## 🔧 Technical Specifications

### Dataset Chunking Strategy
| Parameter | Value | Rationale |
|-----------|-------|-----------|
| **Blob Size** | 1-10 GB | Matches Shelby's optimal erasure coding block |
| **Chunkset Size** | ~10 MB | Network-efficient for fiber-optic transmission |
| **Chunk Size** | ~1 MB | Balance of parallelism vs. overhead |
| **Sample Size** | ~1 KB | Minimum verifiable unit for sampling audits |

### Payment Economics
Based on Shelby's target pricing :

| Operation | Cost | Frequency |
|-----------|------|-----------|
| **Write (1 GB)** | ~$0.01 | Once (dataset upload) |
| **Read (1 GB)** | ~$0.014 | Per training epoch |
| **Verification sample** | ~$0.001 | Per 1000 chunks |
| **On-chain commit** | ~$0.000005 | Per blob (Aptos gas) |

*Example: Training ResNet-50 on 150 GB ImageNet for 90 epochs*
- **Storage cost**: $1.50 (one-time)
- **Training cost**: $189.00 (150 GB × 90 epochs × $0.014)
- **Creator revenue**: ~$170 (after 10% protocol fee)

### Performance Targets
| Metric | Target | Comparison |
|--------|--------|------------|
| **Time-to-first-batch** | <2 seconds | vs. 5-10 min download (S3) |
| **Sustained throughput** | 1 GB/s | Matches NVMe SSD streaming |
| **Verification overhead** | <5% | Merkle proof checking |
| **Payment latency** | <500ms | Off-chain channels, batched settlement |

---

## 🛠️ Development Setup

### Using GitHub Codespaces (Recommended)
```bash
# Pre-configured 32-core environment
# Includes: Python 3.11, Aptos CLI, Shelby SDK (devnet access required)

# Start Codespace from this repo
# Shelby SDK will be mounted at /usr/local/lib/shelby-sdk
```

### Local GPU Burst Training
```bash
# For actual model training, rent GPU instances
# We recommend RunPod or Lambda for spot pricing

# 1. Launch GPU pod with ShelbyTrain pre-installed
runpodctl create pod \
  --gpu "RTX A6000" \
  --image "shelbytrain/pytorch:latest" \
  --env SHELBY_API_KEY="sk_..."

# 2. Train with streaming — weights sync back to Shelby on checkpoint
python train.py --dataset 0x7a8b9c...d2e3f4 --epochs 90

# 3. Terminate immediately after training to minimize costs
runpodctl stop pod $POD_ID
```

---

## 📡 Smart Contract Architecture (Move)

### Dataset Registry
```move
module shelbytrain::dataset_registry {
    struct Dataset has key {
        id: address,
        creator: address,
        blob_commitments: vector<vector<u8>>,  // Shelby blob IDs
        metadata: DatasetMetadata,
        rights: RightsConfiguration,
        created_at: u64,
    }
    
    public fun publish_dataset(
        creator: &signer,
        blob_commitments: vector<vector<u8>>,
        metadata: DatasetMetadata,
        rights: RightsConfiguration
    ): address;
    
    public fun log_read(
        dataset_id: address,
        reader: address,
        bytes_read: u64,
        payment: Coin<APT>
    ): bool;
}
```

### Rights Manager
```move
module shelbytrain::rights_manager {
    enum LicenseType {
        CC0,           // Public domain
        CCBY,          // Attribution
        CCBYSA,        // Share-alike
        CCBYNC,        // Non-commercial
        Commercial,    // Paid license
        Custom,        // Bespoke terms
    }
    
    struct RightsConfiguration has store {
        training_price_per_gb: u64,
        inference_price_per_gb: u64,
        license_type: LicenseType,
        attribution_required: bool,
        derivative_allowed: bool,
    }
    
    public fun check_access(
        dataset_id: address,
        intended_use: u8,  // 0=training, 1=inference
        payment: Coin<APT>
    ): bool;
}
```

---

## 🗺️ Roadmap

### Phase 1: Foundation (Pre-Devnet)
- [x] Core SDK architecture
- [x] Move contract framework
- [ ] Simulated Shelby backend for testing
- [ ] CIFAR-10/100 integration examples

### Phase 2: Devnet Integration (Q4 2025)
- [ ] Shelby SDK integration (actual erasure coding)
- [ ] End-to-end upload → train → verify flow
- [ ] Performance benchmarking vs. S3
- [ ] 3 pilot datasets (computer vision, NLP, audio)

### Phase 3: Testnet & Pilot (2026)
- [ ] Multi-node storage provider testing
- [ ] Real micropayment flow with testnet APT
- [ ] Integration with HuggingFace Datasets
- [ ] Dataset marketplace UI

### Phase 4: Mainnet
- [ ] Production rights management
- [ ] Enterprise SLA guarantees
- [ ] Derived dataset lineage tracking
- [ ] Decentralized model registry integration

---

## 🤝 Contributing

We welcome contributors, especially:
- **ML Engineers**: Optimize streaming performance for different model architectures
- **Move Developers**: Enhance rights management and payment logic
- **Data Curators**: Publish high-quality, licensed datasets

See [CONTRIBUTING.md](./CONTRIBUTING.md) for Early Access Program guidelines.

---

## 📄 License

Apache 2.0 — see [LICENSE](./LICENSE)

---

## 🙏 Acknowledgments

Built with support from:
- **Aptos Labs** — blockchain infrastructure and Move VM
- **Jump Crypto** — protocol design and cryptoeconomic modeling
- **Shelby Protocol Team** — decentralized storage architecture

---

*Built for the Shelby Early Access Program. Protocol documentation: [shelby.xyz](https://shelby.xyz) · Whitepaper: [arXiv:2506.xxxxx](https://arxiv.org)*
