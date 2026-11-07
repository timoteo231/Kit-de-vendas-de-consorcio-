# Kit do Vendedor de Consórcio 2.0

Aplicação Next.js (App Router) integrada ao projeto Supabase `jlvjxepgpkinddypsbre`.

## Funcionalidades incluídas

- Landing page pública e demonstração de scripts comerciais.
- Cadastro de aluno por e-mail/senha e login/logout pelo Supabase Auth.
- Perfil criado automaticamente pelo trigger do banco (papel inicial `student`).
- Dashboard do aluno com cursos liberados.
- Página de curso com aulas, links de vídeo e materiais; links privados do Storage são assinados por 5 minutos.
- Painel administrativo protegido por sessão e papel `admin`.
- Cadastro de cursos, preço em BRL, publicação/arquivamento, cadastro de aulas, upload de arquivos privados (até 25 MB), links de materiais e concessão manual de acesso por UUID.
- Vitrine pública de cursos em `/cursos` e checkout Mercado Pago Checkout Pro pela Orders API (Pix/cartão conforme elegibilidade da conta).
- Webhook de orders com validação HMAC, consulta da order na API oficial, conferência de valor/moeda e liberação idempotente do curso após processamento aprovado.
- RLS no banco e bucket privado `kit-materiais`.

## Requisitos

- Node.js 22 ou superior.
- npm.
- Projeto Supabase existente `jlvjxepgpkinddypsbre` com o schema aplicado.

## Configuração local

1. Copie `.env.example` para `.env.local`.
2. Preencha `NEXT_PUBLIC_SUPABASE_URL` e `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` com os valores de **Project URL** e **Publishable key** do Supabase. Não use `service_role` nem `sb_secret` no navegador.
3. Instale dependências: `npm install`.
4. Inicie: `npm run dev`.
5. Abra `http://localhost:3000`.

## Rotas

- `/` — página pública.
- `/cadastro` — criação de conta de aluno.
- `/login` — autenticação.
- `/dashboard` — cursos liberados.
- `/cursos` — vitrine pública e compra pelo Mercado Pago.
- `/pagamento/retorno` — retorno informativo do checkout (não concede acesso).
- `/api/checkout` — cria preferência autenticada do Mercado Pago.
- `/api/webhooks/mercadopago` — recebe e valida notificações de pagamento.
- `/aluno/cursos/[slug]` — conteúdo de um curso liberado.
- `/admin` — indicadores, contas e liberação manual de cursos.
- `/admin/cursos` — cadastro e publicação de cursos, aulas e materiais.

## Primeiro acesso administrativo

O perfil `Ivaldo` já existe no Supabase e está com `role = admin`. Entre com o e-mail cadastrado e a senha definida no Supabase Auth. Nunca coloque senha no repositório.

## Configuração do Supabase Auth

No Dashboard do Supabase, em Authentication → URL Configuration:
- Site URL: endereço local para desenvolvimento (`http://localhost:3000`) ou domínio publicado.
- Redirect URLs: adicionar `http://localhost:3000/**` e o domínio de produção com `/**`.

Confirmação de e-mail pode permanecer habilitada. O cadastro orienta o aluno a confirmar o e-mail antes de entrar.

## Publicação na Vercel

1. Envie esta pasta para um repositório GitHub.
2. Importe o repositório na Vercel (framework Next.js).
3. Configure as sete variáveis do `.env.example` na Vercel. Para homologação, use `MP_ENVIRONMENT=sandbox`, Access Token de teste e assinatura de webhook de teste. Configure os segredos somente no painel de Environment Variables, nunca no GitHub.
4. Faça deploy.
5. Atualize Site URL e Redirect URLs no Supabase para o domínio final da Vercel.
6. Teste cadastro, confirmação de e-mail, login, logout, acesso do aluno e painel admin.

## Fluxo operacional

1. Admin cria curso em `/admin/cursos` e define o preço em reais.
2. Curso começa como rascunho; publique usando o seletor de status. O curso com preço maior que zero aparece em `/cursos`.
3. Cadastre aulas e materiais HTTPS.
4. O aluno entra em `/cursos`, seleciona o curso e paga pelo Checkout Pro.
5. O webhook validado confirma o pagamento e concede acesso automaticamente. A liberação manual permanece disponível em `/admin` para suporte.
6. O aluno entra em `/dashboard` e abre o curso liberado.

## Mercado Pago — configuração obrigatória antes de cobrar

1. Crie uma aplicação em [Mercado Pago Developers](https://www.mercadopago.com.br/developers/panel/app) e habilite Checkout Pro.
2. Copie o Access Token de teste para `MP_ACCESS_TOKEN` e mantenha `MP_ENVIRONMENT=sandbox`.
3. Configure `SUPABASE_SECRET_KEY` com uma chave secreta `sb_secret_...` do projeto Supabase; ela só é usada no servidor.
4. Configure `APP_URL` com a URL HTTPS pública (Vercel).
5. Em Mercado Pago Developers → sua aplicação → Webhooks, cadastre `https://SEU-DOMINIO/api/webhooks/mercadopago`, habilite o evento **Order (Mercado Pago)** e copie a assinatura secreta para `MP_WEBHOOK_SECRET`.
6. Faça uma compra de teste e simule uma notificação do tipo Order. O retorno do navegador nunca libera acesso; somente o webhook validado e a consulta autenticada da order podem fazê-lo.
7. Depois de homologar, substitua o Access Token por credencial de produção, altere `MP_ENVIRONMENT=production` e configure a URL/assinatura de produção.

A chave privada do Mercado Pago e a chave secreta do Supabase nunca devem ir para o navegador, para variáveis `NEXT_PUBLIC_` ou para o GitHub. Consulte também `docs/mercado-pago.md` para a lista de validação e o fluxo de homologação.

## O que ainda depende de decisão/configuração externa

- **Pagamentos:** integração de código pronta, mas não ativada até inserir credenciais válidas, publicar a URL HTTPS e testar webhooks em sandbox.
- **Vídeos:** informe links HTTPS de um serviço de hospedagem/streaming escolhido. Não há upload de vídeo/transcodificação.
- **E-mail transacional:** usa o serviço de Auth do Supabase; para produção, configure SMTP próprio e teste entregabilidade.
- **Domínio:** configure domínio próprio na Vercel se desejar.
- **Leads/CRM:** telas de leads ainda não fazem parte deste escopo funcional.

## Segurança

- Nunca exponha `service_role`/`sb_secret` em variáveis `NEXT_PUBLIC_` ou no cliente.
- Papéis são lidos da tabela `profiles`; o aluno não pode definir o próprio papel.
- O painel valida sessão e papel no servidor; as tabelas mantêm RLS.
- Faça revisão de políticas e teste com uma conta de aluno antes de produção.

## Verificações

Scripts disponíveis: `npm run typecheck`, `npm run lint` e `npm run build`. Execute após `npm install`. A instalação de dependências não foi concluída neste ambiente, então build e typecheck precisam ser executados no computador/CI antes de considerar o deploy validado.
