# Doomflymerc

   sudo apt update
   sudo apt install -y python3.11 python3.11-venv build-essential clang git nodejs npm

   2
   ind på filmappen
   cd ~/doomfly

   3
      python3.11 -m venv .venv-neural
   source .venv-neural/bin/activate
   python -m pip install --upgrade pip
   python -m pip install -r requirements-neural.txt -r doom/requirements.txt --build-constraint neural-build-constraints.txt

   4 
      python -m doom.connectome malecns_v1
   python -m doom.prepare
   python -m doom.audit_data
   python -m doom.build_kernel


   download script
   notepad download_data.py
   from pathlib import Path
import hashlib, json, urllib.request
name = 'malecns_v1'
registry = json.loads(Path('doom/datasets.json').read_text())['datasets'][name]
locked = json.loads(Path(f'data-provenance/{name}/source.lock.json').read_text())
root = Path('connectome_data') / name
root.mkdir(parents=True, exist_ok=True)
for filename, url in registry['files'].items():
    target = root / filename
    if not target.exists():
        print('Downloading', filename, '...')
        partial = target.with_suffix('.download')
        urllib.request.urlretrieve(url, partial)
        partial.replace(target)
    with target.open('rb') as stream:
        digest = hashlib.file_digest(stream, 'sha256').hexdigest()
    if digest != locked[filename]['sha256']:
        raise RuntimeError(f'Source checksum mismatch: {filename}')
    print('OK', filename)
(root / 'source.lock.json').write_text(json.dumps(locked, indent=2) + '\n')
print('Done')









CLAUDE GUIDE TIL INSTALL
__
Har du tænkt på, at du skal stå i den rigtige undermappe på Ubuntu? På grund af den dobbelte mappestruktur ligger koden tre niveauer nede, og alle README'ens kommandoer regner med, at du står der.

**1. Systempakker og repo**
```sh
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install git python3.11 python3.11-venv python3.11-dev build-essential
git clone https://github.com/Mester1Myhy/Doomflymerc.git
cd Doomflymerc/doomfly-main-1/doomfly-main
```

**2. Python-miljø**
```sh
python3.11 -m venv .venv-neural
source .venv-neural/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements-neural.txt -r doom/requirements.txt \
  --build-constraint neural-build-constraints.txt
```

**3. Hent connectome-data**
[Certain] Kopiér hele `python - <<'PY' ... PY`-blokken fra README'en ind i terminalen. Den henter de tre `.feather`-filer til `connectome_data/malecns_v1/` og stopper, hvis en checksum ikke passer.

**4. Importér, forbered og byg**
```sh
python -m doom.connectome malecns_v1
python -m doom.prepare
python -m doom.audit_data
python -m doom.build_kernel
```

**5. Start serveren**
```sh
python -m doom.server --model experimental-v6 --learning --port 8766 \
  --audit-dir outputs/doom/local-training \
  --checkpoint-dir outputs/doom/local-training/checkpoints \
  --checkpoint-seconds 300 --resume
```

[Likely] Kør trin 5 inde i `tmux`, så serveren ikke stopper, når du lukker SSH-forbindelsen (`sudo apt install tmux`, derefter `tmux new -s doomfly`).

[Certain] Hvis du åbner en ny terminal senere, skal du køre `source .venv-neural/bin/activate` igen, før noget af det virker.
