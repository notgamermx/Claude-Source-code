# Claude Code v2.1.88 — Análise do Código Fonte

> **Aviso de Isenção de Responsabilidade**: Todo o código fonte neste repositório é propriedade intelectual da **Anthropic e Claude**. Este repositório é fornecido estritamente para pesquisa técnica, estudo e intercâmbio educacional entre entusiastas. **O uso comercial é estritamente proibido.** Nenhum indivíduo, organização ou entidade pode usar este conteúdo para fins comerciais, atividades com fins lucrativos, atividades ilegais ou qualquer outro cenário não autorizado.

> Extraído do pacote npm `@anthropic-ai/claude-code` versão **2.1.88**.
> O pacote publicado contém apenas um único arquivo agrupado `cli.js` (~12MB). O diretório `src/` neste repositório contém o **código-fonte TypeScript original** extraído do tarball npm.

**Idioma**: [English](README.md) | [中文](README_CN.md) | [한국어](README_KR.md) | [日本語](README_JA.md) | [Español](README_SP.md) | [हिंदी](README_HN.md) | **Português**

---

## Índice

- [Relatórios de Análise Profunda (`docs/`)](#relatórios-de-análise-profunda-docs) — Telemetria, codinomes, modo disfarçado, controle remoto
- [Aviso de Módulos Ausentes](#aviso-de-módulos-ausentes) — 108 módulos controlados por recursos que não estão no pacote central
- [Visão Geral da Arquitetura](#visão-geral-da-arquitetura) — Entrada → Mecanismo de Consulta → Ferramentas/Serviços

---

## Relatórios de Análise Profunda (`docs/`)

Relatórios derivados da versão 2.1.88 descompilada.

| # | Tópico | Principais Descobertas |
|---|-------|-------------|
| 01 | **Telemetria e Privacidade** | Dois coletores de dados (1P → Anthropic, Datadog). Impressão digital do ambiente, métricas, hash do repositório em cada evento. **Sem opção de cancelamento exposta na UI** para logs. |
| 02 | **Recursos Ocultos e Codinomes** | Codinomes de animais (Capybara v8, Tengu, **Numbat** a seguir). Os sinalizadores de recursos ("feature flags") usam pares de palavras aleatórias. Comandos ocultos: `/btw`, `/stickers`. |
| 03 | **Modo Disfarçado** | Os funcionários da Anthropic entram automaticamente no modo disfarçado em repositórios públicos públicos. O modelo é instruído a não revelar sua autoria de IA. |
| 04 | **Controle Remoto** | Verificação de hora em hora de configurações. Chaves de bloqueio remoto (killswitches) ativas para recursos críticos. |
| 05 | **Roteiro Futuro** | Codinome **Numbat** confirmado. Opus 4.7 em desenvolvimento. **KAIROS** = modo de agente totalmente autônomo com assinaturas de Pull Requests. Modo de voz detectado, mas inativo. |

---

## Aviso de Módulos Ausentes (108 módulos)

> **Este código fonte está incompleto.** 108 módulos referenciados por `feature()` **não estão incluídos** no pacote npm. Eles existem apenas no repositório interno da Anthropic.

### Por que eles estão ausentes?

O construtor `feature()` do Bun funciona em tempo de compilação:
- Retorna `true` no desenvolvimento interno da Anthropic → o código é mantido no arquivo `cli.js`.
- Retorna `false` na compilação oficial → O código morre e é eliminado (DCE - eliminação de código morto).
Assim, recursos como "DAEMON" e ferramentas experimentais não estão no pacote.

---

## Painel de Estatísticas

| Item | Contagem |
|------|-------|
| Arquivos de origem (.ts/.tsx) | ~1.884 |
| Linhas de código | ~512.664 |
| Maior arquivo | `query.ts` (~785KB) |
| Ferramentas Embutidas | ~40+ |
| Comandos de slash (`/`) | ~80+ |
| Dependências (node_modules) | ~192 pacotes |
| Runtime de Compilação | Bun |

---

## O Ciclo do Agente (Padrão)

```text
                    O LOOP PRINCIPAL
                    ================

    Usuário --> messages[] --> API Claude --> resposta
                                           |
                                 stop_reason == "tool_use"?
                                /                          \
                              sim                          não
                               |                            |
                     executar as ferramentas           retornar texto
                     acrescentar tool_result
                     retornar e repetir -------------> messages[]
```

Este é o padrão mínimo de agente autônomo. O Claude Code envolve este loop numa estrutura robusta de produção: permissões de arquivos, streaming de tokens concorrente, descompactação e armazenamento, além da conexão com os Servidores MCP.

---

## Direitos Autorais e Aviso

```text
Copyright (c) Anthropic. Todos os direitos reservados.

Todo o código-fonte deste repositório é propriedade intelectual da Anthropic e da Claude.
As publicações nestes canais são providenciadas apenas por razões de pesquisa e avaliação de desempenho. O uso comercial é estritamente proibido e sujeito a advertências legais.
```

*(Nota: O design de diagramas completo e o inventário de ferramentas constam majoritariamente apenas no arquivo principal README.md)*
