# 🐍 PyVax CLI — Python-to-EVM Smart Contract Compiler

> **Write smart contracts in pure Python. Deploy to any EVM blockchain.**

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue?style=flat-square&logo=python)](https://www.python.org/)
[![EVM Compatible](https://img.shields.io/badge/EVM-Compatible-purple?style=flat-square)](https://ethereum.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)]()

---

## 📌 Overview

**PyVax CLI** is a full-stack developer toolchain that compiles standard Python classes directly into **EVM (Ethereum Virtual Machine) bytecode** — without requiring developers to learn Solidity, Vyper, or any blockchain-specific syntax.

It uses **AST (Abstract Syntax Tree) analysis** and **Python naming conventions** to automatically determine function visibility, generate ABI definitions, and produce deployment-ready smart contracts — all from regular Python code.

> 💡 Built as part of the **py0g** project by [ShahiTechnovation](https://github.com/ShahiTechnovation/py0g)

---

## 🎯 Problem Statement

Blockchain development has a steep learning curve. Developers must abandon familiar tools and learn entirely new languages (Solidity/Vyper) just to write smart contracts.

**PyVax solves this by letting Python developers:**
- Write contracts in standard Python — no new syntax
- Use any Python IDE (VSCode, PyCharm) without special extensions
- Debug with the Python debugger they already know
- Migrate existing Python classes into smart contracts with minimal changes

---

## ✨ Key Features

| Feature | Description |
|---|---|
| 🔄 **Python-to-EVM Transpiler** | Converts Python classes to EVM opcodes and bytecode |
| 🧠 **AST-Based Analysis** | Parses Python code structure without decorators or annotations |
| 📋 **Auto ABI Generation** | Generates complete ABI definitions from function signatures |
| 🔐 **Visibility Detection** | Public/private functions detected via Python naming conventions |
| ⛽ **Gas Optimization** | Configurable optimization settings for deployment |
| 🚀 **Multi-Network Deploy** | Deploys to Avalanche and other EVM-compatible networks |
| 🔒 **Encrypted Wallet** | Keystore management with secure keystore files |
| 📡 **Contract Interaction** | CLI-based interaction with deployed smart contracts |

---

## 🏗️ Architecture

```
PyVax CLI
├── cli.py                  # Typer-powered CLI — 6 commands
├── compiler.py             # Contract compiler (Solidity + Python contracts)
├── transpiler.py           # Python-to-EVM transpiler (3 main classes)
│   ├── PythonASTAnalyzer       # Parses AST → extracts state, functions, events
│   ├── EVMBytecodeGenerator    # Converts AST → EVM opcodes + bytecode
│   └── PythonContractTranspiler # Orchestrates analysis + ABI generation
├── py_contracts.py         # Python Smart Contract Framework
├── deployer.py             # Deployment engine (Avalanche networks)
└── wallet.py               # Encrypted wallet management
```

### Workflow

```
Python Contract (.py)
        │
        ▼
  PythonASTAnalyzer
  (Extract: state vars, public/private functions, events)
        │
        ▼
  EVMBytecodeGenerator
  (Generate: function selectors, dispatcher, init + runtime bytecode)
        │
        ├──────────────────────┐
        ▼                      ▼
  Deployment Bytecode    Runtime Bytecode
  (deploys contract)     (executes functions)
        │
        ▼
  ABI Definition
  (how to interact with deployed contract)
        │
        ▼
  Deployed on EVM Network ✅
```

---

## 🔬 How It Works — Technical Deep Dive

### 1. Function Visibility (No Decorators Needed)

PyVax uses **Python naming conventions** — the same ones every Python developer already follows:

```python
class MyToken:
    def __init__(self, owner: str):        # Constructor — special handling
        self.owner = owner

    def transfer(self, to: str, amount: int):    # PUBLIC — no underscore
        return self._validate_transfer(to, amount)

    def balance_of(self, account: str):          # PUBLIC — no underscore
        return self.balances.get(account, 0)

    def _validate_transfer(self, to, amount):    # PRIVATE — starts with _
        return amount > 0 and to != ""
```

### 2. EVM Bytecode Generation Pipeline

**Step 1 — Function Selector Generation** (like Solidity's 4-byte selector):
```python
# transfer(uint256,uint256) → 0xa9059cbb
signature = f"{name}({','.join(param_types)})"
hash_bytes = hashlib.sha256(signature.encode()).digest()
selector = int.from_bytes(hash_bytes[:4], 'big')
```

**Step 2 — EVM Function Dispatcher**:
```
CALLDATALOAD → extract 4-byte selector
For each public function:
    DUP1 → PUSH4 <selector> → EQ → JUMPI to implementation
Default: REVERT
```

**Step 3 — State Mutability Detection** (AST analysis, no `@view` decorator needed):
```python
# Checks for self.x = ... assignments in AST → nonpayable
# Detects read-only patterns (get_, balance_of, etc.) → view
```

### 3. ABI Generation

The transpiler auto-generates a complete ABI compatible with Web3.js, ethers.js, and all standard EVM tooling — no manual ABI writing required.

---

## 🖥️ CLI Commands

```bash
# Create a new project with sample contracts
pyvax init my_project

# Compile a Python or Solidity contract
pyvax compile contracts/MyToken.py

# Deploy to Avalanche Fuji (testnet) or Mainnet
pyvax deploy contracts/MyToken.py --network fuji

# Manage encrypted wallets
pyvax wallet --create

# Interact with a deployed contract
pyvax interact --address 0xABC... --function transfer

# Get info about a deployed contract
pyvax info --address 0xABC...
```

---

## 📦 Installation

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/pyvax-cli.git
cd pyvax-cli

# Install dependencies
pip install -r requirements.txt

# Run the CLI
python -m pyvax --help
```

**Dependencies:**
- `typer` — CLI framework
- `rich` — Beautiful terminal output
- `web3.py` — Ethereum interaction
- `py-solc-x` — Solidity compiler (for `.sol` contracts)

---

## 📝 Example: Write a Token Contract in Python

```python
# contracts/MyToken.py

class MyToken:
    """A simple ERC-20-style token written in pure Python."""

    def __init__(self, owner: str):
        self.owner = owner
        self.total_supply = 1_000_000
        self.balances = {owner: 1_000_000}

    def transfer(self, to: str, amount: int) -> bool:
        sender = self.msg_sender
        if self.balances.get(sender, 0) >= amount:
            self.balances[sender] -= amount
            self.balances[to] = self.balances.get(to, 0) + amount
            return True
        return False

    def balance_of(self, account: str) -> int:
        return self.balances.get(account, 0)

    def _check_owner(self) -> bool:          # PRIVATE — not exposed to ABI
        return self.msg_sender == self.owner
```

**Compile and deploy:**
```bash
pyvax compile contracts/MyToken.py
pyvax deploy contracts/MyToken.py --network fuji
```

---

## 🆚 PyVax vs Traditional Approaches

| | PyVax (Python) | Solidity | Vyper |
|---|---|---|---|
| **Language** | Standard Python | New language | New language |
| **IDE Support** | Any Python IDE ✅ | Remix / special plugins | Limited |
| **Debugging** | Python debugger ✅ | Hardhat / Truffle | Limited |
| **Learning Curve** | Zero (for Python devs) ✅ | High ❌ | Medium ❌ |
| **Decorator Required** | No ✅ | N/A | Yes (`@external`) |
| **ABI Generation** | Automatic ✅ | Manual / compiler | Manual / compiler |
| **Migration from Python** | Easy ✅ | Full rewrite ❌ | Full rewrite ❌ |

---

## 🗺️ Roadmap

- [x] Python-to-EVM transpiler (core engine)
- [x] AST-based visibility and mutability detection
- [x] ABI auto-generation
- [x] Avalanche network deployment
- [x] Encrypted wallet management
- [ ] Support for events and indexed logs
- [ ] ERC-20 / ERC-721 standard templates
- [ ] Hardhat/Foundry integration
- [ ] Unit test framework for Python contracts
- [ ] VS Code extension

---

## 🤝 Contributing

Contributions are welcome! Please open an issue or submit a pull request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## 👨‍💻 Author

**[Your Name]** — Third-year CS student passionate about blockchain and developer tooling.

- GitHub: [@YOUR_USERNAME](https://github.com/YOUR_USERNAME)
- LinkedIn: [Your LinkedIn](https://linkedin.com/in/yourprofile)

---

> ⭐ If you find this project useful, please consider giving it a star — it helps others discover it!
