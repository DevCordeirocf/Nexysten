# Guia de Limpeza e Setup do Nexysten MVP

Se você está recebendo erros de módulos não encontrados (`class-validator`, `@nestjs/mapped-types`, etc.) ou erros de compilação TypeScript, siga este guia para resolver o problema.

## Problema Identificado

Os erros que você está vendo indicam que:

1. **Dependências não instaladas:** O `node_modules` pode estar incompleto ou corrompido.
2. **Cache do TypeScript:** O compilador pode estar usando cache desatualizado.
3. **Prisma Client não regenerado:** O cliente Prisma pode não ter sido gerado corretamente após o clone.

## Solução Passo a Passo

### Passo 1: Limpar o Ambiente Local

Execute os seguintes comandos no diretório raiz do seu projeto Nexysten:

```bash
# Remover node_modules e arquivos de lock
rm -rf node_modules
rm -rf package-lock.json
rm -rf dist

# Limpar cache do npm (opcional, mas recomendado)
npm cache clean --force
```

**No Windows (PowerShell):**
```powershell
# Remover node_modules e arquivos de lock
Remove-Item -Recurse -Force node_modules
Remove-Item -Force package-lock.json
Remove-Item -Recurse -Force dist

# Limpar cache do npm
npm cache clean --force
```

### Passo 2: Reinstalar Dependências

```bash
npm install
```

Este comando irá:
- Instalar todas as dependências listadas no `package.json`
- Criar um novo `node_modules` limpo
- Gerar um novo `package-lock.json`

### Passo 3: Regenerar o Prisma Client

```bash
npx prisma generate
```

Este comando irá:
- Ler o schema do Prisma (`prisma/schema.prisma`)
- Gerar o cliente Prisma com os tipos corretos
- Garantir que a propriedade `contactRequest` (singular, em camelCase) esteja disponível

### Passo 4: Compilar o Projeto

```bash
npm run build
```

Este comando irá:
- Compilar o TypeScript para JavaScript
- Gerar os arquivos no diretório `dist`
- Verificar se há erros de tipo

### Passo 5: Iniciar o Servidor

Se o build foi bem-sucedido, inicie o servidor:

```bash
# Modo desenvolvimento (com watch)
npm run start:dev

# Ou modo produção
npm run start:prod
```

## Verificação de Sucesso

Você saberá que tudo está funcionando quando ver uma mensagem como:

```
╔════════════════════════════════════════════════════════════════╗
║                                                                ║
║   🚀 NEXYSTEN MVP - Sistema SaaS Multi-tenant para Joias      ║
║                                                                ║
║   ✅ Servidor iniciado com sucesso!                           ║
║   📍 URL: http://localhost:3000                             ║
║   🔒 Isolamento Multi-tenant: ATIVO                           ║
║   📊 Banco de Dados: PostgreSQL (via Prisma)                  ║
║                                                                ║
║   Endpoints disponíveis:                                       ║
║   • GET  /                  - Mensagem de boas-vindas         ║
║   • GET  /health            - Status da aplicação             ║
║   • GET  /products          - Listar produtos                 ║
║   • POST /products          - Criar produto                   ║
║   • GET  /contact-requests  - Listar solicitações             ║
║   • POST /contact-requests  - Criar solicitação               ║
║                                                                ║
║   ⚠️  Não esqueça de enviar o header:                          ║
║   X-Tenant-ID: {uuid-do-seu-tenant}                           ║
║                                                                ║
╚════════════════════════════════════════════════════════════════╝
```

## Troubleshooting Adicional

### Erro: "Cannot find module 'class-validator'"

**Solução:** Execute `npm install` novamente e certifique-se de que o `node_modules` foi criado corretamente.

```bash
npm install
npx prisma generate
npm run build
```

### Erro: "Property 'contactRequest' does not exist on type 'PrismaService'"

**Solução:** Regenere o Prisma Client:

```bash
npx prisma generate
```

Se o erro persistir, verifique se o `schema.prisma` tem a definição correta do modelo `ContactRequest`:

```prisma
model ContactRequest {
  id            String   @id @default(uuid())
  tenantId      String
  productId     String
  customerName  String
  customerEmail String
  customerPhone String?
  message       String?
  status        String   @default("PENDING")
  createdAt     DateTime @default(now())

  tenant  TenantStore @relation(fields: [tenantId], references: [id])
  product Product     @relation(fields: [productId], references: [id])

  @@index([tenantId])
  @@index([productId])
  @@map("contact_requests")
}
```

### Erro: "Cannot find module '@nestjs/mapped-types'"

**Solução:** Instale a dependência manualmente:

```bash
npm install @nestjs/mapped-types
```

### Erro: "DATABASE_URL is not set"

**Solução:** Certifique-se de que o arquivo `.env` existe e contém a variável `DATABASE_URL`:

```bash
# Copiar o arquivo de exemplo
cp .env.example .env

# Editar o arquivo .env e configurar a URL do banco de dados
# DATABASE_URL="postgresql://user:password@localhost:5432/nexysten_db?schema=public"
```

### Erro: "Connection refused" ao conectar no PostgreSQL

**Solução:** Certifique-se de que o PostgreSQL está rodando:

```bash
# Verificar se o container Docker está rodando
docker ps

# Se não estiver, inicie o container
docker run --name nexysten-db \
  -e POSTGRES_USER=user \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=nexysten_db \
  -p 5432:5432 \
  --network host \
  -d postgres:16
```

## Resumo dos Comandos

Para uma limpeza completa e reinstalação rápida, execute:

```bash
rm -rf node_modules package-lock.json dist && \
npm install && \
npx prisma generate && \
npm run build && \
npm run start:dev
```

Isso fará:
1. Remover `node_modules`, `package-lock.json` e `dist`
2. Reinstalar todas as dependências
3. Regenerar o Prisma Client
4. Compilar o projeto
5. Iniciar o servidor em modo desenvolvimento

## Próximas Etapas

Após resolver os erros de compilação, você pode:

1. **Testar os endpoints** usando Postman ou cURL
2. **Criar um tenant** para começar a usar o sistema
3. **Criar produtos** (joias) para o seu tenant
4. **Testar o isolamento de dados** entre tenants

Se ainda tiver problemas, verifique se:
- Node.js versão 18+ está instalado (`node --version`)
- npm versão 9+ está instalado (`npm --version`)
- PostgreSQL está acessível na URL configurada no `.env`
- O arquivo `.env` está configurado corretamente
