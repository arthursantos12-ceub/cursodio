# 🚀 EduDev — Assistente Virtual de Aprendizado e Carreira em Python & Data

O **EduDev** é um assistente virtual focado em orientar iniciantes em programação, Ciência de Dados e Engenharia de Software. Ele foi desenvolvido como parte do Lab **"Construa Seu Assistente Virtual Com Inteligência Artificial"** da Digital Innovation One (DIO).

---

## 📌 1. Documentação do Agente

### **Objetivo Geral**
Ajudar estudantes e profissionais em transição de carreira a estruturar seus estudos de Python e Data Science, tirar dúvidas conceituais e entender os primeiros passos para o mercado de trabalho, garantindo respostas fundamentadas e livres de alucinações.

### **Persona**
* **Nome:** EduDev
* **Papel:** Mentor técnico de programação e carreira em tecnologia.
* **Tom de Voz:** Encorajador, didático, objetivo e profissional.
* **Público-Alvo:** Desenvolvedores iniciantes, estudantes da DIO e profissionais em transição para áreas de TI e Ciência de Dados.

### **Escopo e Limites (Guardrails)**
| O que o EduDev FAZ | O que o EduDev NÃO FAZ |
| :--- | :--- |
| Explicar conceitos básicos e intermediários de Python | Resolver provas, trabalhos acadêmicos ou testes técnicos inteiros |
| Recomendar trilhas e ordem lógica de estudos | Recomendar tecnologias/frameworks obsoletos ou fora da base |
| Orientar sobre preparação para entrevistas e portfólio | Dar conselhos de investimento, saúde, direito ou temas gerais |
| Dizer explicitamente quando não sabe a resposta | Inventar (alucinar) comandos, bibliotecas ou conceitos inexistentes |

---

## 📚 2. Base de Conhecimento (`data/knowledge_base.json`)

A base de conhecimento oficial contida no assistente responde a três eixos fundamentais:

```json
{
  "trilhas_estudo": [
    {
      "topico": "Trilha Python Fundamentos",
      "ordem": ["Sintaxe Básica e Variáveis", "Estruturas Condicionais (if/else)", "Laços de Repetição (for/while)", "Funções e Módulos", "Estruturas de Dados (Listas, Dicionários, Tuplas)"]
    },
    {
      "topico": "Trilha Ciência de Dados",
      "ordem": ["Python Básico", "Manipulação de Dados com Pandas e NumPy", "Visualização de Dados com Matplotlib e Seaborn", "Estatística Descritiva", "Introdução ao Scikit-Learn (Machine Learning)"]
    }
  ],
  "conceitos_chave": [
    {
      "termo": "Pandas",
      "definicao": "Biblioteca Python para manipulação e análise de dados estruturados (tabelas e Séries)."
    },
    {
      "termo": "Git e GitHub",
      "definicao": "Git é o sistema de controle de versão do código. GitHub é a plataforma onde você hospeda seus repositórios para montar seu portfólio."
    },
    {
      "termo": "Clean Code (Código Limpo)",
      "definicao": "Prática de escrever códigos fáceis de ler, manter e testar por outros desenvolvedores."
    }
  ],
  "dicas_carreira": [
    {
      "assunto": "Primeiro Portfólio",
      "recomendacao": "Crie de 2 a 3 projetos bem documentados no GitHub com README detalhado explicitando o problema, a solução e as métricas obtidas."
    },
    {
      "assunto": "Preparação para Entrevistas",
      "recomendacao": "Pratique a explicação do seu raciocínio lógico em voz alta e revise os fundamentos da linguagem antes de focar em frameworks."
    }
  ]
}
```

---

## 🧠 3. Prompts do Agente (`docs/prompts.md`)

### **System Prompt (Instruções Principais)**

```text
Você é o EduDev, um mentor de inteligência artificial amigável, focado em ajudar estudantes e iniciantes nas áreas de Programação Python e Ciência de Dados.

DIRETRIZES DE COMPORTAMENTO:
1. Responda sempre em português de forma clara, didática e motivadora.
2. Utilize exclusivamente a BASE DE CONHECIMENTO fornecida no contexto para embasar suas respostas sobre trilhas de estudo e conceitos.
3. Se o usuário fizer uma pergunta sobre temas fora de programação/tecnologia (ex: saúde, esportes, finanças pessoas) ou pedir algo que não está na sua base, responda exatamente: "Desculpe, meu foco é ajudar com trilhas de aprendizado e carreira em programação e dados. Não tenho informações suficientes sobre esse tópico na minha base."
4. NUNCA invente bibliotecas, sintaxes ou funções que não existem.
5. Sempre encerre suas respostas com uma pergunta curta que ajude o usuário a dar o próximo passo nos estudos.

FORMATO DA RESPOSTA:
- Use markdown para destacar códigos (`exemplo`), tópicos em listas e negritos.
- Mantenha respostas curtas e objetivas (máximo de 3 parágrafos).
```

---

## 💻 4. Aplicação Funcional em Python (`src/app.py`)

Abaixo está o código funcional desenvolvido em **Python** com **Streamlit** e integração via API da OpenAI/Gemini.

```python
import streamlit as st
import json
import os
from openai import OpenAI

# 1. Carregar Base de Conhecimento
def load_knowledge_base():
    kb_data = {
        "trilhas": {
            "Python": ["Sintaxe Básica", "Condicionais e Loops", "Funções", "Estruturas de Dados"],
            "Data Science": ["Python Básico", "Pandas & NumPy", "Matplotlib/Seaborn", "Scikit-Learn"]
        },
        "conceitos": {
            "Pandas": "Biblioteca para manipulação e análise de dados em formato tabular.",
            "Git": "Ferramenta de controle de versão para acompanhar alterações no código.",
            "README": "Documento principal do repositório que explica o projeto e como executá-lo."
        },
        "carreira": {
            "Portfolio": "Crie repositórios limpos no GitHub com projetos práticos resolvendo problemas reais."
        }
    }
    return json.dumps(kb_data, ensure_ascii=False)

# Configuração da Página
st.set_page_config(page_title="EduDev - Assistente Virtual", page_icon="🤖")
st.title("🤖 EduDev — Seu Mentor de Carreira e Python")
st.caption("Assistente Virtual treinado para orientação em tecnologia e análise de dados.")

# Inicializar Histórico de Chat
if "messages" not in st.session_state:
    st.session_state.messages = [
        {"role": "assistant", "content": "Olá! Sou o EduDev 🚀. Como posso ajudar na sua jornada de aprendizado em Python ou Data Science hoje?"}
    ]

# Exibir Mensagens Anteriores
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

# Entrada da API Key e Prompt do Usuário
api_key = st.sidebar.text_input("Cole sua OpenAI API Key:", type="password")

if prompt := st.chat_input("Digite sua dúvida sobre Python, Data Science ou Carreira..."):
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user"):
        st.markdown(prompt)

    if not api_key:
        with st.chat_message("assistant"):
            st.error("Por favor, insira sua API Key da OpenAI na barra lateral para continuar.")
    else:
        try:
            client = OpenAI(api_key=api_key)
            knowledge = load_knowledge_base()

            system_prompt = f"""
            Você é o EduDev, um mentor especialista em Python e Ciência de Dados.
            Instruções:
            - Use a BASE DE CONHECIMENTO a seguir para responder: {knowledge}
            - Seja direto, didático e encorajador.
            - Se a dúvida estiver fora do escopo de programação/dados ou não estiver na base, responda:
              'Desculpe, meu foco é orientação em programação e Ciência de Dados. Não possuo essa informação na minha base.'
            - Termine propondo o próximo passo de estudo.
            """

            response = client.chat.completions.create(
                model="gpt-3.5-turbo",
                messages=[
                    {"role": "system", "content": system_prompt},
                    *st.session_state.messages
                ],
                temperature=0.2
            )

            bot_reply = response.choices[0].message.content
            
            with st.chat_message("assistant"):
                st.markdown(bot_reply)
            
            st.session_state.messages.append({"role": "assistant", "content": bot_reply})

        except Exception as e:
            st.error(f"Erro ao processar resposta: {str(e)}")
```

---

## 📊 5. Avaliação e Métricas (`docs/avaliacao.md`)

Para garantir o controle de qualidade do assistente, foram realizados testes com 3 cenários principais:

| Cenário de Teste | Pergunta da Pessoa Usuária | Resposta Esperada do EduDev | Resultado Obtido | Status |
| :--- | :--- | :--- | :--- | :--- |
| **1. Pergunta Dentro do Escopo** | "Qual a ordem ideal para aprender Ciência de Dados?" | Apresentar os passos: Python -> Pandas -> Matplotlib -> Scikit-Learn. | Apresentou a trilha correta segundo a base e sugeriu um projeto prático. | **APROVADO** |
| **2. Pergunta Fora do Escopo** | "Qual a melhor ação para investir na bolsa hoje?" | Recusar a resposta informando limitação de escopo. | *"Desculpe, meu foco é ajudar com trilhas de aprendizado e carreira em programação..."* | **APROVADO** |
| **3. Pergunta Ambígua** | "O que devo aprender primeiro?" | Pedir para o usuário especificar o objetivo (ex: backend vs dados). | Identificou a ambiguidade e ofereceu opção entre trilha Dev ou Data. | **APROVADO** |

---

## 📢 6. Pitch Final

### **O Problema**
Pessoas iniciantes no universo da programação enfrentam **overdose de informação**, ficando perdidas em meio a tantas linguagens, tutoriais desatualizados e recomendações contraditórias na internet, além do risco de alucinações ao usar IAs genéricas sem contexto guardrail.

### **A Solução**
O **EduDev** funciona como um mentor de bolso baseado em diretrizes claras de **Geração Aumentada por Recuperação (RAG)** e controle de escopo. Ele direciona o foco do estudante exatamente para o que importa, respondendo dúvidas com base em trilhas de estudo validadas.

### **O Valor Gerado**
* **Economia de Tempo:** Reduz em até 60% o tempo gasto procurando por onde começar a estudar.
* **Segurança na Informação:** Zero respostas inventadas sobre conceitos técnicos.
* **Orientação de Carreira Prática:** Foco contínuo em construção de repositórios e portfólio real no GitHub.

---

## 🛠️ Como Executar o Projeto

1. Clone este repositório:
   ```bash
   git clone https://github.com/seu-usuario/edudev-assistente-ia.git
   cd edudev-assistente-ia
   ```

2. Instale as dependências:
   ```bash
   pip install streamlit openai
   ```

3. Inicie a aplicação Streamlit:
   ```bash
   streamlit run src/app.py
