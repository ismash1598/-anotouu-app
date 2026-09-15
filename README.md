# anotouu

Interfaces web mobile-first, feitas pra serem usadas pelo celular. Cada página é
um arquivo só, sem build e sem dependências: é abrir no navegador e usar.

| Arquivo | O que é |
| --- | --- |
| `jogo.html` | Jogo de teste: o boneco anda pela grama com o d-pad da tela |
| `poster.html` | Editor de pôster: sua imagem de fundo, títulos e sua logo |
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

---

# Editor de pôster (`poster.html`)

Diferente do editor de capa, aqui não há desenho fixo: você põe a imagem de
fundo, escreve os títulos e posiciona a logo onde quiser.

## O que dá para fazer

- **Fundo** — sua imagem, com aproximar, escurecer ou clarear, e arrastar para
  reposicionar. Sem imagem, fica o cinza de fundo.
- **Textos** — quantos quiser. Fonte, tamanho, cor, alinhamento, entrelinha,
  giro e contorno com cor e espessura próprias.
- **Logo** — vem embutida no arquivo, então já aparece. Move, gira, muda de
  tamanho, de opacidade e de cor. Dá para trocar por outra imagem e voltar.

Arraste na arte para mover, ou toque para selecionar. A trilha de pastilhas
acima do painel lista tudo que está no pôster e serve para alcançar o que
estiver por baixo.

Formatos: cartaz (2:3), feed (4:5), quadrado (1:1) e story (9:16). As posições
são guardadas em fração da largura e da altura, então trocar de formato
reacomoda tudo em vez de quebrar o arranjo.

## Giro

Texto e logo giram a volta inteira, de −180° a 180°. O slider sozinho daria uns
1,5° por pixel de dedo, o que não acerta ângulo reto, então junto dele vão os
atalhos: 0°, 90°, 180°, −90° e passos de 15°. O valor é normalizado para a
faixa, de forma que somar 15° em 175° dá −170° e não 190°.

O acerto do toque gira o ponto no sentido inverso ao da camada antes de comparar
com a caixa, então pegar e arrastar continua funcionando em qualquer ângulo,
inclusive de cabeça para baixo.

## Fontes

Oito fontes de cartaz, todas do Google Fonts. A **Luckiest Guy** é o padrão por
ser a mais próxima do lettering desenhado à mão da referência: traço grosso,
contorno irregular, caixa alta. As outras: Bungee, Titan One, Lilita One, Anton,
Archivo Black, Permanent Marker e Shrikhand.

## Como a logo troca de cor

A logo é guardada com RGB branco e o desenho todo no canal alfa — bem menor que
o arquivo original, já que o RGB vira constante e comprime quase a zero.

Para tingir, ela é desenhada num canvas de apoio e por cima vai um
`fillRect` com `globalCompositeOperation = "source-in"`, que pinta só onde o
alfa existe. O resultado guarda as bordas suaves, então a logo não serrilha em
nenhuma cor. Cada combinação de imagem e cor fica em cache.

O botão "Original" pula o tingimento e desenha a imagem como ela é — útil
depois de trocar por uma logo colorida.

## Posições e acerto do toque

Cada camada guarda `x` e `y` como fração (0 a 1) e é ancorada pelo **centro**.
O alinhamento do texto só decide como as linhas se distribuem dentro do bloco,
não onde o bloco está — assim a caixa de toque continua centrada mesmo com o
texto alinhado à esquerda.

Para saber o que foi tocado, o ponto é girado no sentido inverso ao da camada e
comparado com a caixa dela, com uma folga de 14 unidades para dedo. A lista é
percorrida de trás para frente, então o que está por cima ganha.

## Exportar

A imagem sai em PNG com 1600px de largura. O contorno de seleção e as listras
do fundo vazio existem só na tela: a exportação redesenha tudo com uma marca de
"exportando" que os omite, tira a imagem e redesenha a tela de novo.

No celular, toque e segure na imagem gerada e escolha "Salvar imagem".

---

# Jogo de teste (`jogo.html`)

O personagem anda casa por casa por um campo de grama, com o d-pad da tela ou
pelo teclado. Visão de cima, câmera presa no boneco e o mapa correndo por
baixo — o jeito do Tibia.

Um toque dá um passo; segurar anda sem parar. O controle de passo vai de 140ms
a 520ms por casa.

## A folha de sprites

`sprites-personagem.png` é a folha recortada da imagem original: 4 linhas
(norte, leste, sul, oeste) × 8 frames, célula de 59 × 97, fundo transparente.
Ela vai embutida no `jogo.html` como data URI, então o jogo é um arquivo só.

O recorte foi feito assim:

1. **Achar a grade** pela cabeça do boneco, que é clara e pouco saturada. O
   brilho azul dos rótulos é saturado, então esse teste separa um do outro sem
   pegar os crachás de direção nem o cabeçalho.
2. **Isolar o boneco** por componente conexo sobre `alfa > 150`. O alfa separa
   limpo (o histograma é quase todo 0 ou 255), enquanto filtrar por
   luminância comia o manto escuro e fazia a altura variar de 66 a 135 px.
   Pegar o maior blob que cruza o meio da célula descarta o número de baixo.
3. **Alinhar pelos pés.** Cada frame entra na célula com a base no mesmo y e o
   centro da caixa no meio. Sem isso o boneco sobe e desce a cada passo, porque
   as poses de leste e oeste são mais estreitas que as de norte e sul. Medido:
   a base varia 1 a 2 px dentro de cada linha, e o centro, 2 a 3 px.
4. **Limpar e comprimir.** A máscara é dilatada 2 px para não cortar a borda
   suave, alfa abaixo de 38 vira zero (poeira de compressão) e a folha é
   posterizada em 5 bits por canal — de 514 KB para 146 KB, sem mudança
   visível numa arte de branco, cinza e azul-marinho.

## O passo

Um passo leva o boneco de uma casa à vizinha em `duracaoPasso`
milissegundos, interpolando a posição da câmera. Os 8 frames são distribuídos
ao longo desse trajeto.

O contador `ciclo` não zera entre passos: o frame sai de
`(ciclo + t) * 8`, então o passo seguinte continua de onde a perna parou, em
vez de recomeçar sempre no mesmo pé. Parado, volta ao frame 0.

## A grama

O campo é infinito e procedural. Um hash inteiro de `(x, y)` escolhe uma entre
16 variantes pré-desenhadas, então a mesma casa é sempre igual e nada precisa
ser guardado.

Todas as variantes partem da **mesma cor de base**. Essa é a parte que importa:
na primeira versão cada variante tinha o próprio tom, e o resultado desenhava a
grade do mapa na tela, feito colcha de retalhos. A variação vem só do salpico
miúdo, dos tufos e dos enfeites esparsos — flor, pedra e uma mancha de terra.

As variantes são redesenhadas no tamanho final da casa a cada mudança de
tamanho da janela, e depois só copiadas. Assim não há imagem escalada, que
borraria.
