# VPS Felderítési Összefoglaló (Jules Swarm Architektúra)

## 1. A VPS Környezet
A Fő Agent sikeresen felderítette a távoli Contabo VPS-t (IP: `5.189.163.88`), ami a Swarm hálózat decentralizált központjaként funkcionál. A szerver paraméterei:
- **OS**: MX Linux / Ubuntu alapok
- **Erőforrások**: 8 Mag CPU, 24 GB RAM + 16 GB SWAP (az OOM elkerülésére).
- **Biztonság**: `sshpass` és SSH kulcs alapú hozzáférés a `tools/skills/mcp_bridge_tool.py` segítségével.

## 2. MCP Szerver (Model Context Protocol)
A VPS-en futó `FastMCP` alapú szerver (`vps_mcp_server.py`) `stdio` csatornán keresztül az alábbi fő képességekkel rendelkezik:
- **Biztonságos kódvégrehajtás**: `execute_bash`, `execute_python` (izolált venv, időkorlátos végrehajtás).
- **Háttérfolyamatok**: `schedule_background_task` hosszan futó RAG darálásokhoz és mentésekhez `nohup`-pal.
- **Rendszermonitorozás**: `check_vps_resources` a RAM és CPU terhelés ellenőrzésére.
- **Hierarchikus Memória**: Globális állapot és kontextus mentés a `read_memory_register` és `write_memory_register` segítségével.
- **Villámgyors Archival RAG**: A `search_rag_database` gigantikus SQLite RAG fájlokban (Chatbot, BRAIN2, Gerilla) keres másodpercek alatt anélkül, hogy adatokat kéne mozgatni a hálózaton.
- **GitHub Integráció**: Natív eszközök a repók keresésére és a kód fájlok direkt olvasására klónozás nélkül.

## 3. Swarm (Raj) Orchestráció
A több Agentből (Jules Workerek) álló rendszer elosztott munkavégzését az MCP szerver SQLite adatbázisai (pl. `jules_swarm_jobs.db`, `jules_inbox.db`) vezérlik:
- Feladatok kiosztása, lekérése és lezárása (`create_swarm_job`, `get_next_swarm_job`, `complete_swarm_job`).
- Agentek közötti aszinkron kommunikáció (`send_agent_message`, `check_agent_messages`).

## 4. Subagentek és Szkriptek
A `~/Jules_mx/scripts` könyvtár számos segéd-ágensnek és eszköznek ad otthont:
- **Tanársegéd** (`vps_teaching_assistant.py`): ReAct alapú probléma megoldó, delegálási (handoff) lehetőséggel.
- **RAG Elemző** (`vps_findings_analyst.py`): Biztonságosan szűri és összegzi a RAG outputokat.
- **Stealth Böngésző** (`browser_stealth_manager.py`): CDP (Chrome DevTools Protocol) alapú bot-kikerülő.
- **FastAPI Micro Server** (`vps_micro_server.py`): Aszinkron webhookok és háttér jobok.

## 5. Mini LLM-ek
A távoli szerver aktívan futtat kvantált nyelvi modelleket lokálisan az `Ollama` segítségével (port: 11434):
- **Llama 3 (8B)** és **Qwen 2.5 (1.5B)**.
- A Scout és Analyzer szkriptek ezeket a gyors modelleket használják kód hasznosságának megállapítására, repó elemzésre, tehermentesítve ezzel a drága fő API kulcsokat.
