# anotouu

Interfaces web mobile-first, feitas pra serem usadas pelo celular. Cada página é
um arquivo só, sem build e sem dependências: é abrir no navegador e usar.

| Arquivo | O que é |
| --- | --- |
| `capa.html` | Editor de capa de revista: troca as fotos e os textos e gera a imagem |
| `index.html` | Bloco de notas: escrever, buscar e ajustar |

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

---

# Editor de capa de revista (`capa.html`)

Reproduz uma capa de banca — manchete, selo de preço, chamada lateral, foto
grande, seção especial com duas fotos e rodapé de sabores — deixando editável
só o que muda: o texto de cada elemento e as três fotos.

## Como usar

Toque em qualquer pedaço da capa: o formulário daquele elemento abre e recebe o
foco. Ou role até os grupos de campos e edite direto. Tudo redesenha na hora.

"Gerar imagem" abre a capa pronta em cima da página. No celular, toque e segure
na imagem e escolha "Salvar imagem" — é assim que ela vai pra galeria. (Um link
de download comum não funciona dentro de página hospedada, por isso o caminho é
esse.)

## Por que canvas e não HTML

A capa é desenhada num `<canvas>`, não montada com elementos HTML. Assim o que
aparece na tela é exatamente o arquivo exportado: contorno de letra, degradê e
recorte de foto saem idênticos, sem depender de biblioteca de captura de tela.

## Como o desenho funciona

Tudo é desenhado num plano fixo de 760 × 1100, e o canvas roda em 2× para a
imagem final sair com 1520 × 2200. A função `desenhar()` pinta de trás pra
frente: fundo, foto principal, selo, chamada, manchete, seção especial, fotos de
baixo, rodapé e lombada. A foto principal vem antes do selo de propósito — na
capa original o selo encosta nela.

Auxiliares:

- `T(texto, x, y, opções)` — escreve com contorno, sombra e espaçamento
- `caber(...)` — diminui o corpo da letra até o texto caber na largura dada, pra
  que um texto longo não vaze do lugar
- `cantos(...)` — retângulo de cantos arredondados
- `moldura(chave, x, y, w, h, raio)` — moldura branca com a foto recortada
  dentro, ou o espaço vazio convidando a escolher uma

## Áreas tocáveis

Enquanto desenha, cada elemento registra sua caixa com `alvo(chave, x, y, w, h)`.
O toque na capa percorre essa lista de trás pra frente, então quem foi desenhado
por último ganha — é o que faz o selo responder no lugar da foto onde os dois se
sobrepõem.

## Fotos

A foto escolhida é reduzida para no máximo 1000px e recomprimida em JPEG antes
de ser guardada, senão o navegador estoura a cota. Ela é recortada para
preencher a moldura; o controle de enquadramento escolhe qual parte aparece,
de cima (0) a baixo (100).

Se as três fotos ainda assim não couberem na cota, aparece um aviso: a capa
continua funcionando, mas pode não voltar igual depois de fechar a página.

## Textos

Dois campos aceitam várias linhas:

- **Lista de tópicos** — um por linha. Eles se distribuem sozinhos, com quantos
  couberem por linha, até três linhas.
- **Chamada lateral** — a quebra de linha que você digitar é respeitada.

O rodapé é dividido nos `•` e recolorido alternando branco e amarelo, encolhendo
a letra até caber na largura da capa.

## Fontes

Anton (números e manchetes), Archivo Black (marca), Yellowtail (os trechos em
manuscrito) e Barlow Condensed (listas, preço, legendas e rodapé), via Google
Fonts. O canvas é redesenhado quando as fontes terminam de carregar.
