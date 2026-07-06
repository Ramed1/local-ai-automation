# 🚀 Local AI Automation (n8n + Ollama)

Dieses Repository enthält die Konfigurationen, Skripte und Workflows für eine vollständig lokale, datenschutzkonforme KI-Automatisierungs-Pipeline auf einem Linux-Server. 

Das Setup kombiniert die Flexibilität von **n8n** mit der Unabhängigkeit einer lokalen **Ollama**-Instanz, um B2B-Prozesse und Workflows intelligent zu automatisieren.

---

## 🛠️ Tech Stack & Architektur

Die gesamte Infrastruktur läuft containerisiert über **Docker Compose** und ist im internen Docker-Netzwerk isoliert.

* **n8n (Advanced AI):** Die visuelle Schaltzentrale für die Logik und Workflow-Verkettung.
* **Ollama:** Die lokale KI-Engine zur Ausführung von Open-Source-Modellen ohne Cloud-Zwang.
* **LLM:** `llama3.2` (für schnelle, präzise Textanalysen und Generierungen).

---

## 💾 Server-Optimierung (RAM & Swap Management)

Da das Laden von Large Language Models (LLMs) im Arbeitsspeicher sehr ressourcenintensiv ist, wurde das Ubuntu-System für maximale Stabilität optimiert. Um `OOM` (Out of Memory) Abstürze zu verhindern, ist eine dedizierte **20 GB Swap-Datei** auf der SSD eingerichtet:

```bash
# Erstellung und Aktivierung des erweiterten Swap-Speichers
sudo fallocate -l 20G /swapfile
sudo chmod 600 /swapfile


# Permanente Verankerung im System
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
sudo mkswap /swapfile
sudo swapon /swapfile
```

## 📦 Infrastruktur (docker-compose.yml)

Die Container kommunizieren intern direkt über den Servicenamen. Der Ollama-Container ist so konfiguriert, dass er dem n8n-Container das Modell im selben Netz zur Verfügung stellt.

```
version: '3.8'

services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    volumes:
      - ollama_data:/root/.ollama
    ports:
      - "11434:11434"
    restart: unless-stopped

  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    depends_on:
      - ollama
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
    volumes:
      - n8n_data:/home/node/.n8n
    restart: unless-stopped

volumes:
  ollama_data:
  n8n_data:
  ```

## 🤖 Modell-Initialisierung

Nach dem Start der Container wird das Modell einmalig über die CLI im Ollama-Container geladen:
```
docker exec -it ollama ollama run llama3.2
```

## 🎯 Use-Case: B2B E-Mail-Infrastruktur Kaltakquise

Der in diesem Repository hinterlegte n8n-Workflow (workflow.json) nutzt die lokale KI für ein konkretes Business-Szenario:

Eingabe / Trigger: Payload einer Domain (z. B. nach einem dig txt domain.de Scan).

Analyse: Die KI prüft die DNS-Einträge auf das Fehlen von SPF, DKIM oder DMARC.

Generierung: llama3.2 verarbeitet die technischen Schwachstellen über eine Advanced AI Chain (inkl. Window Buffer Memory für Kontext) und formatiert eine maßgeschneiderte, verkaufsstarke Kaltakquise-Mail an den Inhaber.

Ziel: Automatisierte Lead-Generierung für IT-Dienstleistungen zur Behebung von Spam- und Zustellungsproblemen bei KMUs und Online-Shops.
