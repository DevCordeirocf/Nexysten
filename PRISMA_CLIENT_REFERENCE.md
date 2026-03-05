# Referência Rápida: Nomes do Prisma Client

Este documento serve como referência para os nomes das propriedades do Prisma Client geradas automaticamente a partir do `schema.prisma`.

## Mapeamento de Modelos para Propriedades do Prisma Client

| Modelo (schema.prisma) | Propriedade no PrismaClient | Tabela no Banco | Exemplo de Uso |
| :--- | :--- | :--- | :--- |
| `TenantStore` | `tenantStore` | `tenant_stores` | `prisma.tenantStore.findMany()` |
| `Product` | `product` | `products` | `prisma.product.create({...})` |
| `ContactRequest` | `contactRequest` | `contact_requests` | `prisma.contactRequest.findFirst({...})` |

## Regra de Conversão

O Prisma Client converte automaticamente os nomes dos modelos de **PascalCase** (como definido no schema) para **camelCase** (como propriedades do cliente):

- `TenantStore` → `tenantStore`
- `Product` → `product`
- `ContactRequest` → `contactRequest`

## Exemplos de Uso Correto

### Criar um Produto

```typescript
const product = await this.prisma.product.create({
  data: {
    tenantId: 'tenant-123',
    name: 'Anel de Ouro',
    description: 'Anel em ouro 18k',
    price: 1500.00,
    images: ['url-da-imagem-1', 'url-da-imagem-2'],
    isAvailable: true,
  },
});
```

### Listar Solicitações de Contato

```typescript
const contactRequests = await this.prisma.contactRequest.findMany({
  where: { tenantId: 'tenant-123' },
  include: { product: true },
});
```

### Atualizar um Tenant

```typescript
const updatedTenant = await this.prisma.tenantStore.update({
  where: { id: 'tenant-123' },
  data: { isActive: false },
});
```

### Deletar uma Solicitação de Contato

```typescript
const deleted = await this.prisma.contactRequest.delete({
  where: { id: 'contact-request-123' },
});
```

## Operações Disponíveis

Para cada modelo, o Prisma Client oferece as seguintes operações:

| Operação | Descrição | Exemplo |
| :--- | :--- | :--- |
| `create()` | Criar um novo registro | `prisma.product.create({data: {...}})` |
| `findMany()` | Buscar múltiplos registros | `prisma.product.findMany({where: {...}})` |
| `findFirst()` | Buscar o primeiro registro que corresponde | `prisma.product.findFirst({where: {...}})` |
| `findUnique()` | Buscar por ID único | `prisma.product.findUnique({where: {id: '...'}})` |
| `update()` | Atualizar um registro | `prisma.product.update({where: {...}, data: {...}})` |
| `delete()` | Deletar um registro | `prisma.product.delete({where: {id: '...'}})` |
| `count()` | Contar registros | `prisma.product.count({where: {...}})` |

## Relacionamentos

Os relacionamentos também seguem a convenção camelCase:

### Incluir Dados Relacionados

```typescript
// Buscar um produto com seu tenant e solicitações de contato
const product = await this.prisma.product.findUnique({
  where: { id: 'product-123' },
  include: {
    tenant: true,           // Inclui dados do TenantStore
    contactRequests: true,  // Inclui todas as ContactRequest
  },
});
```

### Relacionamentos Inversos

```typescript
// Buscar um tenant com seus produtos
const tenant = await this.prisma.tenantStore.findUnique({
  where: { id: 'tenant-123' },
  include: {
    products: true,         // Inclui todos os Product
    contactRequests: true,  // Inclui todas as ContactRequest
  },
});
```

## Dicas Importantes

1. **Sempre use camelCase** ao acessar propriedades do Prisma Client, mesmo que o modelo no schema esteja em PascalCase.

2. **Verifique o schema.prisma** se tiver dúvida sobre o nome de um modelo. O Prisma Client sempre segue a regra PascalCase → camelCase.

3. **Regenere o Prisma Client** após modificar o `schema.prisma`:
   ```bash
   npx prisma generate
   ```

4. **Use o IntelliSense do seu editor** (VS Code, WebStorm, etc.) para autocompletar os nomes das propriedades. Ele mostrará exatamente quais propriedades estão disponíveis.

5. **Tipos TypeScript** são gerados automaticamente, então você terá segurança de tipo ao usar o Prisma Client:
   ```typescript
   // TypeScript saberá que 'product' é do tipo 'Product'
   const product: Product = await this.prisma.product.findUnique({...});
   ```

## Erros Comuns

### ❌ Erro: Usando PascalCase em vez de camelCase

```typescript
// ERRADO
const products = await this.prisma.Product.findMany(); // ❌ Property 'Product' does not exist

// CORRETO
const products = await this.prisma.product.findMany(); // ✅
```

### ❌ Erro: Nome de tabela em vez de nome do modelo

```typescript
// ERRADO
const products = await this.prisma.products.findMany(); // ❌ Property 'products' does not exist

// CORRETO
const products = await this.prisma.product.findMany(); // ✅
```

### ❌ Erro: Esquecendo de regenerar o Prisma Client

Se você modificar o `schema.prisma` e não executar `npx prisma generate`, o Prisma Client não saberá sobre as mudanças:

```bash
# Sempre execute após modificar o schema
npx prisma generate
```

## Regeneração do Prisma Client

Sempre que você modificar o `schema.prisma`, execute:

```bash
# Opção 1: Gerar apenas (sem aplicar migrações)
npx prisma generate

# Opção 2: Aplicar migrações e gerar (recomendado para desenvolvimento)
npx prisma migrate dev --name descricao_da_mudanca
```

## Suporte e Documentação

Para mais informações sobre o Prisma Client, consulte:
- [Documentação Oficial do Prisma](https://www.prisma.io/docs/concepts/components/prisma-client)
- [Referência de API do Prisma Client](https://www.prisma.io/docs/reference/api-reference/prisma-client-reference)
