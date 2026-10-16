# Your machine

**This file is yours, not the kit's.** It holds the facts about THIS install: where ComfyUI lives, what GPUs
are in the box, how to start the server when it is down. The installer creates it once and then never touches
it again, so a `git pull` plus a reinstall cannot wipe what the bootstrap learned. Nothing here ships back to
the repo.

**If the values below are still angle brackets, the bootstrap has not run.** Run it on the first ComfyUI task:
call the MCP `health_check` (or `comfy_client.alive()` plus `GET /system_stats` and `/object_info`) and rewrite
this file with the real values. See `BOOTSTRAP.md`. Do not assume another machine matches an example.

- **ComfyUI**: source install (bare-metal venv, torch 2.14+rocm10.1), core path `/root/ComfyUI-latest` (on the REMOTE ubuntu box, not this Windows machine), API at **`http://100.97.153.80:8011`** (Tailscale; LAN IP 172.16.76.51 NOT routable from Windows). Always `export COMFY_HOST=100.97.153.80:8011` before using `comfy_client.py`. Check: `GET /system_stats` -> 200.
- **GPUs**: 1x AMD Strix Halo gfx1151 (rocm, device cuda:0), 128G unified (RAM 131G, GTT ~96G). Remote — VRAM coordination still applies, service `comfyui-latest.service` runs with pytorch+CK attention (2026-10-16).
- **Models installed** (query live, never hardcode): `GET /object_info/UNETLoader`,
  `/object_info/CheckpointLoaderSimple`, `/object_info/CLIPLoader`, `/object_info/VAELoader`.
  Model dirs on ubuntu: `/root/comfy-models/` (extra_model_paths legacy volume; CLIP encoders live in `clip/` NOT `text_encoders/`).
- **Shared models dir / extra_model_paths**: `/root/ComfyUI-latest/extra_model_paths.yaml` → base `/root/comfy-models/`.
- **GUI workflows folder** (the bridge back to the app): `/root/comfy-user/workflows/` (host) = ComfyUI user dir. WebUI reachable from Windows browser: http://100.97.153.80:8011 (canvas-rendered; params via API not DOM).
- **Template library**: `<path to the workflow_templates clone>`. This one is load-bearing and easy to skip:
  `TASKS.md` opens every job by reading `templates/_quick_index.json` under this path, so leaving it blank
  makes step 1 of every recipe dead. The installer's default target is
  `~/comfyui-agent-kit-data/workflow_templates`; if you cloned it somewhere else, write that path here.
  → **CLONED 2026-10-16**: `~/comfyui-agent-kit-data/workflow_templates` (depth-1; example input/output/thumbnail assets trimmed to save 3.7G; `templates/_quick_index.json` generated, 807 entries incl. LTX-2.3 i2v/ia2v).
- **Launch command**: Windows cannot launch it directly — it's a systemd service on ubuntu: `ssh root@100.97.153.80 'systemctl restart comfyui-latest.service'`. Unit file has the env (HSA_USE_SVM=0 etc.) + `--use-pytorch-cross-attention`.
- **Known local quirks** (2026-10-16): ①same seed+prompt = output cache, always change seed for timing; ②`--use-flash-attention` flag crashes the service (no flash-attn pkg, gfx1151 unsupported — see amd-video HANDOFF); ③Windows→ubuntu only via Tailscale IP; ④output cache means re-runs return in 0s. Record more as you hit them.
