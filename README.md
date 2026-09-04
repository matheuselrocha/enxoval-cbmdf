# Enxoval CFP CBMDF 2026 — web app

App mobile-first de **um arquivo só** (`enxoval-cfp-cbmdf-v2.html`) que lê a planilha
pública no Google Sheets e mostra os dados de um jeito usável no celular. Sem backend,
sem chave de API, sem login. É só arrastar o `.html` para o **Netlify Drop** e pronto.

## As 4 telas (o que a planilha faz, agora como app)

| Aba | O que é |
|---|---|
| **Calculadora** | A tela principal. Cada item tem quantidade (− / +) e loja escolhível; o preço unitário, o subtotal da linha, o subtotal do bloco e o **total geral** recalculam na hora. Marque "já comprei" e o valor sai do "ainda a comprar" sem sair do total. Filtro Todos / Masculino / Feminino. |
| **Lojas** | Contato das 9 lojas (WhatsApp + Instagram), tabela de parcelamento/desconto à vista e o cupom. |
| **Online** | Cards dos itens que valem comprar pela internet, cada um com um botão principal e as demais opções recolhidas. |
| **Bizus** | Materiais do CFP (ordem unida, hinos, nós, cuidado com os pés, manutenção do coturno, kit costura…). |

## Como funciona a leitura dos dados

- Lê a aba **`Enxoval Unificado`** pelo endpoint gviz (CSV público, sem credencial):
  `https://docs.google.com/spreadsheets/d/{ID}/gviz/tq?tqx=out:csv&sheet={Aba}`
- **Célula com 0 = a loja não vende** aquele item (não entra em comparação nem em soma).
- Preço padrão de cada item = **menor entre as lojas que vendem**; quantidade padrão = "Qtd Sugerida".
- Se a leitura falhar (rede/gviz fora do ar), cai no **snapshot embutido** no próprio arquivo
  e segue funcionando. O ponto ao lado de "Preços lidos da planilha agora" fica verde quando
  o dado é fresco; "Cópia local" quando é o snapshot.
- Recarrega sozinho quando você volta pro app (`visibilitychange`).
- Suas escolhas (quantidades, lojas, marcações) ficam no `localStorage`; botão "Zerar" no rodapé.

## Trocar o ID da planilha

No `<script>`, no objeto **`CFG`** (topo do código):

```js
const CFG = {
  planilhaId: "1QI-vGc--zXDDX3Tvq_F1i3bZGOFiY5o0C88MHbNrDGc",  // <- troque aqui
  aba: "Enxoval Unificado",                                      // <- e o nome da aba
  ...
};
```

## Editar direto na planilha (sem mexer no código)

A leitura é dinâmica: os blocos são detectados pelos divisores da **coluna A** (`COTURNOS`,
`EQUIPAMENTOS…`) e os nomes das lojas vêm da aba **Validação de dados**. Então dá pra fazer
quase tudo só na planilha, sem tocar no HTML:

- **Mudar preço / Qtd Sugerida** de um item existente → edite a célula. (0 = a loja não vende.)
- **Adicionar / remover um produto** → adicione ou apague/esvazie a linha dentro do bloco certo.
  Pode inserir ou deletar linhas à vontade; o parser se realinha sozinho.
- **Adicionar / remover uma loja** → faça as DUAS coisas, na mesma ordem:
  1. Aba **Validação de dados**: acrescente/remova o nome da loja na coluna do bloco
     (coluna 1 = Enxoval, 2 = Coturno, 3 = Equipamentos).
  2. Aba **Enxoval Unificado**: acrescente/remova a coluna de preços correspondente, na
     **mesma posição/ordem** da lista de Validação (a 1ª loja da lista = 1ª coluna de preço, coluna D).

> A ordem das lojas na aba Validação precisa bater com a ordem das colunas de preço na aba
> Enxoval Unificado — é assim que o app liga preço → loja.

O que **não** é lido ao vivo (só sai no código/snapshot): Contatos, Pagamento, Cupom, Compras
Online e Bizus. Para mudar esses, me avise (ou edite o `SNAPSHOT` no `index.html`).

Itens femininos (aparecem só no filtro Feminino/Todos): lista `CFG.femininos`
(`Saia Passeio`, `Saia Túnica`).

## Regerar o snapshot embutido (quando os preços mudarem muito)

O snapshot é o objeto grande `const SNAPSHOT = {…}` no topo do `<script>`. Ele guarda preços,
links online, contatos, pagamento, cupom e bizus. Para atualizar:

1. Baixe a planilha em Excel: `https://docs.google.com/spreadsheets/d/{ID}/export?format=xlsx`
   (o `.xlsx` preserva os **hyperlinks** das abas Online, Contatos e Bizus, que o CSV perde).
2. Extraia preços (aba Enxoval Unificado), links (Compras Online), contatos e bizus, montando o
   mesmo formato do objeto `SNAPSHOT` atual.
3. Cole por cima do `SNAPSHOT` no arquivo.

Regra de sanidade: nenhum preço 0 deve entrar em comparação ou soma (0 = a loja não vende).
Na primeira versão da planilha, o "melhor preço item a item" dos três blocos com as quantidades
sugeridas dava **R$ 6.806** — se você mudou preços/lojas depois, esse número naturalmente muda.
