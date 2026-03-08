# IA locale — Ollama + Open WebUI + Docker

Stack d'IA entièrement locale sur EndeavourOS (RTX 3050 Mobile, 4 Go VRAM), accessible depuis mon téléphone via Tailscale.
L'objectif était de faire tourner des modèles de langage en local, sans dépendre d'un service cloud, avec une interface web propre et un accès distant. Plusieurs modèles testés, de 1B à 11B paramètres.

---

## Architecture

| Composant | Rôle |
|---|---|
| **Ollama** | Backend — chargement et inférence des modèles |
| **Open WebUI** | Frontend — interface web type ChatGPT, en local |
| **Docker** | Conteneurisation d'Open WebUI |
| **Tailscale** | Accès distant sécurisé depuis le téléphone |

```
[Samsung Z Fold 6]
   Tailscale (100.x.y.z:8080)
          │
          ▼
[Open WebUI — Docker]  ──── http://127.0.0.1:11434 ────►  [Ollama]
   interface web                                           inférence GPU/CPU
                                                          RTX 3050 (4 GB VRAM)
```

---

## Installation

### Ollama

Installé via AUR avec le support CUDA pour utiliser le GPU NVIDIA en priorité :

```bash
yay -S ollama-cuda
```

> `ollama-cuda` permet à Ollama d'utiliser le GPU (NVIDIA CUDA) en priorité pour l'inférence.
> Les modèles trop lourds pour la VRAM basculent automatiquement sur le CPU.

Configuré pour écouter sur toutes les interfaces (nécessaire pour Docker et Tailscale) :

```bash
# /etc/systemd/system/ollama.service.d/override.conf
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
```

```bash
systemctl enable --now ollama
```

### Open WebUI (Docker)

```bash
sudo docker run -d \
  --network=host \
  --gpus all \
  -v open-webui:/app/backend/data \
  --name open-webui \
  ghcr.io/open-webui/open-webui:main
```

Interface accessible sur `http://localhost:8080`, connexion vers Ollama sur `http://127.0.0.1:11434`.

### Tailscale

Installé sur le PC et le téléphone — permet d'accéder à Open WebUI depuis n'importe où
via l'IP Tailscale du PC, sans exposer de port sur internet.

---

## Ce que j'ai appris

- Comment fonctionne l'inférence locale — différence entre backend (Ollama) et frontend (Open WebUI)
- Les bases de Docker : conteneurisation, volumes, mode `--network=host`, accès GPU
- La contrainte VRAM : pourquoi la taille d'un modèle impacte directement les performances
- Tailscale comme VPN mesh simple pour exposer un service local à distance sans ouvrir de ports
- Les modèles multimodaux (vision) — différence entre un LLM texte et un modèle capable d'analyser des images
