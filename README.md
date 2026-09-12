# anotouu

Interface web mobile-first, feita pra ser usada pelo celular. Um arquivo só,
sem build, sem dependências: é abrir o `index.html` e usar.

## Como rodar

Não precisa de servidor nem de `npm`. Três formas, da mais simples pra menos:

1. **Link hospedado** — a versão publicada abre direto no navegador do celular.
2. **GitHub Pages** — em Settings → Pages, escolha a branch e a raiz (`/`).
   O `index.html` passa a ser servido como site.
3. **Local** — abra o arquivo `index.html` no navegador.

Pra usar como app, adicione à tela de início: no iPhone, Compartilhar →
"Adicionar à Tela de Início"; no Android, menu de três pontos → "Adicionar à
tela inicial".

## O que já funciona

- **Notas** — escrever, editar (toque no texto), fixar e apagar. Apagar pede
  confirmação na própria linha, sem diálogo do navegador.
- **Busca** — filtra as notas e destaca o trecho encontrado.
- **Ajustes** — tema (sistema / claro / escuro), total de notas e caracteres,
  copiar tudo como texto e apagar tudo.

Horários aparecem em português e em formato brasileiro: "agora", "há 12 min",
"ontem", `12/09`.

## Onde os dados ficam

Em `localStorage`, no próprio aparelho. Nada sai do navegador — e nada
sincroniza entre aparelhos. Limpar os dados do site apaga as notas. Toda
leitura e escrita está protegida com `try/catch`, então a página continua
funcionando em aba anônima ou com armazenamento bloqueado.

Chaves usadas: `anotouu.notes.v1`, `anotouu.draft.v1`, `anotouu.theme.v1`.

## Estrutura do arquivo

`index.html` carrega na ordem: tokens de cor → estilos → marcação → script.

Os comentários `<!--#artifact-start-->`, `#artifact-split`, `#artifact-resume`
e `#artifact-end` delimitam a parte publicável da página (tudo menos o
esqueleto `html`/`head`/`body`). Servem pra extrair o trecho sem duplicar
código:

```sh
awk '/#artifact-start/{k=1;next} /#artifact-split/{k=0;next} \
     /#artifact-resume/{k=1;next} /#artifact-end/{k=0;next} k' index.html
```

### Cores

Todos os valores são tokens CSS em `:root`, redefinidos em três estados: claro
(padrão), `prefers-color-scheme: dark` e `[data-theme="dark"]`. Nenhuma cor é
declarada só dentro de um bloco de tema — mudar a paleta é mexer nos tokens.

| Token | Papel |
| --- | --- |
| `--paper` | fundo da página |
| `--surface` | folha das notas e painéis |
| `--ink`, `--ink-2`, `--ink-3` | texto principal, secundário, terciário |
| `--accent` | azul de caneta esferográfica |
| `--mark` | amarelo de marca-texto: nota fixada e destaque na busca |

### Tipografia

Bricolage Grotesque (marca), Instrument Sans (texto), DM Mono (metadados e
etiquetas), via Google Fonts, com pilha de fallback declarada.

## Próximos passos

O conteúdo do app ainda não foi definido. A casca — abas, tema, persistência,
busca — já está pronta, então dá pra trocar o miolo sem refazer a estrutura.
