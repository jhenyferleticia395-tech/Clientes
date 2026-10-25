# JLC Soluções — catálogo para Cloudflare Workers + Supabase

Projeto completo com o visual da JLC, página inicial por catálogos, páginas individuais, pesquisa, filtros, pedido pelo WhatsApp, capas dos catálogos, personalização opcional por produto, formulário de projetos 3D, compartilhamento e painel em `/admin`.

## O que já está configurado

- Worker: `jlc-solucoes` (novo Worker; não substitui `3d-dallas`).
- WhatsApp: +55 14 99709-6646.
- Mesmo projeto Supabase registrado na configuração do projeto anterior.
- As variáveis `SUPABASE_URL` e `SUPABASE_PUBLISHABLE_KEY` estão no `wrangler.jsonc`; confira no seu painel Supabase se continuam atuais. A chave incluída é pública, não é uma service-role.
- Tabelas exclusivas: `jlc_collections`, `jlc_products`, `jlc_admins`.
- Fotos exclusivas: bucket privado `jlc-fotos`.
- Banco protegido com RLS e login Supabase no painel. O público pode navegar sem ChatGPT e sem fazer login.

**Usar o mesmo projeto não significa mostrar os mesmos produtos.** A JLC começa vazia. Os itens e as fotos do site Dallas não são copiados nem alterados. Os registros da prévia no ChatGPT também não são transferidos por este pacote.

## 1. Preparar o Supabase (uma vez)

1. Abra o mesmo projeto Supabase usado no outro site.
2. Em **SQL Editor**, execute o conteúdo de `supabase/01-jlc-estrutura.sql`.
3. Confira o resultado final: devem aparecer as contas autorizadas que já existem e têm e-mail confirmado.
4. Em **Authentication > Users**, confirme que existe uma conta sua com senha definida e e-mail confirmado. Você pode usar a conta já existente do outro site.
5. O SQL e o código autorizam inicialmente `jhenyferleticia395@gmail.com` e `ruanpereira145@gmail.com`. Se a conta não existir, crie/confirme pelo Supabase e execute o SQL de novo. A senha é a do Supabase, não a do ChatGPT.
6. Se quiser outro administrador, adicione o e-mail em `lib/session.mjs` E o `user_id` correspondente em `jlc_admins`, pelo SQL Editor. Não habilite escrita pública nas tabelas.

O script cria recursos JLC; não contém operações nas tabelas da Dallas. Não execute scripts antigos de limpeza, `DROP TABLE` ou reset do banco.

## 2. Publicar pelo computador

Requisito: Node.js 22.13 ou superior e uma conta Cloudflare.

Na pasta descompactada:

```bash
npm install
npm run test
npm run check
npm run build
npx wrangler login
npm run deploy
```

A publicação usa `dist/server/wrangler.json`, gerado pelo build. Não envie só `worker.ts` pelo editor do painel: o site também precisa dos arquivos gerados do frontend.

Este ZIP também contém `dist/` do build validado. Depois de instalar as dependências, `npm run deploy` permite publicar esse build sem refazê-lo. Se você editar o código, execute o build de novo antes do deploy.

## 3. Publicar pelo GitHub / Cloudflare Workers Builds

1. Crie um repositório novo para a JLC e envie os arquivos da pasta do projeto. Não use o repositório da Dallas para substituir o site dela.
2. Na Cloudflare, crie um **Worker** conectado ao repositório.
3. Configure a instalação com `npm install`, o build com `npm run build` e o deploy com `npm run deploy`.
4. Use Node 22 ou superior no ambiente de build.
5. Confira as duas variáveis Supabase no `wrangler.jsonc` antes de publicar.
6. Rode primeiro o SQL no Supabase; ele não é aplicado automaticamente pela Cloudflare.

## 4. Entender o endereço

O endereço típico é:

`https://jlc-solucoes.SUBDOMINIO-DA-SUA-CONTA.workers.dev`

Se publicar na mesma conta da Dallas, o subdomínio da conta tende a ser o mesmo usado por ela, e o nome do novo Worker será `jlc-solucoes`. O link definitivo é o retornado pelo deploy. Este pacote não registra nem reserva `jlc-solucoes.workers.dev`. Não altere o subdomínio geral da conta apenas para encurtar este link, pois isso pode afetar outros Workers.

Depois de publicar, abra `/admin` no novo endereço.

## Como usar o painel

1. Em **Catálogos**, crie o catálogo, descreva-o e envie a imagem de capa. Você pode editar, ocultar ou excluir um catálogo vazio. Se não enviar capa, o site usa uma foto de produto como alternativa.
2. Em **Produtos**, escolha o catálogo, adicione fotos, descrição, preço e disponibilidade. Marque a opção de personalização quando desejar.
3. Para o cliente, nome/texto, cor, tamanho e observações são opcionais e acompanham o pedido enviado ao WhatsApp. A mesma personalização se aplica a todas as unidades daquele produto no pedido. A confirmação de viabilidade e preço acontece na conversa.
4. **Solicitar projeto 3D** prepara um briefing no WhatsApp; não cria um pedido nem envia automaticamente a mensagem. Fotos e arquivos podem ser enviados depois na conversa.
5. **Compartilhar produto/catálogo** usa o compartilhamento do dispositivo, cópia do link ou um campo manual com opção de WhatsApp.
6. **Duplicar** e ordenação manual ainda não fazem parte desta versão.

## Verificação após publicar

- Abra o site sem login: o catálogo deve carregar; `/admin` deve pedir e-mail e senha.
- Crie um catálogo e produto de teste no painel, envie uma capa e uma foto, recarregue e confira que persistiram.
- Teste a personalização, o total, o compartilhamento e o texto preparado para WhatsApp.
- Oculte um catálogo e confirme, em janela anônima, que ele e suas peças não são exibidos.
- O site Dallas deve continuar exibindo seus dados.

## Validação deste pacote

Testes de sessão e autorização, verificação TypeScript e build de produção aprovados no ambiente de desenvolvimento. O SQL foi preparado para o Supabase; não foi aplicado ao seu projeto remoto, e o deploy na sua conta Cloudflare não foi executado. É necessário concluir os passos acima para validar a integração real.

## Estrutura principal

- `app/shop.tsx`: vitrine, pedido, personalização e solicitação de projetos.
- `app/admin/`: login e painel.
- `app/api/`: catálogo, checkout e uploads protegidos.
- `lib/session.mjs`: validação de sessão, contas autorizadas e origem das alterações.
- `lib/supabase.ts`: acesso ao projeto Supabase e bucket JLC.
- `supabase/01-jlc-estrutura.sql`: tabelas, políticas, bucket e administradores.
- `worker.ts`, `vite.config.ts`, `wrangler.jsonc`: publicação Cloudflare.
