# rescile-adapters

Provider-specific adapters for the [Rescile UCS](https://github.com/rescile/ucs) orchestration layer.

This repository contains the execution implementations that bridge Rescile's intent-based orchestration with real cloud provider APIs. Each adapter directory corresponds to a supported provider and implements the domain contracts defined by the UCS domain controllers.

---

## How it fits into Rescile UCS

Rescile UCS operates in three layers:

```
Blueprint Composer        ← interprets intent, composes involved domains
    │
Domain Controllers        ← resolve configuration from the dependency graph
    │
Adapters (this repo)      ← execute provider-specific API calls
```

Domain controllers are provider-agnostic. They call into this adapter library to perform the actual operations — creating VPCs, resolving DNS, registering endpoints, and so on — against a specific target environment.

---

## Repository structure

```
rescile-adapters/
├── aws/
│   ├── vpc_builder.py
│   ├── dns_resolver.py
│   ├── endpoint_builder.py
│   └── ...
├── azure/
│   ├── vnet_builder.py
│   ├── dns_resolver.py
│   └── ...
├── gcp/
│   └── ...
├── tests/
│   ├── aws/
│   ├── azure/
│   └── ...
└── README.md
```

Each provider directory is a self-contained Python package. Files are named after the domain operation they implement, following the `<domain>_<role>.py` convention (e.g. `vpc_builder.py`, `dns_resolver.py`).

---

## Requirements

- Python 3.11+
- Rescile UCS core (`pip install rescile-ucs`)
- Provider-specific SDKs (e.g. `boto3` for AWS, `azure-mgmt-*` for Azure)

Install all dependencies for a specific provider:

```bash
pip install -r aws/requirements.txt
```

---

## Usage

Adapters are not typically called directly. They are resolved and invoked by UCS domain controllers at execution time based on the target provider configured in your blueprint.

For local development and testing, you can invoke an adapter function directly:

```python
from rescile_adapters.aws.vpc_builder import build_vpc

result = build_vpc(config={
    "cidr": "10.0.0.0/16",
    "region": "eu-central-1",
    "name": "my-vpc"
})
```

Refer to each adapter's docstring for its expected input schema and return contract.

---

## Contributing

Contributions are welcome — whether you're adding support for a new provider, extending an existing one, or improving test coverage.

### Adding a new provider

1. Create a new directory named after the provider (e.g. `hcloud/`, `exoscale/`).
2. Implement the relevant domain operations as individual Python modules following the `<domain>_<role>.py` naming convention.
3. Each function must accept a `config: dict` argument and return a standardised result dict (see [Adapter Contract](docs/adapter-contract.md)).
4. Add a `requirements.txt` for any provider SDK dependencies.
5. Add tests under `tests/<provider>/`.

### Adding to an existing provider

1. Follow the existing naming conventions in that provider's directory.
2. Cover your implementation with unit tests and at least one integration test.
3. Document the expected `config` schema in the module docstring.

### Development setup

```bash
git clone https://github.com/rescile/rescile-adapters.git
cd rescile-adapters
python -m venv .venv && source .venv/bin/activate
pip install -r requirements-dev.txt
pytest tests/
```

### Pull request checklist

- [ ] New module follows `<domain>_<role>.py` naming
- [ ] Function signature matches the adapter contract
- [ ] Unit tests added under `tests/<provider>/`
- [ ] Docstring documents `config` schema and return value
- [ ] No provider credentials committed

---

## License

Apache 2.0 — see [LICENSE](LICENSE).

---

## Related

- [rescile/ucs](https://github.com/rescile/ucs) — UCS core orchestration engine
- [Rescile documentation](https://docs.rescile.io)
