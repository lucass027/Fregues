# Freguês

Assistente de pedidos com IA para mercadinhos de bairro. O cliente pede por texto ou áudio, e a IA consulta preço e estoque reais da loja antes de fechar a entrega.

Projeto Integrador IV — Aplicações de Inteligência Artificial · FAESA · 2026/2
Professor: M.Sc. Howard Cruz Roatti

## Problema

Mercadinhos de bairro recebem pedidos de entrega manualmente pelo WhatsApp ou pagam comissões de 12% a 23% em marketplaces. Parte dos clientes, em especial idosos, tem dificuldade com aplicativos. O Freguês permite pedir conversando, com preço e estoque sempre vindos do banco da loja.

- Área temática: Trabalho e Empreendedorismo
- ODS: 8 e 9
- Comunidade: donos de mercadinhos da Grande Vitória e seus clientes com pouca familiaridade digital

## Status das entregas

| Checkpoint | Prazo | Entrega | Status |
| --- | --- | --- | --- |
| C1 — Relatório | 18/09/2026 | Escopo, metodologia, comunidade, indicadores e repositório inicial | Entregue |
| C2 — Protótipo | 30/10/2026 | Vídeo com a IA funcionando, primeiros testes e eval (30 casos) | Em andamento |
| C3 — MVP | 04/12/2026 | MVP publicado, eval completo, testes com usuários e análise de impacto | Planejado |

## Stack

| Camada | Tecnologia |
| --- | --- |
| Backend | C# / ASP.NET Core |
| Frontend | React |
| LLM | Groq — `openai/gpt-oss-120b` com tool calling |
| Áudio | Groq — `whisper-large-v3-turbo` |
| Proteção contra prompt injection | Groq — Llama Prompt Guard 2 |
| Banco | PostgreSQL (Neon, plano gratuito) com `pg_trgm` |
| Testes | xUnit, WebApplicationFactory, Testcontainers, Vitest, React Testing Library |
| Deploy | Render (API em Docker e frontend estático) |
| CI | GitHub Actions |

## Como a IA funciona

O LLM nunca acessa o banco diretamente. Ele só pode pedir ferramentas, que o backend executa e valida:

| Ferramenta | O que faz |
| --- | --- |
| `buscar_produtos(termo)` | Busca produtos por nome, marca ou apelido, tolerando erros de digitação |
| `consultar_produto(id)` | Retorna preço e estoque atuais |
| `adicionar_item(id, qtd)` | Adiciona ao carrinho, validando estoque |
| `remover_item(id)` | Remove do carrinho |
| `ver_carrinho()` | Itens, subtotal e frete calculados no backend |
| `calcular_frete(bairro)` | Taxa de entrega por bairro |
| `consultar_politicas(pergunta)` | Horário, formas de pagamento e área de entrega |
| `confirmar_pedido()` | Grava o pedido após confirmação explícita do cliente |

Preço, estoque e total são sempre calculados em C#. O pedido só é gravado depois que o cliente confirma o resumo.

## Estrutura do repositório

```
fregues/
├── .github/workflows/            CI (build e testes em cada pull request)
├── data/seed/                    catálogo simulado do mercadinho
├── docs/
│   ├── arquitetura/              diagramas
│   ├── c1-relatorio/             relatório da entrega C1
│   └── decisoes/                 registros de decisões técnicas (ADRs)
├── eval/
│   ├── Fregues.Eval/             runner do eval (console C#)
│   ├── casos/                    conversas de teste anotadas
│   └── relatorios/               resultados de cada execução
├── prompts/desenvolvimento/      prompts de IA usados no desenvolvimento (seção 11 do edital)
├── src/
│   ├── backend/
│   │   ├── Fregues.Api/          endpoints HTTP
│   │   ├── Fregues.Domain/       regras de negócio (carrinho, frete, estoque, pedido)
│   │   ├── Fregues.Agent/        orquestração do LLM e ferramentas
│   │   │   └── Prompts/          prompt de sistema versionado
│   │   └── Fregues.Infrastructure/  banco, cliente do Groq e busca
│   └── frontend/src/
│       ├── chat/                 chat do cliente
│       └── painel/               painel do lojista
└── tests/
    ├── Fregues.Domain.Tests/
    ├── Fregues.Agent.Tests/
    └── Fregues.Api.IntegrationTests/
```

## Como rodar localmente

Disponível a partir do C2. Requisitos previstos:

- .NET SDK
- Node.js
- Docker (para os testes de integração)
- Chave de API do Groq

Variáveis de ambiente:

| Variável | Uso |
| --- | --- |
| `GROQ_API_KEY` | Chave da API do Groq |
| `ConnectionStrings__Fregues` | Conexão com o PostgreSQL |

Nunca faça commit de chaves ou strings de conexão.

## Testes e avaliação da IA

- Testes unitários e de integração: pasta `tests/`
- Eval do componente de IA: pasta `eval/`, com métricas de acerto de produto, ferramenta correta, alucinação de preço, recusa correta, robustez a prompt injection, latência e custo por pedido

Por causa dos limites diários do plano gratuito do Groq, o eval completo roda sob demanda, não a cada commit.

## Fluxo de trabalho

- `main` protegida; mudanças entram só por pull request revisado
- Uma branch por issue: `feature/<numero>-<descricao>`
- Mensagens de commit no padrão Conventional Commits
- Quadro de tarefas no GitHub Projects

## Uso de IA no desenvolvimento

Conforme a seção 11 do edital, todo trabalho feito com auxílio de IA é registrado em `prompts/desenvolvimento/`, com o prompt utilizado, a data, o integrante e o que foi aproveitado.

## Equipe

| Integrante | Papel |
| --- | --- |
| [Integrante 1] | Coordenação e documentação |
| [Integrante 2] | Backend e banco |
| [Integrante 3] | IA e eval |
| [Integrante 4] | Frontend |
| [Integrante 5] | Qualidade e DevOps |
