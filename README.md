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

## Ajustar os limites de bloco (se mexer no layout da planilha)

Ainda no `CFG`, cada bloco aponta as **linhas de itens** (1-indexadas, como no CSV do gviz —
a 1ª linha do CSV é o cabeçalho) e as **colunas de loja** (0-indexadas: A=0, B=1, C=2, D=3…):

```js
blocos: {
  enxoval: { ini: 2,  fim: 28, c0: 3, c1: 10, titulo: "Enxoval" },
  coturno: { ini: 32, fim: 38, c0: 3, c1: 9,  titulo: "Coturno" },
  equip:   { ini: 41, fim: 50, c0: 3, c1: 5,  titulo: "Equipamentos / Acessórios" }
}
```

Os **nomes das lojas** de cada bloco ficam em `SNAPSHOT.lojasPorBloco` (na planilha só o bloco
Enxoval tem cabeçalho nomeando as lojas; nos outros os nomes são posicionais por coluna).
Para conferir os limites, abra a URL do gviz acima no navegador e conte as linhas.

Itens femininos: lista `CFG.femininos` (`Saia Passeio`, `Saia Túnica`).

## Regerar o snapshot embutido (quando os preços mudarem muito)

O snapshot é o objeto grande `const SNAPSHOT = {…}` no topo do `<script>`. Ele guarda preços,
links online, contatos, pagamento, cupom e bizus. Para atualizar:

1. Baixe a planilha em Excel: `https://docs.google.com/spreadsheets/d/{ID}/export?format=xlsx`
   (o `.xlsx` preserva os **hyperlinks** das abas Online, Contatos e Bizus, que o CSV perde).
2. Extraia preços (aba Enxoval Unificado), links (Compras Online), contatos e bizus, montando o
   mesmo formato do objeto `SNAPSHOT` atual.
3. Cole por cima do `SNAPSHOT` no arquivo.

Teste de sanidade: o "melhor preço item a item" somando os três blocos com as quantidades
sugeridas tem que dar **R$ 6.806**. Se der outro número, o parser está pegando 0 como preço
válido ou lendo linha fora do bloco.
