# AVIF + WebP on Upload

🇺🇸 [Read in English](README.md) · 🇧🇷 Português (você está aqui)

Plugin leve para WordPress que converte automaticamente as imagens enviadas para **AVIF**, gera espelhos em **WebP** para todos os tamanhos e remove os arquivos JPG/PNG originais, para que a biblioteca de mídia ocupe menos disco e o site carregue mais rápido.

> **Versão:** 2.4 · **Requer WordPress:** 6.5+ · **Requer PHP:** 7.4+ · **Licença:** GPL-2.0-or-later

---

## Por que usar

Imagens costumam ser a maior parte do peso de uma página. O AVIF entrega a mesma qualidade visual com arquivos muito menores que JPG e PNG, e o WebP serve como alternativa para navegadores sem suporte a AVIF.

Resultados reais de testes com uploads comuns (qualidade AVIF 60, WebP 75):

| Imagem enviada | Original | AVIF principal | WebP principal |
|---|---|---|---|
| Foto 2048 × 2048 (JPG) | 465 KB | 160 KB (-66%) | 199 KB (-57%) |
| Captura de tela 3840 × 2160 (JPG) | 447 KB | 32 KB (-93%) | 48 KB (-89%) |
| Foto de celular 2304 × 4096 (JPG) | 2,1 MB | 62 KB (-97%) | 80 KB (-96%) |

Os ganhos variam conforme o conteúdo da imagem. Capturas de tela e imagens com áreas lisas comprimem muito mais que fotos com muito detalhe.

---

## O que o plugin faz

1. **Converte para AVIF** o arquivo principal e todas as miniaturas geradas pelo WordPress, inclusive a versão `-scaled` de imagens maiores que 2560 px.
2. **Gera espelhos em WebP** para o arquivo principal e para cada tamanho, ao lado dos AVIF, com o mesmo nome (`foto-300x200.avif` e `foto-300x200.webp`).
3. **Remove os originais** (JPG, PNG, WebP e o `original_image` antes do `-scaled`), mas só depois de validar que o AVIF principal e todos os WebP existem e abrem como imagem de verdade.
4. **Preserva a orientação** de fotos de celular (EXIF) nos espelhos WebP.
5. **Não toca em imagens animadas** (WebP animado, AVIF animado e APNG). GIF e SVG também passam direto.
6. **Apaga tudo junto**: ao excluir uma mídia no WordPress, os espelhos WebP também são removidos do disco.

### Segurança do processo

- O AVIF principal é gravado em um arquivo temporário e só é movido para o destino se for válido.
- Se qualquer etapa falhar, **os originais são mantidos** e o motivo vai para o `debug.log` do PHP.
- Se o servidor não tiver suporte a AVIF, o plugin não faz nada.
- Se o servidor não tiver suporte a WebP, o plugin continua e apenas pula os espelhos.

---

## Requisitos

- WordPress **6.5 ou superior** (suporte a AVIF no núcleo).
- PHP **7.4 ou superior** (testado em PHP 8.4).
- Servidor com suporte a **AVIF** no editor de imagens: Imagick com libheif/libavif, ou GD com `imageavif()`.
- Para os espelhos WebP, suporte a WebP no mesmo editor.

Para conferir o suporte, vá em **Ferramentas > Saúde do site > Informações > Mídia**.

---

## Instalação

1. Baixe o arquivo `avif-webp.php` deste repositório.
2. Envie para `wp-content/plugins/avif-webp/` (ou para `wp-content/mu-plugins/`).
3. Ative em **Plugins** no painel do WordPress.

---

## Configuração

Não há tela de opções. Os valores ficam no próprio arquivo:

| Item | Valor padrão | Onde alterar |
|---|---|---|
| Qualidade AVIF | 60 | filtro `wp_editor_set_quality` e `set_quality( 60 )` |
| Qualidade WebP | 75 | filtro `wp_editor_set_quality` e `set_quality( 75 )` |

Para capturas de tela com muito texto, uma qualidade AVIF maior (por exemplo 70 a 80) evita suavização nas bordas das letras.

---

## Limitações e avisos importantes

**Leia antes de usar em produção.**

- **A remoção dos originais é definitiva.** Faça backup da pasta `uploads` antes de ativar o plugin. Depois de apagados, o JPG/PNG original não pode ser recuperado.
- **Só atua em novos uploads.** Mídias que já estão na biblioteca não são convertidas.
- **Edição de imagem no painel** (cortar, girar, restaurar original) não está coberta. Como o original é removido, a opção de restaurar não funciona.
- **Plugins de regeneração de miniaturas** (como Regenerate Thumbnails) vão recriar os tamanhos a partir do AVIF já comprimido, com perda de qualidade a cada geração.
- **Os espelhos WebP não são registrados no WordPress.** Eles ficam no disco, mas o plugin não altera o HTML nem a URL das imagens. Quem entrega o WebP é o servidor (veja abaixo).
- **URLs antigas** que apontam para `.jpg` ou `.png` em posts já publicados continuam apontando para arquivos que, em novos uploads, deixam de existir.

---

## Servindo o WebP para navegadores sem AVIF (opcional)

O WordPress passa a usar o `.avif` como arquivo principal, e hoje quase todos os navegadores modernos o suportam. Se você quiser entregar o `.webp` ao pequeno grupo que não suporta AVIF, configure o servidor.

Exemplo para **nginx** (adapte e teste no seu ambiente antes de usar):

```nginx
map $http_accept $avif_ok {
    default          0;
    "~*image/avif"   1;
}

location ~* ^(?<base>.+)\.avif$ {
    add_header Vary Accept;
    if ($avif_ok = 0) {
        try_files $base.webp $uri =404;
    }
}
```

No Apache, regras equivalentes com `mod_rewrite` e o cabeçalho `Accept` funcionam da mesma forma.

---

## Como testar

1. Envie um JPG pela biblioteca de mídia.
2. Na pasta `uploads`, confirme que existem pares `.avif` e `.webp` para o principal e para cada tamanho, e que o JPG não existe mais.
3. Compare os tamanhos: o `.avif` deve ser menor que o `.webp`.
4. Exclua a mídia e confirme que todos os arquivos somem do disco.

Se algum original continuar no disco, procure no `debug.log` por `[AVIF+WEBP] Originais mantidos por seguranca`.

---

## Roteiro

- [ ] Rotina em lote via **WP-CLI** para converter a biblioteca existente
- [ ] Tela de configurações (qualidade, manter originais, tipos de imagem)
- [ ] Suporte à edição de imagens no painel
- [ ] Entrega automática do WebP via `<picture>`

---

## Contribuindo

Issues e pull requests são bem-vindos. Ao reportar um problema, inclua:

- versão do WordPress e do PHP;
- editor de imagens em uso (Imagick ou GD) e versões;
- trecho relevante do `debug.log`;
- tipo, dimensões e tamanho da imagem enviada.

---

## Licença

Distribuído sob a licença **GPL-2.0-or-later**, a mesma do WordPress.

---

## Autor

**Carlos Araújo** · [suainternet.com](https://suainternet.com)
