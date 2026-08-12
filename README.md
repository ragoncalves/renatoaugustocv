# Currículo — Renato Augusto Gonçalves

Currículo em página única, publicado via GitHub Pages.

- `index.html` / `style.css` — a página.
- `curriculo-renato-augusto-goncalves.pdf` — o arquivo que o botão "Baixar PDF" entrega.
- `puc.pdf`, `una.pdf`, `cc50.pdf` — diplomas e certificados linkados na sidebar.

## Regerar o PDF (obrigatório após editar o currículo)

O botão "Baixar PDF" serve um arquivo estático. Ele **não** se atualiza sozinho quando você
edita `index.html` ou `style.css` — sem regerar, o download fica diferente da página.

No Windows, com o Chrome instalado:

```bash
"/c/Program Files/Google/Chrome/Application/chrome.exe" --headless --disable-gpu \
  --no-pdf-header-footer \
  --print-to-pdf="curriculo-renato-augusto-goncalves.pdf" \
  "file:///C:/Users/<usuario>/Documents/GitHub/renatoaugustocv/index.html"
```

O caminho do `--print-to-pdf` e da URL precisam ser absolutos.

## O layout cabe em exatamente 1 página A4

O bloco `@media print` do `style.css` está calibrado para isso: o conteúdo mede ~1044px contra
os 1123px de um A4 com margem zero, ou seja, ~79px de folga.

Ao adicionar texto (uma nova experiência, um parágrafo mais longo), confira se ainda cabe:

```bash
# gere o PDF e conte as páginas — precisa dar 1
```

Se passar de 1 página, aperte primeiro os espaçamentos do bloco `@media print`
(`gap`, `padding`, `padding-bottom` de `.timeline-item`) antes de reduzir fonte.
Não deixe o corpo de texto abaixo de 9.5px.

O `@page` usa `margin: 0` e a barra lateral depende de `print-color-adjust: exact` para imprimir
com o fundo escuro.
