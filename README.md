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
