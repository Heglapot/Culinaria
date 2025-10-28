# Culinaria
#!/usr/bin/env python3
"""
fix_template.py

Organiza arquivos de um template HTML para GitHub Pages. Move CSS, JS,
imagens e fontes para as pastas css/, js/, img/ e fonts/, respectivamente,
e ajusta os caminhos no arquivo index.html.

Uso:
1. Copie este script para a pasta do template (onde está index.html).
2. No terminal, acesse essa pasta e execute:
   python3 fix_template.py
3. Verifique se o index.html abre corretamente. Faça commit e push
   dos arquivos organizados para o GitHub.
"""

import os
import re
import shutil
from pathlib import Path

def ensure_dirs(root: Path) -> None:
    """Cria as pastas css/, js/, img/, fonts/ se não existirem."""
    for d in ["css", "js", "img", "fonts"]:
        (root / d).mkdir(exist_ok=True)

def classify_and_move_files(root: Path) -> None:
    """Move arquivos de acordo com a extensão."""
    css_exts  = {".css"}
    js_exts   = {".js"}
    img_exts  = {".jpg", ".jpeg", ".png", ".gif", ".webp", ".svg"}
    font_exts = {".eot", ".ttf", ".woff", ".woff2", ".otf"}

    for item in root.iterdir():
        # Ignorar pastas já criadas e arquivos ocultos
        if item.name.startswith(".") or item.name.lower() in {"css", "js", "img", "fonts"}:
            continue
        if item.is_file():
            ext = item.suffix.lower()
            if ext in css_exts:
                dst = root / "css" / item.name
            elif ext in js_exts:
                dst = root / "js" / item.name
            elif ext in img_exts:
                dst = root / "img" / item.name
            elif ext in font_exts:
                dst = root / "fonts" / item.name
            else:
                continue  # outros arquivos permanecem
            if not dst.exists():
                print(f"Movendo {item.name} → {dst}")
                shutil.move(str(item), str(dst))

def update_html_paths(html_path: Path) -> None:
    """Ajusta caminhos de CSS, JS e imagens em index.html."""
    content = html_path.read_text(encoding="utf-8")

    def replace_links(match: re.Match) -> str:
        prefix, filename, suffix = match.groups()
        return f"{prefix}css/{filename}{suffix}"

    def replace_scripts(match: re.Match) -> str:
        prefix, filename, suffix = match.groups()
        return f"{prefix}js/{filename}{suffix}"

    def replace_imgs(match: re.Match) -> str:
        prefix, filename, ext, suffix = match.groups()
        return f"{prefix}img/{filename}.{ext}{suffix}"

    # Ajustar href="arquivo.css" → href="css/arquivo.css"
    content = re.sub(r'(href=")([^/#\s]+\.css)(")', replace_links, content, flags=re.IGNORECASE)
    # Ajustar src="arquivo.js" → src="js/arquivo.js"
    content = re.sub(r'(src=")([^/#\s]+\.js)(")', replace_scripts, content, flags=re.IGNORECASE)
    # Ajustar src="imagem.jpg" → src="img/imagem.jpg"
    content = re.sub(r'(src=")([^/\s]+?)\.(jpe?g|png|gif|svg|webp)(")', replace_imgs, content, flags=re.IGNORECASE)

    # Corrigir caminho do Font Awesome se necessário
    content = content.replace("fonts/font-awesome/css/font-awesome.css", "css/font-awesome.css")

    html_path.write_text(content, encoding="utf-8")
    print(f"Caminhos atualizados em {html_path}")

def main() -> None:
    root = Path(__file__).resolve().parent
    index_html = root / "index.html"
    if not index_html.exists():
        raise FileNotFoundError("index.html não encontrado na pasta atual.")

    ensure_dirs(root)
    classify_and_move_files(root)
    update_html_paths(index_html)
    print("\nProcesso concluído. Faça commit e push dos arquivos organizados para o GitHub.")

if __name__ == "__main__":
    main()
