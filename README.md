# site-vendas-copiloto

Site de vendas do CopilotoVendas.ai — Copiloto de Call de Vendas para Windows.

## Arquivos

- `index.html` — página de vendas (sem vídeo): abertura, prints do programa, para quem é, como funciona, o que recebe,
  “com sinceridade” (limitações e custo da IA), preço, perguntas frequentes e o checkout Pix.
  Os prints ficam em `assets/print-*.png` (telas reais do Copiloto com falas de exemplo).
- `obrigado.html` — página de compra realizada. Confere o pagamento no Mercado Pago e só então mostra o botão do WhatsApp
  (número na variável `WHATSAPP`, no `<script>` do arquivo) com a mensagem pedindo a chave e o número do pedido.
- `..\Supabase\Funcoes\pix\index.ts` — função do Supabase que cria o Pix e consulta se foi pago
  (tudo do Supabase fica na pasta `..\Supabase`).
- `assets/` — logo, favicon e fonte Inter (licença em `assets/OFL.txt`).

A pressel (as 4 perguntas) fica em outro repositório: `site-pressel-copiloto`.

## Antes de publicar

- `index.html` e `obrigado.html`: troque `SEU-PROJETO` na variável `API` pelo endereço do projeto Supabase.
- `obrigado.html`: confira o `WHATSAPP`.
- Preço: o valor cobrado é o `PRECO` no começo do `..\Supabase\Funcoes\pix\index.ts`. Mude também o texto
  `R$ 97,00` no `index.html` para ficarem iguais.

## Pagamento (Pix pelo Mercado Pago)

1. Os botões `QUERO O MEU COPILOTO` abrem uma janela que pede nome, e-mail e WhatsApp e gera o Pix (QR Code + Copia e Cola).
2. A função `pix` anota o pedido na tabela `vendas` como `pendente`.
3. A janela pergunta à função `pix` a cada 4 segundos se o pagamento foi aprovado.
4. Aprovado → vai para `obrigado.html?id=<pedido>`, que confere de novo e libera o WhatsApp.
5. A tabela passa para `pago` quando a página confere, ou pelo aviso (webhook) que o Mercado Pago manda para
   `.../functions/v1/pix?aviso=mp` — mesmo que o cliente feche a página antes.
6. Chegou a mensagem no WhatsApp: no SQL Editor, `select public.liberar_venda('<nº do pedido>');` → gera a chave
   (só se o pedido estiver pago) e anota na venda. Mande a chave e o link da página de entrega.

O Access Token do Mercado Pago fica **só** no Supabase (segredo `MP_ACCESS_TOKEN`), nunca no HTML.
Opcional: segredo `SITE_ORIGEM` com o endereço do site, para só ele conseguir chamar a função pelo navegador.

**Tabela:** rode `..\Supabase\4-vendas.sql` no SQL Editor (por último, depois do 1, 2 e 3).
A visão `vendas_com_pressel` mostra cada venda com as respostas da pressel, quando a pessoa veio de lá
(a pressel manda o código dela no link: `?d=...`). No fim do `4-vendas.sql` há consultas prontas: quem pagou, funil e quem não pagou.

Para publicar ou atualizar a função: Supabase → Edge Functions → função `pix` → cole o conteúdo de
`..\Supabase\Funcoes\pix\index.ts` → Deploy. A opção de verificar JWT fica **desligada** (a página chama a função sem chave).

## Rodar local

```
npx --yes serve .
```

E abra o endereço que aparecer (algo como `http://localhost:3000`).
