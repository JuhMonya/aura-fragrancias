# Aura Fragrâncias — Catálogo Digital

Vitrine de perfumes que **não processa pagamentos**. O cliente escolhe o produto,
preenche os dados de entrega e o pedido é enviado formatado para o WhatsApp da
vendedora. O pagamento é combinado por Pix na conversa.

## Arquivos do projeto

`mr-parfum-natal.html` — HTML, CSS e JavaScript embutidos em um só arquivo.
Não há build, dependências ou servidor.

`imagens/` — as 44 fotos (30 da linha padrão, 9 da Linha Exclusiva — os 3 Extrait
usam duas fotos cada — e 5 do Perfume de Cabelo, prefixadas com `cabelo-`),
referenciadas por caminho relativo
(`imagens/bleu-noir.jpg`). **O HTML sozinho não funciona** — a pasta precisa
ficar ao lado dele.

> O nome do arquivo é herança da primeira versão do projeto (a loja chamava-se
> "Mr. Parfum Natal"). A marca mudou para **Aura Fragrâncias**, mas o arquivo
> manteve o nome para não quebrar links já compartilhados.

## Dados do negócio

| Item | Valor |
|---|---|
| Nome da loja | Aura Fragrâncias (rodapé: "Aura Perfumes") |
| Slogan | Fragrâncias inspiradas em grifes internacionais |
| WhatsApp | `5584991138834` — constante `NUMERO_WHATSAPP` no JS |
| Pagamento | Pix (chave enviada na conversa) |
| Entrega local | Natal e Nova Parnamirim (motoboy) |
| Envio | Correios para todo o Brasil |

**O número de WhatsApp nunca é exibido na tela.** Ele só é usado para montar os
links `https://wa.me/...`.

## Estrutura da página

Três "páginas" dentro do mesmo arquivo, alternadas por JavaScript (sem recarregar):

1. **Início** (`#pagina-inicio`) — produto em destaque + coleção com busca e categorias
2. **Contato** (`#pagina-contato`) — canais de atendimento + botão WhatsApp
3. **Área do Cliente** (`#pagina-cliente`) — formulário de consulta de pedido via WhatsApp

A navegação usa `.link-nav[data-pagina]` e a função `irParaPagina()`.

### Categorias da coleção

Seis filtros, disponíveis tanto na barra lateral quanto nas abas:

- **Coleção Completa** — todos os produtos
- **Linha Exclusiva** — produtos com `linha: "Exclusiva"` (6 itens)
- **Perfume de Cabelo** — produtos com `linha: "Cabelo"` (5 itens)
- **Novidades** — produtos com `novidade: true`
- **Femininos** / **Masculinos** — filtra por `genero`

A aba **Mais Vendidos** foi removida em 14/08/2026. O campo `maisVendido` continua
em cada produto, mas está **inativo** — para trazer a aba de volta, recriar o botão
`data-cat="vendidos"` (sidebar e abas) e o ramo correspondente em `filtrarProdutos()`.

`genero: "Compartilhável"` faz o produto aparecer **nos dois** filtros de gênero.
Hoje nenhum produto usa esse valor: Bergamota &amp; Alecrim e Limão e Mandarina eram
compartilháveis e passaram a **Feminino** em 14/08/2026, por decisão da loja. A regra
segue no código para quando algum produto voltar a ser unissex.

A busca (`#campoBusca`) ignora acentuação — "pera" encontra "Pêra e Frésia" — e
procura também na inspiração e na família olfativa ("jo malone" traz 4 perfumes).

## Tarefas comuns de manutenção

### Alterar preços

**Todos os perfumes usam a mesma tabela.** Não há preço por produto — o preço
vem do tamanho escolhido no card. Para reajustar, edite a constante `TAMANHOS`:

```js
const TAMANHOS = [
  { ml: 10,  preco:  32.00 },
  { ml: 30,  preco:  65.00 },
  { ml: 50,  preco:  90.00 },
  { ml: 150, preco: 170.00 }
];
const TAMANHO_PADRAO = 10;  // tamanho já marcado quando a página abre
```

Para adicionar ou remover um tamanho, basta mexer nesse array — os botões, o
preço, o modal e a mensagem do WhatsApp se ajustam sozinhos. O `TAMANHO_PADRAO`
precisa existir na lista.

### Linha Exclusiva — tabela própria

Um produto pode ter a **própria** tabela via campo `tamanhos`, ignorando a padrão.
A Linha Exclusiva tem duas tabelas, e **Extrait e Deo Parfum são produtos
separados** — são concentrações diferentes, não tamanhos do mesmo perfume:

```js
const TAMANHOS_EXTRAIT = [
  { ml: 10, preco:  40.00, tipo: "Extrait de Parfum", curto: "Extrait" },
  { ml: 50, preco: 200.00, tipo: "Extrait de Parfum", curto: "Extrait" }
];
const TAMANHOS_PARFUM_100 = [
  { ml: 100, preco: 190.00, tipo: "Deo Parfum", curto: "Parfum" }
];
```

São 6 cards: 3 fragrâncias (Absolu Pêra e Frésia, Rosa e Lichia, Rosa e Cereja)
× 2 concentrações. O Extrait tem 30% de essência e até 12h de fixação; o Deo
Parfum, 20% e 6–8h. É por isso que 50ml de Extrait (R$ 200) custa mais que 100ml
de Deo Parfum (R$ 190) — **não é erro de digitação**, e é justamente por isso que
os dois não ficam no mesmo card.

O campo `curto` só aparece embaixo do número no botão quando a tabela do produto
mistura tipos. Hoje nenhuma mistura, mas a lógica continua valendo se alguma
tabela futura misturar.

**Tabela com um só tamanho não gera botões** (`montarBotoesTamanho` devolve
string vazia) — um botão sozinho e sempre marcado não é escolha nenhuma. O preço
aparece direto. É o caso dos três Deo Parfum de 100ml.

A nota técnica da ficha alterna entre `TEXTO_EXTRAIT` e `TEXTO_CONCENTRACAO`
conforme o `curto` do tamanho selecionado.

Se `TAMANHO_PADRAO` não existir na tabela do produto, ele abre no menor tamanho.

### Perfume de Cabelo (Body &amp; Hair Mist)

Cinco produtos, `linha: "Cabelo"`, tamanho único de 200 ml e **preço que muda de
fragrância para fragrância** — R$ 110 (Rose Lumière, Pêra e Frésia) e R$ 120
(Absolu Pêra e Frésia, Rosa e Lichia, Rosa e Cereja). Por isso a tabela vem de
uma função em vez de constante:

```js
function tabelaMist(preco){
  return [{ ml: 200, preco: preco, tipo: "Body &amp; Hair Mist", curto: "Mist", nota: TEXTO_MIST }];
}
// no produto:  tamanhos: tabelaMist(110.00)
```

Como só há um tamanho, esses cards não mostram botões — só o preço.

O campo `selo` troca o texto do selo preto sem mexer no filtro: `linha: "Cabelo"`
com `selo: "Hair Mist"` mostra "HAIR MIST" no card e continua filtrando por
`linha`.

### Nota técnica da ficha

Cada entrada de tamanho pode trazer um campo `nota` com o texto que aparece no
rodapé da ficha. Sem `nota`, usa `TEXTO_CONCENTRACAO` (o Deo Parfum padrão).
`TEXTO_EXTRAIT` e `TEXTO_MIST` são os outros dois. A nota é atualizada junto com
o preço quando o cliente troca de tamanho.

⚠️ `TEXTO_EXTRAIT` e `TEXTO_MIST` precisam estar declarados **antes** das tabelas
que os referenciam — `const` não sofre hoisting e a página quebra inteira
("Cannot access before initialization") se a ordem inverter.

### Foto diferente por tamanho

Nos Extrait o frasco de 10ml é diferente do de 50ml, então esses produtos trazem
`imgPorTamanho` e a foto troca junto com o botão:

```js
imgPorTamanho: { 10:"imagens/...-10.jpg", 50:"imagens/...-50.jpg" }
```

`img` continua obrigatório — é o fallback de quem não tem `imgPorTamanho`.

### Estrutura de um produto

```js
{
  nome: "Miss Blue",
  inspiracao: "Light Blue (Dolce &amp; Gabbana)",  // null se não houver
  familia: "Floral Frutado",
  genero: "Feminino",       // "Feminino" | "Masculino" | "Compartilhável"
  img: "imagens/miss-blue.jpg",
  disponivel: true,         // false = card cinza, botão desativado
  novidade: false,          // aba "Novidades" + selo dourado no card
  maisVendido: true,        // INATIVO — a aba "Mais Vendidos" foi removida
  linha: undefined,         // "Exclusiva" põe o selo preto e entra na aba própria
  tamanhos: undefined,      // tabela de preços própria (padrão: TAMANHOS)
  imgPorTamanho: undefined  // troca a foto conforme o tamanho escolhido
}
```

**Inspiração em branco não vira texto.** `inspiracao: null` faz a linha sumir do
card — não inventar "criação exclusiva da casa" para quem o fornecedor só não
informou. Rose Lumière e Mr Black estão nesse caso; Absolu Pêra e Frésia e
Bergamota &amp; Alecrim são criações exclusivas de verdade, declaradas pelo fornecedor.

**Só um selo por card.** Se o produto tem `linha`, aparece o selo da linha; senão,
o de "Novidade". Os dois juntos se sobrepõem quando a grade está em 4 colunas.

A frase sobre concentração e fixação é **igual para todos** e fica na constante
`TEXTO_CONCENTRACAO`. Use entidades HTML (`&amp;`) nos textos — eles são
injetados via `innerHTML`.

### Marcar produto como esgotado

Troque `disponivel: true` por `disponivel: false`. O card fica em escala de cinza
e o botão vira "Indisponível", sem link.

### Trocar o produto em destaque

Edite o objeto `DESTAQUE`. O campo `nome` **precisa ser idêntico** ao `nome` de um
produto do array `produtos` — é assim que a foto e o preço são buscados.

```js
const DESTAQUE = {
  nome: "Pêra e Frésia",          // igual ao array produtos
  selo: "Fragrância em Destaque",
  subtitulo: "Deo Parfum · 10, 30, 50 e 150 ml",
  texto: "...",
  notas: [ { titulo: "Inspiração", valor: "..." }, ... ]
};
```

O bloco de destaque também tem seletor de tamanho. Como o perfume em destaque
aparece **duas vezes** na página (destaque + grade), trocar o tamanho em um lugar
atualiza o outro — é o que faz `trocarTamanho()` receber `document` em vez de só
o card clicado.

### Frete — não é calculado (decisão de 14/08/2026)

**O site não simula frete.** O checkout mostra "Frete a combinar" e a mensagem do
WhatsApp sai com `Frete: a combinar`, sem total fechado. O valor é acertado na
conversa, junto com a chave Pix.

Existia antes uma calculadora com valores fixos por região (`FRETE_LOCAL = 15`,
`FRETE_POR_REGIAO` de 25 a 45). Aqueles números **eram chutes**, não vinham de
tabela dos Correios, e não olhavam peso nem tamanho do frasco — um 10 ml e um
150 ml pagavam o mesmo. Como a mensagem do WhatsApp fechava um "Total", qualquer
erro para menos saía do bolso da vendedora. Foi tudo removido.

⚠️ **Não reintroduzir estimativa sem números reais.** Se um dia voltar a calcular,
os valores precisam sair de cotação de verdade (Correios, SuperFrete, Melhor Envio)
com o peso da caixa fechada, e a mensagem não deve prometer total antes da
confirmação.

O ViaCEP (`viacep.com.br`, público e gratuito) continua em uso, mas agora só para
**preencher endereço e bairro sozinho** quando o cliente sai do campo de CEP —
função `preencherEnderecoPeloCep()`. Se a API falhar, aparece um aviso e o cliente
digita à mão; nada trava o pedido.

### Trocar o número de WhatsApp

Altere a constante `NUMERO_WHATSAPP`. O formato é `55` + DDD + número, sem
espaços, traços ou parênteses. Ela é usada em quatro lugares (checkout, contato,
rodapé e área do cliente) — mudar a constante atualiza todos.

## Detalhes técnicos importantes

### Imagens embutidas em base64

A **logo** e o **favicon** estão embutidos como `data:image/png;base64,...` direto
no HTML. Isso evita arquivos soltos e faz o site funcionar mesmo aberto localmente
com `file://`.

⚠️ **Ao editar essas strings, cuidado com truncamento.** São strings muito longas;
uma edição parcial corrompe a imagem silenciosamente (o navegador mostra ícone
quebrado). Se precisar trocar a logo, gere o base64 completo e substitua a string
inteira de uma vez, verificando o início (`iVBORw0KGgo...`) e o fim (`...ErkJggg==`).

### Comentário HTML residual

Existe um bloco `<!-- ... -->` grande logo após a tag `<img class="logo">`, com
sobras de versões antigas da logo em base64. É inofensivo (o navegador ignora),
mas pode ser removido para deixar o arquivo mais leve.

### Fotos dos produtos

Ficam em `imagens/`, **todas `.jpg`** e com nome derivado do perfume
(`flor-de-acacia.jpg`). As da Linha Exclusiva levam o tamanho no fim
(`exclusiva-rosa-e-lichia-50.jpg`) porque cada formato tem foto própria. Vieram do catálogo do fornecedor
[atacadomrparfum.com.br](https://atacadomrparfum.com.br) (categoria 6316).

Os originais eram fotos de celular de 3024×4032 (17 MB no total). Foram
redimensionadas para no máximo 1000 px e salvas em JPEG qualidade 85 —
**1,3 MB no total**, nenhuma passando de 120 KB. Ao trocar uma foto, repita esse
tratamento: o site é mobile-first e 17 MB de imagem inviabilizam a página no 4G.

A constante `PASTA_IMAGENS` existe como documentação do caminho; os produtos já
trazem o caminho completo no campo `img`. As imagens da grade usam
`loading="lazy"` — só carregam quando o cliente rola até elas.

## Hospedagem

O site é estático, mas **não é mais um arquivo único** — precisa subir o HTML
**e** a pasta `imagens/` juntos:

- **Netlify Drop** (`app.netlify.com/drop`) — arrasta a **pasta inteira**, recebe o link. Sem conta.
- **GitHub Pages** — se já usa GitHub.
- **Vercel** — deploy por arrastar ou via Git.

Para o link ficar bonito na bio do Instagram, vale registrar um domínio próprio
e apontar para qualquer uma dessas opções.

## Limitações conhecidas

- **Sem login real.** A "Área do Cliente" apenas monta uma mensagem de consulta
  para o WhatsApp. Login com senha e histórico de pedidos exigiriam backend.
- **Sem carrinho.** Cada pedido é de um produto e um tamanho só. Múltiplos itens
  exigiriam refatorar o modal de checkout.
- **Sem estoque por tamanho.** `disponivel: false` derruba o produto inteiro; não
  dá para marcar só o 150ml como esgotado.
- **Dois perfumes sem inspiração informada** (Rose Lumière e Mr Black) — o
  fornecedor não publica a descrição deles. O campo está `null` e a linha
  "Inspiração" simplesmente não aparece no card.
- **Frete não é calculado** — combinado no WhatsApp (veja a seção de frete).
- **Sem controle de estoque.** A disponibilidade é marcada manualmente no código.

## Convenções ao editar

- Todo o texto visível ao cliente é em **português do Brasil**.
- Nomes de variáveis e funções também em português (`renderizarProdutos`,
  `preencherEnderecoPeloCep`, `produtoSelecionado`) — manter o padrão.
- Paleta: preto `#111`, dourado `#c9a24b`, dourado claro `#e8d8a8`,
  cinza claro `#f7f6f4`, verde WhatsApp `#25d366`. Definida em `:root`.
- Design **mobile-first** — a maioria dos acessos vem do Instagram, pelo celular.
