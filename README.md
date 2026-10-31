# Recarga Fire

Site estático (`index.html`) + uma função da Vercel (`api/nick.js`) que mostra o
nick do jogador quando o cliente digita o ID.

```
recarga-fire/
├── index.html      o site inteiro (loja, checkout, Pix, suporte, FAQ)
├── hero.jpg        imagem de fundo
├── api/nick.js     função que consulta o nick
└── package.json
```

## Como publicar na Vercel

1. Suba esta pasta para um repositório no GitHub (ou use a CLI: `npx vercel` dentro da pasta).
2. Na Vercel: **Add New > Project**, escolha o repositório e clique em **Deploy**.
   Não precisa mudar nenhuma configuração (Framework Preset: *Other*).
3. Pronto: o site já funciona. Só o nick do jogador depende do próximo passo.

## Como ligar a consulta de nick

A Garena não tem API pública de consulta de nick. Você precisa de um serviço de
consulta de terceiros. Antes de escolher, confira os termos de uso dele, se é
estável e se cobra.

Na Vercel, abra **Settings > Environment Variables** e crie:

| Variável | Para que serve |
|---|---|
| `NICK_API_URL` | **Obrigatória.** URL do serviço, com `{id}` no lugar do ID. Ex.: `https://servico.exemplo/player?uid={id}&region={region}` |
| `NICK_API_REGION` | Região usada em `{region}` (padrão: `br`) |
| `NICK_API_KEY` | Chave do serviço, se ele pedir |
| `NICK_API_KEY_HEADER` | Nome do header da chave (padrão: `x-api-key`). Se o serviço usa `Authorization: Bearer ...`, coloque `Authorization` aqui e `Bearer SUA_CHAVE` em `NICK_API_KEY` |

Depois de salvar, faça um **Redeploy** para valer.

A função procura o nick sozinha nas respostas em JSON (campos `nickname`, `nick`,
`name`, `player_name`, `playerName` ou `username`, mesmo dentro de outros
objetos). Se o seu serviço usar outro nome de campo, é só incluir em `NAME_KEYS`
no começo de `api/nick.js`.

### Testar

Abra `https://SEU-SITE.vercel.app/api/nick?id=123456789` (use um ID real):

- `{"ok":true,"nickname":"..."}`: funcionando.
- `503 not_configured`: falta a variável `NICK_API_URL` (ou o redeploy).
- `404 not_found`: o serviço não achou esse ID.
- `502 unavailable`: o serviço está fora do ar ou recusou a chave.

### Como o site se comporta

- **Nick encontrado:** aparece "Jogador encontrado: NICK" no resumo e no checkout.
- **ID não encontrado (404):** o botão Confirmar fica bloqueado até corrigir o ID.
- **Serviço fora do ar ou não configurado:** o campo de nick simplesmente não
  aparece e a compra continua normalmente, para você não perder venda por causa de
  uma falha do serviço.

## Fluxo de compra

1. O cliente digita o ID, escolhe o pacote e clica em **Confirmar**.
2. Clica em **Pagar com Pix** e vai para a tela **Finalizar compra**
   (pagamento, dados e resumo).
3. Preenche nome, sobrenome e e-mail e clica em **Finalizar e ir ao pagamento**.
4. Aparece a tela com o **Pix copia e cola** e o botão de copiar.

## Importante

- **O site não recebe os pedidos nem confirma o Pix.** Ele não tem banco de dados e
  não envia os dados do cliente para lugar nenhum. Para os diamantes serem
  depositados de verdade, é preciso um serviço que avise quando o Pix cair e faça o
  envio (por exemplo, um gateway de Pix com webhook). Sem isso, a entrega é manual.
  Os dados de "Seus dados" (nome, sobrenome, e-mail) ficam só na tela do cliente.
- **Os códigos Pix e o e-mail ficam visíveis no código da página.** São dados que o
  cliente já vê, mas qualquer pessoa pode copiá-los.
- **Para mudar preços ou códigos Pix**, edite as listas `PK` e `PIX` no começo do
  `<script>` em `index.html`. Cada código Pix tem o valor fixo do pacote.
- **Imagem e nome:** `hero.jpg` é arte do Free Fire (Garena). Confira se pode usá-la
  na sua loja e troque por uma sua se preferir.
