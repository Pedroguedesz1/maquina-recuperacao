# Máquina de Recuperação de Clientes · Click

Produto de entrada da Click (R$ 49,90/mês). O restaurante assina, importa a planilha de clientes ou pedidos e descobre quem parou de pedir. Depois prepara mensagens de recuperação no WhatsApp.

## Arquivos

| Arquivo | O que é |
|---|---|
| `index.html` | LP de venda + demonstração (`index.html#maquina`) com dados fictícios |
| `app.html` | **A ferramenta real**, entregue depois do pagamento |
| `raio-x.html` | **Raio-X grátis**: cadastro + importação, mostra os números reais com nomes e telefones borrados e leva para a assinatura |
| `modelo-clientes.csv` | Planilha modelo para quem não tem sistema |

## Fluxo

Anúncio → `index.html` → botão de assinatura → checkout do gateway → **página de obrigado / e-mail de entrega com o link do `app.html`** → cliente importa a planilha → painel com quem sumiu → mensagens no WhatsApp.

## Como o app funciona

- Aceita **Excel (.xlsx)** e **CSV** (separado por `;` ou `,`, em UTF-8 ou no padrão do Excel brasileiro).
- Aceita **uma linha por cliente** (com data do último pedido) ou **uma linha por pedido**. Nesse caso, junta os pedidos pelo telefone, ou pelo nome quando não há telefone.
- Identifica as colunas sozinho, e o dono confirma numa tela de conferência.
- Faixas: ativo (até 14 dias), atenção (15–29), inativo (30–59), perdido (60+). Também mostra "pediram só 1 vez" quando há número de pedidos.
- Filtros, busca, ordenação, seleção, "copiar telefones", "exportar lista" (CSV), campanha com mensagem editável (cupom, benefício, prazo) e botão que abre o WhatsApp **no número do cliente** com a mensagem pronta.
- Marca quem já foi contatado, e a marcação continua depois de reimportar a base.
- **Nomes e telefones dos clientes ficam só no navegador do restaurante** (`localStorage`). A Click recebe apenas números agregados (veja "Banco de dados"). Trocou de aparelho ou limpou o navegador? É só importar de novo.

## Configuração

`index.html`, no fim do arquivo:

```js
var CONFIG = { checkoutUrl: "", whatsappClick: "", leadWebhook: "",
  supabaseUrl: "https://msrilojbncclwksphjjt.supabase.co", supabaseKey: "sb_publishable_..." };
```

`app.html`, no fim do arquivo:

```js
var CONFIG = { whatsappClick: "", modeloUrl: "modelo-clientes.csv",
  supabaseUrl: "https://msrilojbncclwksphjjt.supabase.co", supabaseKey: "sb_publishable_..." };
```

## Banco de dados (Supabase)

- `leads`: cadastros da LP (demonstração e "Conhecer a Click completa") e do **Raio-X** (`tipo = raio_x`, com tamanho da base, clientes sumidos e receita parada), com UTMs.
- `uso_app`: a cada importação e campanha, só números agregados (total de clientes, ativos, 15/30/60+ dias, receita parada, mensagens, sistema de origem). Nenhum nome ou telefone de cliente final é enviado.
- A chave publicável só permite **inserir** (RLS). Para ler os dados, use o painel do Supabase (Table Editor).

## Limitações desta versão (MVP)

- **Acesso:** o `app.html` não tem login. Quem tiver o link consegue abrir, mas cada pessoa só vê os dados que ela mesma importou. O link não aparece na LP e a página pede aos buscadores para não ser indexada. Login de verdade (ex.: Supabase) fica para a fase 2.
- **Disparo:** manual, um clique por cliente, pelo WhatsApp do próprio restaurante. A API oficial fica para depois.
- **Planilhas .xls antigas:** não são aceitas. O app pede para salvar como .xlsx ou CSV.

## Eventos (window.dataLayer)

App: `app_open`, `import_start`, `import_sample`, `import_done`, `template_download`, `filter_use`, `recover_click`, `campaign_prepared`, `campaign_whatsapp_open`, `copy_phones`, `export_list`, `click_full_interest`, `click_whatsapp`.
LP: ver os eventos listados no código (`page_view`, `cta_click`, `calculator_used`, `lead_captured`, `checkout_click`...).
