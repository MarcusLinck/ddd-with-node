# DDD with Node

Projeto de estudo em TypeScript sobre modelagem de domínio com **Domain-Driven Design (DDD)** e princípios de **Clean Architecture**. O código representa o núcleo de um fórum de perguntas e respostas, incluindo regras de domínio, casos de uso, contratos de repositório, eventos de domínio e notificações.

> **Escopo atual:** este repositório contém a camada de domínio/aplicação e testes unitários. Não há servidor HTTP, interface web, banco de dados ou implementação de repositório de produção.

## Funcionalidades modeladas

### Fórum

- Criar, editar e excluir perguntas.
- Consultar uma pergunta pelo slug e listar perguntas recentes com paginação por página.
- Criar, editar e excluir respostas; listar respostas de uma pergunta.
- Escolher a melhor resposta de uma pergunta.
- Adicionar, consultar e excluir comentários de perguntas e de respostas.
- Associar anexos a perguntas e respostas por meio de listas de entidades.

### Notificações

- Criar notificações para um destinatário.
- Marcar uma notificação como lida, validando se ela existe e se pertence ao destinatário.
- Gerar notificações a partir de eventos do fórum:
  - uma nova resposta notifica o autor da pergunta;
  - a escolha da melhor resposta notifica o autor da resposta escolhida.

## Organização do código

```text
src/
  core/                         # Primitivas compartilhadas do domínio
    entities/                   # Entity, AggregateRoot, UniqueEntityID e WatchedList
    errors/                     # Erros de aplicação/domínio
    events/                     # Eventos de domínio e despacho de handlers
    repositories/               # Tipos compartilhados, como paginação
    either.ts                   # Resultado tipado de sucesso ou erro
  domain/
    forum/
      application/
        repositories/           # Contratos dos repositórios do fórum
        use-cases/              # Operações da aplicação e seus testes
      enterprise/
        entities/               # Perguntas, respostas, comentários e anexos
        events/                  # Eventos próprios do fórum
    notification/
      application/
        repositories/           # Contrato do repositório de notificações
        subscribers/            # Handlers que reagem aos eventos do fórum
        use-cases/              # Envio e leitura de notificações
      enterprise/entities/      # Entidade Notification
test/
  factories/                    # Factories usadas pelos testes
  repositories/                 # Repositórios em memória para testes
  utils/                        # Utilitários de teste
```

Os arquivos `*.spec.ts` ficam próximos às classes e casos de uso que testam. Os testes usam Vitest, factories e repositórios em memória; estes repositórios são apoio de teste, não uma camada de persistência pronta para produção.

## Arquitetura e fluxo

- **Enterprise/domain:** entidades, agregados, objetos de valor e eventos encapsulam conceitos do fórum e das notificações.
- **Application:** os casos de uso coordenam as operações e dependem de interfaces de repositório, sem acoplar o domínio a um banco de dados.
- **Core:** oferece tipos e comportamentos compartilhados, como identificadores únicos, listas observáveis, eventos, erros e `Either`.
- **Infraestrutura e apresentação:** devem ser fornecidas por uma aplicação que consuma este núcleo — por exemplo, adaptadores HTTP, persistência e composição das dependências.

Em geral, um caso de uso recebe suas dependências pelo construtor e executa a operação por `execute(...)`. Os resultados usam `Either<L, R>`: `left(...)` representa falha de aplicação e `right(...)` representa sucesso. Os contratos em `application/repositories` descrevem as operações que os adaptadores concretos devem implementar.

O despacho de eventos é baseado em agregados: eventos são associados ao agregado e handlers podem se inscrever em classes de evento. Os subscribers de notificação conectam eventos do fórum ao caso de uso de envio de notificações. Uma aplicação hospedeira deve fornecer as implementações de repositório e realizar a composição/ativação dos handlers conforme o fluxo adotado.

## Requisitos

- Node.js e npm instalados.
- As versões mínimas de Node.js e npm não estão declaradas no projeto.

## Instalação

Na raiz do repositório, instale as dependências:

```bash
npm install
```

## Comandos disponíveis

| Comando | Descrição |
| --- | --- |
| `npm test` | Executa a suíte de testes uma vez com Vitest. |
| `npm run test:watch` | Executa os testes em modo de observação. |
| `npm run lint` | Verifica o código TypeScript em `src/` com ESLint. |
| `npm run lint:fix` | Executa o ESLint e aplica correções automáticas quando possível. |

Não há scripts de `build`, `start` ou `dev` definidos no `package.json`; portanto, este projeto, isoladamente, não inicia uma API ou aplicação web.

## Testes

Os testes cobrem as primitivas compartilhadas, entidades, eventos, casos de uso do fórum e fluxos de notificações. Para executar todos:

```bash
npm test
```

Para acompanhar alterações durante o desenvolvimento:

```bash
npm run test:watch
```

Os imports usam o alias `@/`, configurado no `tsconfig.json` para apontar para `src/` e resolvido pelo plugin `vite-tsconfig-paths`.

## Tecnologias

- **TypeScript** com verificação estrita habilitada.
- **Vitest** para testes.
- **Vite** e `vite-tsconfig-paths` como configuração do runner e resolução do alias.
- **ESLint** com a configuração da Rocketseat.
- **Day.js** para operações com datas nas entidades.
- **Faker** para geração de dados em testes.

## Próximos passos possíveis

Para usar o núcleo em uma aplicação executável, ainda é necessário implementar adaptadores concretos para os contratos de repositório, configurar a composição de dependências e adicionar a camada de entrada desejada (por exemplo, uma API HTTP). As decisões de persistência, autenticação e transporte não estão definidas por este repositório.
