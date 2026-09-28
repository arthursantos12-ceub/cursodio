# Relatório do Projeto Final: Padrões de Projeto (Design Patterns)

Este documento apresenta a estruturação e a implementação prática do desafio final da jornada de Padrões de Projeto, consolidando a aplicação de conceitos de arquitetura de software utilizando Java.

---

## 🚀 Visão Geral do Projeto

O objetivo principal deste trabalho é demonstrar a aplicação consciente de **Design Patterns** para resolver problemas comuns de desenvolvimento, promovendo um código desacoplado, extensível e de fácil manutenção.

### 📋 Padrões de Projeto Aplicados
* **Singleton**: Utilizado para gerenciar instâncias únicas em serviços de configuração ou conexões.
* **Strategy**: Empregado para alternar diferentes comportamentos ou regras de cálculo de forma dinâmica.
* **Facade**: Aplicado para encapsular a complexidade de múltiplos subsistemas, expondo uma interface unificada e simplificada para o cliente.

---

## 🛠️ Tecnologias Utilizadas
* **Java** (Versão 17+)
* **Spring Boot** (para produtividade e injeção de dependências)
* **Spring Data JPA** (para persistência de dados, quando aplicável)
* **Maven** (gerenciamento de dependências)

---

## 📂 Arquitetura e Estrutura de Pacotes

A organização do código segue uma divisão clara de responsabilidades:

```text
src/
└── main/
    └── java/
        └── br/
            └── dio/
                └── gof/
                    ├── controller/   # Camada de Apresentação (API REST)
                    ├── model/        # Entidades e Repositórios de Dados
                    ├── service/      # Camada de Regra de Negócio (Implementação do Strategy/Facade)
                    └── singleton/    # Implementação do padrão Singleton
