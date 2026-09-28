# API Inteligente de Orçamento com Spring AI e Spring Boot

Este repositório apresenta a resolução do desafio final de **Spring AI**, integrando modelos de linguagem de inteligência artificial a uma aplicação Java para processar comandos (via texto/áudio) e gerenciar transações financeiras de forma automatizada por meio de *Tool Calling*.

---

## 🚀 O Que o Projeto Faz

A aplicação atua como um assistente financeiro inteligente. O fluxo principal consiste em:
1. **Recebimento do Comando**: O usuário envia uma requisição contendo um comando de transação (ex: "Adicionar uma despesa de R$ 50,00 com alimentação").
2. **Processamento com IA**: O `ChatClient` do Spring AI interpreta o texto e identifica a intenção do usuário.
3. **Tool Calling (Execução de Função)**: A IA aciona automaticamente os métodos Java reais da aplicação para criar, atualizar ou consultar registros financeiros no banco de dados.
4. **Resposta Consolidada**: O sistema retorna uma resposta clara e natural para o usuário com o resultado da operação.

---

## 🛠️ Tecnologias Utilizadas
* **Java 17+**
* **Spring Boot**
* **Spring AI** (Integração com modelos de linguagem e ferramentas)
* **Spring Data JPA** (Persistência de dados)
* **Maven** (Gerenciamento de dependências)

---

## ✨ Melhoria Implementada
* **Validação Pré-transação**: Adicionada uma camada de validação inteligente antes de persistir a transação, garantindo que valores negativos ou descrições vazias sejam rejeitadas com mensagens amigáveis geradas pelo assistente.

---

## ⚙️ Como Executar a Aplicação

1. Clone o repositório:
   ```bash
   git clone [https://github.com/SEU_USUARIO/NOME_DO_REPOSITORIO.git](https://github.com/SEU_USUARIO/NOME_DO_REPOSITORIO.git)
   cd NOME_DO_REPOSITORIO
   spring.ai.openai.api-key=SUA_CHAVE_DE_API_AQUI
   ### 2. Implementação das Classes Principais (Exemplo Prático)

Para estruturar o código do seu projeto no padrão exigido, utilize as classes abaixo como referência:

#### **Entidade de Transação (`Transaction.java`)**
```java
package com.example.orcamento.model;

import jakarta.persistence.*;
import java.time.LocalDate;

@Entity
@Table(name = "transactions")
public class Transaction {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String description;
    private Double amount;
    private String type; // RECEITA ou DESPESA
    private LocalDate date;

    // Getters e Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    public Double getAmount() { return amount; }
    public void setAmount(Double amount) { this.amount = amount; }
    public String getType() { return type; }
    public void setType(String type) { this.type = type; }
    public LocalDate getDate() { return date; }
    public void setDate(LocalDate date) { this.date = date; }
}
package com.example.orcamento.repository;

import com.example.orcamento.model.Transaction;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface TransactionRepository extends JpaRepository<Transaction, Long> {
}
package com.example.orcamento.ai;

import com.example.orcamento.model.Transaction;
import com.example.orcamento.repository.TransactionRepository;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Description;
import org.springframework.stereotype.Component;

import java.time.LocalDate;
import java.util.function.Function;

@Component
public class FinancialTools {

    private final TransactionRepository repository;

    public FinancialTools(TransactionRepository repository) {
        this.repository = repository;
    }

    @Bean
    @Description("Adiciona uma nova transação financeira no sistema (receita ou despesa)")
    public Function<AddTransactionRequest, String> addTransaction() {
        return request -> {
            if (request.amount() <= 0) {
                return "Erro: O valor da transação deve ser maior que zero.";
            }
            Transaction tx = new Transaction();
            tx.setDescription(request.description());
            tx.setAmount(request.amount());
            tx.setType(request.type());
            tx.setDate(LocalDate.now());
            repository.save(tx);
            return "Transação de " + request.amount() + " (" + request.description() + ") salva com sucesso!";
        };
    }

    public record AddTransactionRequest(String description, Double amount, String type) {}
}
package com.example.orcamento.controller;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/budget")
public class BudgetController {

    private final ChatClient chatClient;

    public BudgetController(ChatClient.Builder chatClientBuilder, com.example.orcamento.ai.FinancialTools tools) {
        this.chatClient = chatClientBuilder
                .defaultFunctions("addTransaction")
                .build();
    }

    @PostMapping("/command")
    public String processCommand(@RequestParam String message) {
        return chatClient.prompt()
                .user(message)
                .call()
                .content();
    }
}
