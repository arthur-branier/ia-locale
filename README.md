[README-ia-locale.md](https://github.com/user-attachments/files/25813760/README-ia-locale.md)
# IA locale — Ollama + Open WebUI + Docker

Stack d'IA entièrement locale sur EndeavourOS, accessible depuis mon téléphone via Tailscale.
L'objectif était de faire tourner des modèles de langage en local, sans dépendre d'un service cloud,
avec une interface web propre et un accès distant.

---

## Architecture

| Composant | Rôle |
|---|---|
| **Ollama** | Backend — chargement et inférence des modèles LLM |
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

## Matériel

- **PC** : ASUS TUF Gaming A17 — EndeavourOS
- **GPU** : NVIDIA RTX 3050 Mobile (4 Go VRAM)
- **Accès distant** : Samsung Galaxy Z Fold 6 via Tailscale

---

## Installation

### Ollama

Installé nativement via pacman :

```bash
sudo pacman -S ollama
```

Configuré pour écouter sur toutes les interfaces (nécessaire pour Docker et Tailscale) :

```bash
# /etc/systemd/system/ollama.service.d/override.conf
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
```

Démarrage du service :

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

Interface accessible sur `http://localhost:8080`.
Connexion vers Ollama configurée sur `http://127.0.0.1:11434`.

### Tailscale

Installé sur le PC et le téléphone — permet d'accéder à Open WebUI depuis n'importe où
via l'IP Tailscale du PC (`100.x.y.z:8080`), sans exposer de port sur internet.

---

## Modèles utilisés

| Modèle | Taille | Exécution |
|---|---|---|
| `llama3.2:3b` | ~2 Go | ✅ GPU (RTX 3050) |
| `qwen2.5:3b` | ~2 Go | ✅ GPU (RTX 3050) |
| `qwen2.5:14b` | >4 Go | ⚠️ CPU/RAM (lent) |

> Avec 4 Go de VRAM, les modèles jusqu'à ~3-4B paramètres tournent bien sur le GPU.
> Au-delà, le modèle bascule sur le CPU, ce qui est nettement plus lent.

---

## Accès

| Point d'accès | URL |
|---|---|
| Local | `http://localhost:8080` |
| Distant (Tailscale) | `http://100.x.y.z:8080` |
| API Ollama | `http://localhost:11434` |

---

## Ce que j'ai appris

- Comment fonctionne l'inférence locale — la différence entre un backend (Ollama) et un frontend (Open WebUI)
- Les bases de Docker : conteneurisation, volumes, mode `--network=host`, accès GPU
- La contrainte VRAM : pourquoi la taille d'un modèle impacte directement les performances
- Tailscale comme solution VPN mesh simple pour exposer un service local à distance sans ouvrir de ports
- La quantization GGUF — comment réduire la taille d'un modèle pour le faire tourner sur du matériel grand public

---

## État actuel

| Fonctionnalité | État |
|---|---|
| Ollama | ✅ Fonctionnel |
| Open WebUI | ✅ Fonctionnel |
| Accès GPU (modèles ≤3B) | ✅ Fonctionnel |
| Accès distant Tailscale | ✅ Fonctionnel |
| Modèles >4B | ⚠️ CPU uniquement (lent) |
