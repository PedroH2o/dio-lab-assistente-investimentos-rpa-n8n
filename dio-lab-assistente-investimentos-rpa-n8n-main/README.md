# Assistente de Investimentos com RPA e n8n

Automação que recebe uma lista de clientes, cruza o **perfil de investidor** e o **saldo** de cada um com um catálogo de produtos e gera um **e-mail personalizado**, curto e sem promessa de ganhos.

## Visão geral

| Etapa | O que acontece |
|---|---|
| 1. Extração | O notebook `src/extrair_clientes.ipynb` lê os clientes da página `docs/index.html` e envia ao Webhook do n8n (POST). |
| 2. Catálogo | O n8n lê `docs/data.csv` (perfil, produto, mínimo, rentabilidade) e converte em objetos. |
| 3. Recomendação | Um nó Code filtra os produtos do perfil do cliente cujo **mínimo cabe no saldo**. |
| 4. Redação | Um nó de IA escreve o e-mail curto, citando o nome do cliente e os produtos. |
| 5. Padronização | Um nó Code limpa a resposta, garante o aviso de risco, aplica assinatura fixa e valida o e-mail. |
| 6. Resposta | O workflow devolve o resultado pelo Respond to Webhook. |

## Estrutura do repositório

```
dio-lab-assistente-investimentos-rpa-n8n/
├── README.md
├── src/
│   └── extrair_clientes.ipynb   # Notebook Python que extrai os clientes e chama o Webhook
├── n8n/
│   └── workflow.json            # Workflow exportado do n8n
└── docs/
    ├── index.html               # Página de clientes
    └── data.csv                 # Catálogo de opções de investimento
```

## Dados

**Clientes** (campos usados pelo workflow):

```json
{ "nome": "Isabela Martins", "email": "isabela@email.com", "saldo": "R$ 500,00", "perfil": "Conservador" }
```

**Catálogo** (`docs/data.csv`):

```csv
perfil,produto,minimo,rentabilidade
Conservador,Poupança,100,6.2%
Conservador,CDB Liquidez Diária,1000,12.5%
Conservador,Tesouro Selic,500,13.0%
Moderado,CDB Prefixado,1000,14.0%
Moderado,Fundo Multimercado,5000,16.0%
Moderado,Tesouro IPCA+,1000,IPCA+6.0%
Arrojado,Ações ETF,500,Variável
Arrojado,Fundo de Ações,10000,Variável
Arrojado,Criptomoedas,100,Variável
```

## Como executar

1. **Importar o workflow:** no n8n, crie um workflow novo, abra o menu `⋯` → **Import from file** e selecione `n8n/workflow.json`.
2. **Configurar credenciais** (não vão no arquivo exportado):
   - Nó de IA: API key do provedor usado (OpenAI, ou Llama API da Meta via endpoint compatível com OpenAI).
   - Nó de e-mail, se for usado: SMTP ou conta de e-mail.
3. **Ajustar a URL do catálogo:** no nó HTTP Request, aponte para o `data.csv` publicado (por exemplo, a URL raw do GitHub ou do GitHub Pages).
4. **Ativar o workflow** e copiar a URL do Webhook (use a *Production URL*).
5. **Rodar o notebook:** abra `src/extrair_clientes.ipynb`, informe a URL do Webhook no ponto indicado e execute as células.

## Exemplo de saída

```
Olá, Isabela Martins!

Considerando seu perfil conservador e seu saldo atual, separamos duas opções
para você avaliar: Poupança e Tesouro Selic. Lembramos que todo investimento
envolve risco e que a rentabilidade passada não garante resultados futuros.
Se quiser conversar sobre as opções, é só responder este e-mail.

Atenciosamente,
Equipe de Investimentos
```

## Cuidados e limitações

- **Sem promessa de ganhos:** o prompt proíbe percentuais, projeções e expressões como "garantido" ou "sem risco". Um nó de padronização reforça o aviso de risco e sinaliza e-mails suspeitos com `status = revisar`.
- **Dados fictícios:** os clientes deste repositório são exemplos. Não publique dados reais de clientes.
- **Sem chaves no repositório:** credenciais e API keys ficam apenas no n8n.
- **Perfis:** o perfil do cliente precisa existir no catálogo (Conservador, Moderado, Arrojado). Maiúsculas e acentos são ignorados.
- **Revisão humana:** as mensagens de IA devem ser revisadas antes do envio em produção, de acordo com as regras de comunicação da sua área.

## Tecnologias

n8n · JavaScript (nós Code) · Python (Jupyter Notebook) · API de LLM · HTML/CSV
