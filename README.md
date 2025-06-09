# **Documentação do Projeto: Sistema de Aluguel de Carros em Nuvem**  

## **Visão Geral**  
Este projeto consiste em uma solução em nuvem para **gerenciamento de aluguéis de carros**, utilizando serviços da **Microsoft Azure** para garantir escalabilidade, segurança e eficiência. A arquitetura foi projetada para automatizar processos como reservas, pagamentos e gestão de veículos, seguindo uma abordagem **serverless** e **microservices**.  

---

## **1. Arquitetura da Solução**  

### **Diagrama da Arquitetura**  
![Captura de tela de 2025-06-09 18-29-54](https://github.com/user-attachments/assets/296fb91e-73e6-49c1-b8e9-8599e3e6f264)


### **Componentes Principais**  
| **Componente**               | **Descrição**                                                                 |
|------------------------------|-----------------------------------------------------------------------------|
| **Front-end (Cliente)**      | Aplicação web/mobile que interage com o usuário (React, Angular, Flutter). |
| **API Gateway**              | Gerencia e roteia requisições para o BFF (Azure API Management).           |
| **BFF (Backend for Frontend)** | Camada intermediária que consolida dados de múltiplos serviços.            |
| **Azure Function: RentProcess** | Processa reservas, verifica disponibilidade e registra aluguéis.          |
| **Azure Function: Payment**  | Gerencia transações financeiras e integra com gateways de pagamento.       |
| **Azure SQL Database**       | Armazena dados de clientes, veículos e aluguéis.                          |
| **Azure Service Bus**        | Fila de mensagens para processamento assíncrono (ex: confirmação de pagamento). |
| **Azure Logic Apps**         | Automação de workflows (ex: envio de e-mail de confirmação).              |

---

## **2. Desenvolvimento da API BFF (Backend for Frontend)**  

### **Tecnologias Utilizadas**  
- **Linguagem**: C# (.NET Core)  
- **Hospedagem**: Azure App Service  
- **Autenticação**: Azure Active Directory (JWT)  

### **Endpoints Principais**  
| **Endpoint**              | **Método** | **Descrição**                                |
|---------------------------|-----------|--------------------------------------------|
| `/api/reservations`       | POST      | Cria uma nova reserva de veículo.          |
| `/api/reservations/{id}`  | GET       | Consulta o status de uma reserva.          |
| `/api/payments`           | POST      | Processa o pagamento de um aluguel.        |
| `/api/vehicles`           | GET       | Lista veículos disponíveis para aluguel.   |

### **Fluxo de Reserva**  
1. Cliente seleciona um veículo e envia uma requisição para `/api/reservations`.  
2. O BFF valida os dados e chama a **Azure Function RentProcess**.  
3. A função verifica disponibilidade no banco de dados e registra a reserva.  
4. Retorna confirmação ou erro ao cliente.  

---

## **3. Azure Function: RentProcess**  

### **Função Principal**  
- **Gatilho**: HTTP (acionado pelo BFF).  
- **Lógica**:  
  - Valida se o veículo está disponível no período solicitado.  
  - Registra a reserva no **Azure SQL Database**.  
  - Publica um evento no **Service Bus** para processamento posterior (ex: notificação).  

### **Código Exemplo (C#)**  
```csharp
[FunctionName("RentProcess")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req,
    [ServiceBus("reservations-queue")] IAsyncCollector<string> queueMessages,
    ILogger log)
{
    var requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    var reservation = JsonConvert.DeserializeObject<Reservation>(requestBody);

    // Validação e persistência no banco de dados
    if (await IsVehicleAvailable(reservation.VehicleId, reservation.StartDate, reservation.EndDate))
    {
        await SaveReservation(reservation);
        await queueMessages.AddAsync($"Reserva confirmada: {reservation.Id}");
        return new OkObjectResult("Reserva criada com sucesso!");
    }
    return new BadRequestObjectResult("Veículo indisponível no período solicitado.");
}
```

---

## **4. Azure Function: Payment**  

### **Função Principal**  
- **Gatilho**: HTTP (acionado pelo BFF após confirmação de reserva).  
- **Lógica**:  
  - Processa pagamento via **Stripe/PagSeguro**.  
  - Atualiza status no banco de dados.  
  - Dispara **Logic App** para enviar e-mail de confirmação.  

### **Código Exemplo (C#)**  
```csharp
[FunctionName("ProcessPayment")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req,
    ILogger log)
{
    var paymentData = await JsonSerializer.DeserializeAsync<PaymentRequest>(req.Body);
    var paymentResult = await PaymentGateway.Process(paymentData);

    if (paymentResult.Success)
    {
        await UpdateReservationStatus(paymentData.ReservationId, "PAID");
        return new OkObjectResult("Pagamento aprovado!");
    }
    return new BadRequestObjectResult("Falha no processamento do pagamento.");
}
```

---

## **5. Hospedagem e Infraestrutura**  

### **Serviços Azure Utilizados**  
| **Serviço**               | **Propósito**                                  |
|---------------------------|----------------------------------------------|
| **Azure Functions**       | Execução serverless das regras de negócio.   |
| **Azure SQL Database**    | Armazenamento de dados transacionais.        |
| **Azure API Management**  | Gerenciamento e versionamento da API.        |
| **Azure Service Bus**     | Comunicação assíncrona entre serviços.       |
| **Azure Logic Apps**      | Automação de notificações e workflows.       |

### **Configuração de Deployment**  
- CI/CD via **Azure DevOps** ou **GitHub Actions**.  
- Ambiente de produção isolado com **Azure Resource Groups**.  
- Monitoramento via **Azure Application Insights**.  

---

## **6. Próximos Passos**  
✅ **Melhorias Futuras**  
- Integrar **Azure Cognitive Services** para análise de feedback dos clientes.  
- Adicionar **Azure Cosmos DB** para consultas mais rápidas de disponibilidade.  
- Implementar **Azure Kubernetes Service (AKS)** para escalar microsserviços.  

---

## **Conclusão**  
Esta solução em nuvem oferece **alta disponibilidade**, **escalabilidade automática** e **segurança**, reduzindo custos operacionais com infraestrutura serverless. A arquitetura modular permite fácil expansão para novos recursos, como **rastreamento de veículos** ou **integração com IoT**.  

📌 **Repositório do Projeto**: [GitHub Link](#)  
📧 **Contato**: [equipe@aluguelcarroscloud.com](#)  

--- 

**🔹 Documentação atualizada em: 05/07/2024**
