# Boas Práticas de Uso do Claude Code

> Guia prático para desenvolvedores que querem extrair o máximo do Claude Code com disciplina, segurança e resultados previsíveis.

---

## Sumário

- [O Que é o Claude Code?](#o-que-é-o-claude-code)
- [Configuração Inicial](#configuração-inicial)
- [CLAUDE.md — A Peça Central](#claudemd--a-peça-central)
- [Boas Práticas de Prompt](#boas-práticas-de-prompt)
- [Gestão de Contexto](#gestão-de-contexto)
- [Fluxo de Trabalho Recomendado](#fluxo-de-trabalho-recomendado)
- [Segurança e Permissões](#segurança-e-permissões)
- [Integração com SDD](#integração-com-sdd)
- [Padrões a Evitar](#padrões-a-evitar)
- [Exemplos Práticos](#exemplos-práticos)
- [Recursos e Referências](#recursos-e-referências)

---

## O Que é o Claude Code?

**Claude Code** é a CLI oficial da Anthropic para o modelo Claude. Ele permite interagir com o Claude diretamente no terminal, integrando-se ao seu ambiente de desenvolvimento para ler, editar, criar e executar código com contexto real do projeto.

```bash
# Instalação
npm install -g @anthropic-ai/claude-code

# Iniciar uma sessão
claude
```

> **Diferencial:** ao contrário de chatbots web, o Claude Code tem acesso real ao sistema de arquivos, pode executar comandos e mantém contexto do repositório inteiro.

---

## Configuração Inicial

### 1. Autenticação

```bash
# Via Anthropic (conta pessoal)
claude

# Via AWS Bedrock
claude --bedrock

# Via Google Vertex AI
claude --vertex
```

### 2. Estrutura de Arquivos de Configuração

```
projeto/
├── CLAUDE.md              ← contexto global do projeto (versionado)
├── .claude/
│   ├── settings.json      ← configurações locais (não versionar)
│   └── commands/          ← comandos slash personalizados
│       └── deploy.md
└── src/
    └── ...
```

### 3. Permissões por Modo

| Modo | Descrição | Indicado Para |
|---|---|---|
| **Default** | Aprovação manual para ações destrutivas | Uso geral, aprendizado |
| **Auto-approve** | Aprova automaticamente ferramentas seguras | Fluxos automatizados |
| **Plan Mode** | Apenas planeja, não executa | Revisão de estratégia |

---

## CLAUDE.md — A Peça Central

O arquivo `CLAUDE.md` é a **memória permanente** do Claude Code para o seu projeto. É lido automaticamente em cada sessão e define o contexto que guia todas as interações.

### Estrutura Recomendada

```markdown
# CLAUDE.md

## Visão Geral do Projeto
Breve descrição do sistema, seu propósito e contexto de negócio.

## Stack Tecnológica
- Linguagem: Python 3.11
- Framework: FastAPI
- Banco de Dados: PostgreSQL 15 + SQLAlchemy
- Infra: Docker + AWS ECS
- Testes: pytest + coverage

## Estrutura de Diretórios
src/
  api/        → rotas e controllers
  core/       → lógica de negócio
  models/     → modelos ORM
  schemas/    → validação Pydantic
  tests/      → testes automatizados

## Padrões de Código
- Type hints obrigatórios em todas as funções
- Docstrings no formato Google Style
- Cobertura mínima de testes: 80%
- Linting: ruff + black (linha max: 88 chars)

## Convenções de Nomenclatura
- Variáveis e funções: snake_case
- Classes: PascalCase
- Constantes: UPPER_SNAKE_CASE
- Arquivos: snake_case.py

## Comandos Úteis
- Rodar testes: `make test`
- Lint: `make lint`
- Build: `make build`
- Dev local: `make dev`

## Restrições Importantes
- NÃO instale dependências sem perguntar
- NÃO faça push direto para main/master
- NÃO use `print()` para logging — use `logging.getLogger()`
- NÃO exponha secrets em código ou logs
- Sempre rode os testes antes de concluir uma tarefa

## Decisões Arquiteturais
- JWT escolhido sobre sessions (sistema stateless)
- Repository pattern para isolamento de banco de dados
- Async/await em todos os endpoints de I/O
```

### Boas Práticas para o CLAUDE.md

- **Seja específico, não genérico** — "use ruff para linting" é melhor que "siga boas práticas"
- **Inclua comandos reais** — o Claude pode executar `make test` se souber que existe
- **Liste o que NÃO fazer** — restrições explícitas evitam surpresas
- **Mantenha atualizado** — CLAUDE.md desatualizado é pior que nenhum
- **Versione no repositório** — toda a equipe deve compartilhar o mesmo contexto

---

## Boas Práticas de Prompt

### Seja Específico sobre o Escopo

```
# Vago (evitar)
"Melhora o código de autenticação"

# Específico (recomendado)
"No arquivo src/api/auth.py, a função login() não está
validando se o usuário está ativo antes de emitir o token.
Adicione essa validação após a verificação de senha,
retornando 403 com mensagem 'account_disabled' se inativo."
```

### Referencie Arquivos e Linhas

```
"Leia o arquivo src/core/user_service.py e explique
o que a função create_user() faz, especialmente
a lógica de validação entre as linhas 45-72."
```

### Forneça Contexto de Negócio

```
"Estamos construindo um sistema de pedidos para restaurantes.
A função calculate_total() em src/core/order.py precisa
considerar que itens do cardápio podem ter adicionais
(extras) com preços variáveis. Adicione suporte a isso
mantendo compatibilidade com pedidos existentes."
```

### Use Restrições Explícitas

```
"Refatore a função process_payment() para lidar com
timeouts do gateway. Restrições:
- NÃO mude a assinatura da função (outros módulos dependem dela)
- Use a biblioteca 'tenacity' que já está no requirements.txt
- Máximo 3 tentativas com backoff exponencial
- Mantenha os testes existentes passando"
```

### Peça Confirmação antes de Ações Destrutivas

```
"Antes de deletar qualquer arquivo ou fazer alterações
irreversíveis, liste o que você pretende fazer e
aguarde minha confirmação."
```

---

## Gestão de Contexto

### O Problema da Janela de Contexto

O Claude Code tem uma janela de contexto finita. Em sessões longas ou repositórios grandes, o contexto pode se degradar, levando a respostas inconsistentes.

### Estratégias de Gestão

**1. Compactar o contexto quando necessário**
```
/compact
```
Use quando a sessão está longa e você quer liberar espaço sem perder o histórico essencial.

**2. Começar novas sessões para tarefas independentes**

Não tente resolver tudo em uma única sessão. Sessões focadas produzem resultados melhores.

**3. Usar o CLAUDE.md como âncora de contexto**

O CLAUDE.md é lido no início de cada sessão — coloque nele tudo que precisa ser lembrado permanentemente.

**4. Referenciar explicitamente o que é relevante**

```
"Com base no CLAUDE.md deste projeto e no arquivo
src/models/user.py (leia-o agora), implemente..."
```

### Hierarquia de Contexto

```
CLAUDE.md (projeto)           ← sempre presente
    └── Spec da feature       ← injetada no prompt
        └── Arquivo relevante ← lido sob demanda
            └── Prompt atual  ← tarefa específica
```

---

## Fluxo de Trabalho Recomendado

### Para Novas Features

```
1. Escreva a spec da feature (SPEC.md ou inline no prompt)
      ↓
2. Peça ao Claude para REVISAR a spec antes de implementar
      ↓
3. Aprove ou ajuste a spec
      ↓
4. Peça a implementação referenciando a spec
      ↓
5. Revise o código gerado
      ↓
6. Peça os testes automatizados
      ↓
7. Execute os testes e corrija falhas
```

### Para Correção de Bugs

```
1. Descreva o comportamento esperado vs. atual
      ↓
2. Forneça o stack trace ou log de erro completo
      ↓
3. Indique o arquivo/função suspeita
      ↓
4. Peça diagnóstico ANTES da correção
      ↓
5. Aprove a abordagem de correção
      ↓
6. Aplique e teste
```

### Para Refatoração

```
1. Defina o objetivo da refatoração
2. Estabeleça o que NÃO deve mudar (API pública, contratos)
3. Peça um plano antes da execução (/plan)
4. Refatore em partes pequenas e verificáveis
5. Rode os testes após cada parte
```

### Usando o Plan Mode

```bash
# Entre no modo de planejamento
/plan

# O Claude descreve o que fará sem executar nada
# Revise o plano, faça ajustes
# Saia do modo de planejamento para executar
```

---

## Segurança e Permissões

### Princípio do Menor Privilégio

Conceda ao Claude Code apenas as permissões necessárias para a tarefa atual. Evite aprovar automaticamente todas as ferramentas.

### Arquivos Sensíveis — Proteja com .gitignore

```gitignore
# Nunca versione estes arquivos
.env
.env.*
*.pem
*.key
secrets/
.claude/settings.json   ← configurações locais da sessão
```

### O Que Revisar Antes de Aprovar

| Ação | Nível de Risco | Revisar com Atenção |
|---|---|---|
| Leitura de arquivos | Baixo | Verificar se não lê secrets |
| Edição de arquivos | Médio | Verificar o diff completo |
| Execução de comandos | Alto | Entender exatamente o que roda |
| Deleção de arquivos | Muito alto | Confirmar que há backup/git |
| Push para repositório | Muito alto | Revisar commits antes |
| Instalação de pacotes | Alto | Verificar a fonte e versão |

### Configuração de Allowlist

```json
// .claude/settings.json
{
  "permissions": {
    "allow": [
      "Bash(make:*)",
      "Bash(pytest:*)",
      "Bash(git status)",
      "Bash(git diff)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(git push --force:*)"
    ]
  }
}
```

---

## Integração com SDD

O Claude Code é a ferramenta ideal para implementar **Spec-Driven Development (SDD)**. O `CLAUDE.md` atua como camada de contexto permanente, enquanto as specs de feature guiam cada geração de código.

### Estrutura de Repositório SDD + Claude Code

```
projeto/
├── CLAUDE.md                  ← contexto global
├── specs/
│   ├── auth/
│   │   ├── SPEC.md            ← requisitos da feature
│   │   └── ACCEPTANCE.md      ← critérios de aceite
│   └── products/
│       ├── SPEC.md
│       └── DATA_MODEL.md
├── .claude/
│   └── commands/
│       ├── new-spec.md        ← /new-spec: gera template de spec
│       └── implement.md       ← /implement: gera código a partir da spec
└── src/
    └── ...
```

### Comandos Slash Personalizados para SDD

**`.claude/commands/implement.md`**
```markdown
Leia o arquivo specs/$ARGUMENTS/SPEC.md e implemente
a feature seguindo as convenções do CLAUDE.md.

Antes de escrever código:
1. Liste os requisitos que serão implementados
2. Confirme a abordagem arquitetural
3. Implemente um requisito por vez
4. Ao final, gere os testes baseados no ACCEPTANCE.md
```

**Uso:**
```bash
/implement auth
```

### Fluxo SDD com Claude Code

```
Você escreve SPEC.md
       ↓
/plan → Claude revisa e planeja
       ↓
Você aprova o plano
       ↓
/implement <feature> → Claude gera código guiado pela spec
       ↓
Claude roda os testes automaticamente
       ↓
Você revisa o diff e aprova
```

---

## Padrões a Evitar

### Anti-padrão 1 — O Prompt Infinito

```
# Evitar
"Cria todo o sistema de e-commerce completo com
autenticação, catálogo, carrinho, pagamento,
notificações e relatórios."

# Preferir
"Implemente o endpoint POST /auth/login conforme
o SPEC.md em specs/auth/SPEC.md."
```

**Por quê:** tarefas grandes geram código inconsistente e difícil de revisar.

### Anti-padrão 2 — Aceitar sem Revisar

Nunca aprove uma edição de arquivo sem ler o diff completo. O Claude pode introduzir mudanças sutis fora do escopo solicitado.

### Anti-padrão 3 — Sessões Eternas

Sessões muito longas degradam a qualidade das respostas. Use `/compact` ou inicie uma nova sessão para tarefas não relacionadas.

### Anti-padrão 4 — Ignorar os Testes

```
# Evitar
"Implementa e já pode ir para a próxima feature."

# Preferir
"Implementa e depois rode `make test` para confirmar
que nada quebrou."
```

### Anti-padrão 5 — CLAUDE.md Genérico

```markdown
# Ruim — muito vago
## Padrões
Siga boas práticas de código limpo.

# Bom — específico e acionável
## Padrões
- Use ruff para linting: `ruff check src/`
- Formate com black: `black src/ --line-length 88`
- Type hints obrigatórios: verifique com mypy
- Todo endpoint deve ter docstring com descrição, args e returns
```

### Anti-padrão 6 — Secrets no Prompt

```
# NUNCA faça isso
"Conecta no banco com host=prod.db.myapp.com,
user=admin, password=SuperSecret123"

# Use variáveis de ambiente referenciadas no código
"Conecta no banco usando as variáveis DATABASE_URL
definidas no .env (não incluído aqui por segurança)."
```

---

## Exemplos Práticos

### Exemplo 1 — Explorar o Codebase

```
"Leia a estrutura do projeto e me explique:
1. Como o fluxo de autenticação funciona (de ponta a ponta)
2. Quais são os modelos de dados principais
3. Como os testes estão organizados
Não modifique nenhum arquivo."
```

### Exemplo 2 — Adicionar Validação

```
"No arquivo src/api/products.py, a rota POST /products
não valida se o preço é positivo. Adicione validação
usando o schema Pydantic existente em src/schemas/product.py.
Adicione também um teste em tests/test_products.py
para cobrir o caso de preço negativo."
```

### Exemplo 3 — Refatoração Segura

```
"A função send_email() em src/core/notifications.py
está sendo chamada de forma síncrona e bloqueando o
event loop. Refatore para async/await.

Restrições:
- Mantenha a mesma assinatura da função
- Use o cliente SMTP já configurado em src/config/mail.py
- Não quebre os testes existentes em tests/test_notifications.py
- Rode `make test` ao final para confirmar"
```

### Exemplo 4 — Code Review

```
"Revise o arquivo src/core/payment.py como um sênior
engineer faria. Aponte:
1. Problemas de segurança
2. Possíveis race conditions
3. Tratamento de erros inadequado
4. Oportunidades de melhoria de performance

Não modifique o arquivo ainda — apenas liste os problemas."
```

### Exemplo 5 — Geração de Testes

```
"Com base na implementação em src/core/order.py,
gere testes pytest abrangentes para a função
calculate_total(). Cubra:
- Pedido com um único item
- Pedido com múltiplos itens
- Pedido com desconto aplicado
- Pedido vazio (deve lançar ValueError)
- Preços com casas decimais

Use fixtures do pytest quando apropriado."
```

---

## Recursos e Referências

- [Documentação Oficial do Claude Code](https://docs.anthropic.com/claude-code)
- [Anthropic — Model Context Protocol (MCP)](https://modelcontextprotocol.io)
- [GitHub Spec Kit — Templates SDD](https://github.com/anthropics/spec-kit)
- [Curso: Engenharia de Software Guiada por Intenção e SDD — Data Science Academy](https://www.datascienceacademy.com.br)
- [OpenAPI Specification](https://spec.openapis.org)

---

## Contribuindo

Sugestões de novas boas práticas, anti-padrões ou exemplos são bem-vindas via Pull Request. Por favor:

1. Abra uma issue descrevendo a prática sugerida
2. Inclua um exemplo concreto (antes/depois)
3. Explique o porquê da recomendação

---

*Mantido com apoio do Claude Code — produzido seguindo os princípios de Spec-Driven Development (SDD).*
