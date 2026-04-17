# Registro de Bugs — Agente de IA em Python

**Projeto:** Agente de IA para Dúvidas sobre Python (Trabalho de Curso — Data Science Academy)
**Responsável:** Isaac Trindade Araújo Júnior
**Stack:** Python · Streamlit · Groq API
**Modelo:** `openai/gpt-oss-20b` (via Groq)
**Período de QA:** 2025
**Total de bugs registrados:** 8
**Status geral:** 8/8 corrigidos ✅

---

## Legenda de Severidade

- 🔴 **Alta** — impede uso do agente ou expõe risco (credencial, dados)
- 🟡 **Média** — afeta a experiência mas tem contorno
- 🟢 **Baixa** — cosmético ou de baixo impacto

---

## BUG-001 — Histórico de conversa resetando a cada mensagem

- **Severidade:** 🔴 Alta
- **Área:** Estado de sessão (Streamlit)
- **Status:** ✅ Corrigido

**Descrição**
Cada nova pergunta chegava à API sem o histórico anterior, então o agente não conseguia responder perguntas contextuais como "me dê um exemplo" — respondia como se fosse a primeira mensagem.

**Passos para reproduzir**
1. Perguntar "O que é uma list comprehension?"
2. Após a resposta, perguntar "Me dê um exemplo"

**Resultado obtido:** Agente respondia "exemplo de quê?"
**Resultado esperado:** Agente mantém contexto e dá exemplo de list comprehension

**Correção aplicada**
O histórico passou a ser armazenado em `st.session_state.messages` e montado como lista completa na chamada à API a cada turno.

---

## BUG-002 — Chave de API commitada no repositório

- **Severidade:** 🔴 Alta
- **Área:** Segurança
- **Status:** ✅ Corrigido

**Descrição**
Em uma versão inicial, a chave de API da Groq estava escrita diretamente no código como string, e o arquivo foi commitado para o repositório público.

**Passos para reproduzir**
1. Clonar o repositório em uma versão antiga
2. Abrir `app.py`

**Resultado obtido:** Chave visível em texto plano no arquivo versionado
**Resultado esperado:** Chave carregada via variável de ambiente, fora do versionamento

**Correção aplicada**
Migração para `.env` com biblioteca `python-dotenv`, `.env` adicionado ao `.gitignore`, chave antiga rotacionada na plataforma Groq, e histórico do Git limpo para remover vestígios da chave exposta.

---

## BUG-003 — App travando quando a API retorna erro

- **Severidade:** 🔴 Alta
- **Área:** Integração com API
- **Status:** ✅ Corrigido

**Descrição**
Quando a API da Groq retornava erro (rate limit, timeout ou chave inválida), o app exibia um stack trace longo e a interface ficava inutilizável até o usuário reiniciar manualmente.

**Passos para reproduzir**
1. Forçar erro na API (ex: chave inválida)
2. Enviar pergunta
3. Observar a interface

**Resultado obtido:** Stack trace Python exposto no navegador, conversa travada
**Resultado esperado:** Mensagem amigável de erro e possibilidade de continuar a conversa

**Correção aplicada**
Envolvimento da chamada à API em `try/except` capturando exceções específicas da Groq e exibindo mensagens amigáveis via `st.error()`. Histórico preservado mesmo após erro.

---

## BUG-004 — Mensagens duplicando na interface após envio

- **Severidade:** 🟡 Média
- **Área:** Streamlit — renderização
- **Status:** ✅ Corrigido

**Descrição**
Devido ao modelo de re-execução do Streamlit, a pergunta do usuário aparecia duas vezes no chat após o envio, causando visual confuso.

**Passos para reproduzir**
1. Enviar qualquer pergunta
2. Observar a área do chat

**Resultado obtido:** Pergunta duplicada no histórico visível
**Resultado esperado:** Pergunta aparecendo apenas uma vez

**Correção aplicada**
A renderização do histórico foi centralizada em um único `for` sobre `st.session_state.messages`, removendo a exibição manual que era feita em paralelo antes da chamada à API.

---

## BUG-005 — System prompt sendo ignorado em conversas longas

- **Severidade:** 🟡 Média
- **Área:** Prompt engineering
- **Status:** ✅ Corrigido

**Descrição**
Após muitas mensagens, o agente começava a responder perguntas fora do escopo de Python, indicando que o system prompt havia saído da janela de contexto.

**Passos para reproduzir**
1. Realizar 15+ turnos de conversa
2. Perguntar algo fora do escopo (ex: "qual a capital da França?")

**Resultado obtido:** Agente respondia normalmente, fora do escopo
**Resultado esperado:** Agente mantém foco em Python

**Correção aplicada**
O system prompt passou a ser injetado como primeira mensagem em toda chamada à API, independentemente do histórico acumulado. Também foi adicionado um limite razoável de turnos mantidos no contexto.

---

## BUG-006 — Campo de entrada não limpando após envio

- **Severidade:** 🟡 Média
- **Área:** Interface
- **Status:** ✅ Corrigido

**Descrição**
Após enviar uma pergunta, o texto continuava no campo de entrada, obrigando o usuário a apagar manualmente antes de digitar a próxima.

**Passos para reproduzir**
1. Digitar uma pergunta
2. Enviar
3. Observar o campo de entrada

**Resultado obtido:** Texto anterior continuava no campo
**Resultado esperado:** Campo vazio após envio

**Correção aplicada**
Uso de `st.chat_input()` (que limpa automaticamente após submit) em vez da combinação manual de `text_input + button` que mantinha o valor no estado.

---

## BUG-007 — Blocos de código saindo sem formatação

- **Severidade:** 🟢 Baixa
- **Área:** Interface — renderização Markdown
- **Status:** ✅ Corrigido

**Descrição**
Respostas com código Python apareciam como texto corrido, sem syntax highlighting ou fonte monoespaçada, dificultando a leitura.

**Passos para reproduzir**
1. Pedir "Como abrir um arquivo CSV em Python?"
2. Observar a resposta

**Resultado obtido:** Código exibido como texto comum
**Resultado esperado:** Código renderizado em bloco com formatação adequada

**Correção aplicada**
O conteúdo da resposta passou a ser renderizado com `st.markdown(..., unsafe_allow_html=False)` em vez de `st.write()`, permitindo que os blocos ```python retornados pela API fossem renderizados com formatação correta.

---

## BUG-008 — Indicador de carregamento ausente durante chamada à API

- **Severidade:** 🟢 Baixa
- **Área:** Interface — feedback visual
- **Status:** ✅ Corrigido

**Descrição**
Durante a chamada à API (1 a 5 segundos), a interface ficava parada sem nenhum indicador, passando a impressão de que o app tinha travado.

**Passos para reproduzir**
1. Enviar uma pergunta
2. Observar a interface enquanto a API é chamada

**Resultado obtido:** Interface parada sem feedback
**Resultado esperado:** Spinner ou mensagem indicando que o agente está gerando a resposta

**Correção aplicada**
Chamada à API envolvida em `with st.spinner("Gerando resposta..."):`, dando feedback visual claro ao usuário durante o tempo de espera.

---

## Resumo Estatístico

| Severidade | Quantidade | % |
|---|---|---|
| 🔴 Alta | 3 | 37% |
| 🟡 Média | 3 | 37% |
| 🟢 Baixa | 2 | 26% |
| **Total** | **8** | **100%** |

| Área | Bugs |
|---|---|
| Estado / Integração com API | 2 |
| Segurança | 1 |
| Prompt engineering | 1 |
| Interface / Renderização | 4 |

**Taxa de correção:** 100% (8/8)

## Lições Aprendidas

- O modelo de re-execução do Streamlit exige cuidado especial com estado: tudo que precisa persistir entre turnos precisa estar em `st.session_state`.
- Credenciais nunca entram no código. O uso de `.env` + `.gitignore` deve ser a primeira coisa a configurar num projeto com API externa.
- Tratamento de erro em chamadas à API não é detalhe: é o que diferencia um protótipo frágil de uma aplicação utilizável.
- Injeção do system prompt precisa ser garantida a cada chamada, não apenas na primeira.
