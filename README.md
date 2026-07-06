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
sudo mkswap /swapfile
sudo swapon /swapfile

# Permanente Verankerung im System
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
