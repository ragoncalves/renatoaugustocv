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
  --virtual-time-budget=20000 \
  --print-to-pdf="curriculo-renato-augusto-goncalves.pdf" \
  "file:///C:/Users/<usuario>/Documents/GitHub/renatoaugustocv/index.html"
```

O caminho do `--print-to-pdf` e da URL precisam ser absolutos.

**`--virtual-time-budget` não é opcional.** Sem ele o Chrome imprime antes de terminar de baixar
as fontes do Google Fonts, e o PDF sai em Arial/Times em vez de Outfit e Crimson Pro. O sintoma é
silencioso: o arquivo é gerado normalmente, só está com as fontes erradas.

Para conferir quais fontes entraram no PDF (requer `pymupdf`):

```bash
python -c "import fitz; d=fitz.open('curriculo-renato-augusto-goncalves.pdf'); print(d.page_count,'pagina(s)'); [print(f[3] or '(Type3)', f[2]) for f in d[0].get_fonts(full=True)]"
```

As fontes saem como `Type3` porque o Google Fonts serve fontes variáveis, que o Chrome converte
em contornos ao imprimir. O texto continua selecionável e extraível por sistemas de triagem (ATS)
— verificado com `get_text()`, inclusive os acentos.

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
