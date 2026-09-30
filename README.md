# LateralMovementTracer
Lateral Movement Tracer
# Lateral Movement Tracer (LMT)

Offline Python library + local batch API that reconstructs **ranked lateral movement paths** from a foothold to sensitive assets.

Telemetry is parsed into a documented relational model (ER → 3NF dims + `fact_event` in SQLite). A weighted attack graph is **projected** from that store; Yen’s k-shortest paths ranks routes. Every hop cites `fact_event` IDs.

## Quick start

```bash
cd "C:\Kal new\Personal Projects\lmt"
python -m pip install -r requirements.txt
python -m pip install -e .
python cli.py --seed LAPTOP-ALICE -k 5
```

### API

```bash
uvicorn api.app:app --reload --app-dir .
```

```bash
curl -s http://127.0.0.1:8000/health
curl -s -X POST http://127.0.0.1:8000/analyze -H "Content-Type: application/json" -d "{\"seed_host\":\"LAPTOP-ALICE\",\"k\":5}"
```

## Pipeline

1. Deterministic parsers (`windows_security`, `sysmon`, `vpn`, `db_audit`) → CSE  
2. Load-time entity resolution → SQLite dims + facts ([`schema.sql`](schema.sql), [`docs/data_model.md`](docs/data_model.md))  
3. Deterministic edge costs (lower = more suspicious)  
4. NetworkX MultiDiGraph + Yen (`shortest_simple_paths`) with **time-ordered** hops  
5. JSON report / FastAPI (`/health`, `/analyze`, `/jobs`)

## Sample scenario

Planted chain in [`samples/`](samples/):

`LAPTOP-ALICE → VPN-GW → HR-APP → DB-HR`

plus noise (failed RDP, file share, benign VPN).

## Linux forensic demos

Under [`demos/`](demos/): realistic Linux directory trees with auth/apache/DB logs and a planted attack path.

| Demo | Story |
|------|--------|
| `demos/01_single_host` | One host (`web-hr-01`): SSH foothold → secrets → local MySQL dump |
| `demos/02_two_hosts` | `web-01` → SSH → `db-01`; pull the second host with `pull_remote_logs.ps1` |

```bash
python cli.py --samples demos/01_single_host/lmt_ingest --seed WEB-HR-01 -k 5
python cli.py --samples demos/02_two_hosts/lmt_ingest --seed WEB-01 -k 5
```

## Edge cost polarity

Cost is the sum of time-gap, protocol/action risk, privilege jump, outcome, and identity-join strength. **Lower total cost ranks first** (stronger lateral signal).

## Tests

```bash
pytest -q
```
