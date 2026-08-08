# Aura Fragrâncias — Catálogo Digital

Vitrine de perfumes que **não processa pagamentos**. O cliente escolhe o produto,
preenche os dados de entrega e o pedido é enviado formatado para o WhatsApp da
vendedora. O pagamento é combinado por Pix na conversa.

## Arquivo principal

`mr-parfum-natal.html` — **arquivo único** com HTML, CSS e JavaScript embutidos.
Não há build, dependências ou servidor. Basta abrir no navegador ou hospedar.

> O nome do arquivo é herança da primeira versão do projeto (a loja chamava-se
> "Mr. Parfum Natal"). A marca mudou para **Aura Fragrâncias**, mas o arquivo
> manteve o nome para não quebrar links já compartilhados.

## Dados do negócio

| Item | Valor |
|---|---|
| Nome da loja | Aura Fragrâncias (rodapé: "Aura Perfumes") |
| Slogan | Fragrâncias inspiradas em grifes internacionais |
| WhatsApp | `5584999419888` — constante `NUMERO_WHATSAPP` no JS |
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

Cinco filtros, disponíveis tanto na barra lateral quanto nas abas:

- **Coleção Completa** — todos os produtos
- **Novidades** — produtos com `novidade: true`
- **Mais Vendidos** — produtos com `maisVendido: true`
- **Menor Preço** / **Maior Preço** — ordenação por `preco`

A busca (`#campoBusca`) ignora acentuação — "pera" encontra "Pêra e Frésia".

## Tarefas comuns de manutenção

### Alterar preço de um produto

No array `produtos` (dentro do `<script>`), edite o campo `preco`.
Use `precoOriginal` para mostrar o preço riscado e o selo de desconto —
a porcentagem é calculada sozinha. Deixe `null` se não houver promoção.

```js
{
  nome: "Miss Blue 150ml",
  preco: 239.90,          // preço atual
  precoOriginal: 279.90,  // riscado; null se não houver desconto
  disponivel: true,       // false = card cinza, botão desativado
  novidade: true,         // aparece na aba "Novidades"
  maisVendido: false,     // aparece na aba "Mais Vendidos"
  img: "https://...",
  desc: "Floral frutal, leve e envolvente."
}
```

### Marcar produto como esgotado

Troque `disponivel: true` por `disponivel: false`. O card fica em escala de cinza
e o botão vira "Indisponível", sem link.

### Trocar o produto em destaque

Edite o objeto `DESTAQUE`. O campo `nome` **precisa ser idêntico** ao `nome` de um
produto do array `produtos` — é assim que a foto e o preço são buscados.

```js
const DESTAQUE = {
  nome: "Rosa e Lichia Extrait de Parfum 50ml", // igual ao array produtos
  selo: "Fragrância em Destaque",
  subtitulo: "Extrait de Parfum · 50ml",
  texto: "...",
  notas: [ { titulo: "Saída", valor: "..." }, ... ]
};
```

### Ajustar valores de frete

As constantes `FRETE_LOCAL` e `FRETE_POR_REGIAO` guardam **estimativas**, não
valores reais dos Correios (que exigem contrato para consulta via API).

```js
const FRETE_LOCAL = 15.00;          // Natal e Nova Parnamirim
const FRETE_POR_REGIAO = {
  RN: 25.00,                        // demais cidades do RN
  Nordeste: 30.00,
  "Sudeste/Sul": 38.00,
  "Norte/Centro-Oeste": 45.00
};
```

A cidade é descoberta pela API pública **ViaCEP** (`viacep.com.br`), que é gratuita
e não exige cadastro. O mapa `UF_REGIAO` converte a UF retornada em faixa de preço.

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

As imagens apontam para o CDN do Mercado Livre (`http2.mlstatic.com`), da loja
[MR PARFUM](https://lista.mercadolivre.com.br/loja/mr-parfum). Se um anúncio for
removido de lá, a foto quebra. Para produção, o ideal é hospedar cópias próprias.

## Hospedagem

Como é um arquivo único e estático, as opções mais simples:

- **Netlify Drop** (`app.netlify.com/drop`) — arrasta o arquivo, recebe o link. Sem conta.
- **GitHub Pages** — se já usa GitHub.
- **Vercel** — deploy por arrastar ou via Git.

Para o link ficar bonito na bio do Instagram, vale registrar um domínio próprio
e apontar para qualquer uma dessas opções.

## Limitações conhecidas

- **Sem login real.** A "Área do Cliente" apenas monta uma mensagem de consulta
  para o WhatsApp. Login com senha e histórico de pedidos exigiriam backend.
- **Sem carrinho.** Cada pedido é de um produto só. Múltiplos itens exigiriam
  refatorar o modal de checkout.
- **Frete estimado**, não cotado em tempo real.
- **Sem controle de estoque.** A disponibilidade é marcada manualmente no código.

## Convenções ao editar

- Todo o texto visível ao cliente é em **português do Brasil**.
- Nomes de variáveis e funções também em português (`renderizarProdutos`,
  `calcularFrete`, `produtoSelecionado`) — manter o padrão.
- Paleta: preto `#111`, dourado `#c9a24b`, dourado claro `#e8d8a8`,
  cinza claro `#f7f6f4`, verde WhatsApp `#25d366`. Definida em `:root`.
- Design **mobile-first** — a maioria dos acessos vem do Instagram, pelo celular.
