# Controle Financeiro com LLM e n8n

## Visão geral
Este foi um projeto pessoal onde queria começar a monitorar mais de perto quais eram meus gastos ao longo do mês, então criei um grupo no Whatsapp onde adicionei meus Pais e Namorada. O objetivo é permitir o registro rápido de gastos pessoais falando no WhatsApp: o áudio é transcrito e convertido para JSON (data, categoria, valor, descrição, pessoa e método de pagamento), salvo tudo isso em uma planilha e exibo em um dashboard para melhorar o acompanhamento.

**Fluxo (resumido):**

* WhatsApp (Webhook) → n8n (orquestração) → Conversão Áudio→ Texto → LLM (estrutura JSON) → Google Sheets (append) → Power BI (visualização)

![Fluxo do n8n](docs/fluxo_n8n.png)


---

##  Estrutura do repositório

```
whatsapp-financial-tracker/
├─ docs/                    # prints, diagramas e material visual
├─ workflows/               # export do workflow do n8n (.json)
├─ prompts/                 # prompts e exemplos de instruções para o LLM
├─ README.md                

```

---

##  Componentes principais

* **n8n** — Orquestração do fluxo (Webhook, switches, conversão de base64, chamadas HTTP para LLM, formatação e append ao Sheets).
* **Evolution API** — Ponto de entrada dos áudios (via webhook para n8n).
* **LLM (Gemini)** — Transforma texto/transcrição em JSON estruturado.
* **Google Sheets API** — Persistência simples para histórico de gastos.
* **Power BI** — Dashboard para análise de gastos agregados (categoria, por período, médias).

---

## Descrição dos nós do workflow (baseado no print)

1. **Webhook (INPUT DADOS)**

   * Recebe POST do Evolution API.
   * Dispara o fluxo no n8n.

2. **Verificações (É meu grupo?)**

   * Filtra mensagens que não são do grupo do Controle Financeiro.
   * Se não for válido → operation do nothing.

3. **Transcrições**

   * **Converte Audio - String**: move base64 string para arquivo (para transcrição se necessário).
   * **Transcrição**: nó que executa a transcrição utilizando a LLM Gemini e estrutura em JSON.
   * **Analista Financeiro (LLM)**: recebe texto e executa prompt para estruturar em JSON com campos padrão.

4. **Manipulação (Formatação JSON)**

   * Normaliza campos.

5. **Salva Dados (Adiciona a Planilha)**

   * Nó Google Sheets: append row com o JSON transformado.

6. **Confirmação (Envia Mensagem de Confirmação)**

   * Post de retorno ao usuário confirmando que a despesa foi registrada com sucesso.

---

## Exemplo de prompt (arquivo em `prompts`)

```text
Você é um analista financeiro. Recebe uma transcrição de áudio com a seguinte mensagem:

"Ontem gastei 37.50 no almoço com cliente e foi pago no crédito"

Retorne apenas um JSON com as chaves: data (YYYY-MM-DD), categoria, valor (float), descricao.
Se algum campo não for informado, deixe como null.
```

**Exemplo de saída esperada:**

```json
{
  "data": "2025-11-11",
  "categoria": "Alimentação",
  "valor": 37.5,
  "descricao": "Almoço com cliente",
  "forma_pagamento": "Crédito"
}
```
---

## Dashboard dos Gastos

Para o dashboard eu criei indicadores que eu gostaria de acompanhar e que ficasse o mais clean possível, e pra isso utilizei cores mais neutras. Os indicadores que colquei foram:
* Total de Gastos
* Média por Transação
* Maior Gasto
* Gastos por Categoria
* Evolução dos Gastos ao Longo do Tempo
* Gastos por Pessoa (Eu, Pai, Mãe e Namorada)
* Tabela com a Descrição dos Gastos

![Dashboard dos Gastos](docs/dashboard.png)

---
