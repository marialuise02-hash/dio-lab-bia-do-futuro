# Código da Aplicação

Esta pasta contém o código do seu agente financeiro.

## Estrutura Sugerida

```
src/
├── app.py              # Aplicação principal (Streamlit/Gradio)
├── agente.py           # Lógica do agente
├── config.py           # Configurações (API keys, etc.)
└── requirements.txt    # Dependências
```

## Exemplo de requirements.txt

```
streamlit
openai
python-dotenv
```

## Como Rodar

```python

import pandas as pd
import json
import requests
import streamlit as st
import os

# CONFIGURAÇÃO
OLLAMA_URL = 'http://localhost:11434/api/generate'
MODELO = "gemma4"

# CARREGAR DADOS (Caminhos Absolutos Automáticos)
DIRETORIO_ATUAL = os.path.dirname(os.path.abspath(__file__))

perfil_path = os.path.join(DIRETORIO_ATUAL, 'data', 'perfil_investidor.json')
transacoes_path = os.path.join(DIRETORIO_ATUAL, 'data', 'transacoes.csv')
historico_path = os.path.join(DIRETORIO_ATUAL, 'data', 'historico_atendimento.csv')
produtos_path = os.path.join(DIRETORIO_ATUAL, 'data', 'produtos_financeiros.json')

perfil = json.load(open(perfil_path, encoding='utf-8'))
transacoes = pd.read_csv(transacoes_path)
historico = pd.read_csv(historico_path)
produtos = json.load(open(produtos_path, encoding='utf-8'))

# CONTEXTO
contexto = f"""
CLIENTE: {perfil['nome']}, {perfil['idade']} anos, perfil {perfil['perfil_investidor']}
OBJETIVO: {perfil['objetivo_principal']}
PATRIMÔNIO: R$ {perfil['patrimonio_total']} | RESERVA: R$ {perfil['reserva_emergencia_atual']}

TRANSAÇÕES RECENTES:
{transacoes.to_string(index=False)}

ATENDIMENTOS ANTERIORES:
{historico.to_string(index=False)}

PRODUTOS DISPONÍVEIS:
{json.dumps(produtos, indent=2, ensure_ascii=False)}
"""

# SYSTEM PROMPT
system = """Você é a Maria - uma assistente financeira coringa para quem está começando no mundo dos investimentos

OBJETIVOS:
Ensinar conceitos acerca de investimentos de forma simples e didática com base nos dados do cliente.

REGRAS:
1. Sempre baseie suas respostas nos dados fornecidos
2. Nunca invente informações financeiras
3. Se não souber algo, admita e ofereça alternativas
4. Não recomente nenhum investimento, apenas explique como cada um funciona
5. Sempre pergunte se o cliente entendeu
6. Use os dados do cliente para fazer um atendimento personalizado
7. Nunca responda perguntas que não sejam sobre finanças
8. Responda de forma sucinta e direta
"""

def perguntar(msg):
    prompt = f"""
    {system}

    CONTEXTO DO CLIENTE:
    {contexto}

    Pergunta: {msg}"""

    try:
        r = requests.post(OLLAMA_URL, json={'model': MODELO, "prompt": prompt, "stream": False})
        resposta_json = r.json()
        
        if 'response' in resposta_json:
            return resposta_json['response']
        elif 'error' in resposta_json:
            return f"❌ Erro no Ollama: {resposta_json['error']}"
        else:
            return f"❓ Resposta inesperada do Ollama: {resposta_json}"
            
    except requests.exceptions.ConnectionError:
        return "🔌 Erro: Não foi possível conectar ao Ollama. Ele está aberto no seu computador?"

# INTERFACE
st.title('Maria, Sua Educadora Financeira')

if pergunta := st.chat_input('Sua dúvida sobre finanças...'):
    st.chat_message('user').write(pergunta)
    with st.spinner('...'):
        st.chat_message('assistant').write(perguntar(pergunta))

```
# Instalar dependências
pip install streamlit pandas requerest

# Rodar a aplicação
streamlit run app.py
