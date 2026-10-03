# Exercise-7
Mermaid Code
classDiagram
    %% --- Jerarquía de Identidades ---
    class Identity {
        <<abstract>>
        +getDailyLimit() double
        +isAllowed(OperationType type) boolean
        +validateExtraRules(BankOperation op, LocalDate date) void
    }
    class PersonalIdentity
    class BusinessIdentity
    class MinorIdentity
    class ForeignResidentIdentity
    
    Identity <|-- PersonalIdentity
    Identity <|-- BusinessIdentity
    Identity <|-- MinorIdentity
    Identity <|-- ForeignResidentIdentity

    %% --- Jerarquía de Operaciones ---
    class BankOperation {
        <<abstract>>
        +OperationType type
        +double amount
        +getAmountAgainstLimit() double
    }
    class Deposit {
        +getAmountAgainstLimit() double
    }
    class Payroll {
        -List~String~ employees
        +getAmountAgainstLimit() double
    }
    class StandardTransfer {
        +getAmountAgainstLimit() double
    }
    
    BankOperation <|-- Deposit
    BankOperation <|-- Payroll
    BankOperation <|-- StandardTransfer

    %% --- Patrón Adapter para Processors ---
    class Processor {
        <<interface>>
        +supports(OperationType type) boolean
        +calculateFee(BankOperation op) double
        +process(BankOperation op) OperationResult
    }
    
    class NationalBankAdapter {
        -NationalBankAPI api
        +process(BankOperation op) OperationResult
    }
    class PacificBankAdapter {
        -PacificBankAPI api
        +process(BankOperation op) OperationResult
    }
    class SwiftGatewayAdapter {
        -SwiftAPI api
        +process(BankOperation op) OperationResult
    }
    
    Processor <|.. NationalBankAdapter
    Processor <|.. PacificBankAdapter
    Processor <|.. SwiftGatewayAdapter

    %% --- APIs Externas (No modificables) ---
    class NationalBankAPI {
        <<External SDK>>
        +postTransaction(String ref, String acc, double amt) String
    }
    class PacificBankAPI {
        <<External SDK>>
        +submit(String acc, String dest, long cents, String desc) String
        +submitPayroll(String acc, List~String~ dests, long cents) String
    }
    class SwiftAPI {
        <<External SDK>>
        +sendWire(String sender, String bic, String acc, double amt, String curr) String
    }

    NationalBankAdapter --> NationalBankAPI : envuelve (wraps)
    PacificBankAdapter --> PacificBankAPI : envuelve (wraps)
    SwiftGatewayAdapter --> SwiftAPI : envuelve (wraps)

    %% --- Servicio Orquestador ---
    class BankingService {
        -List~Processor~ processors
        +execute(Identity id, BankOperation op) OperationResult
    }
    
    BankingService --> Identity : usa
    BankingService --> BankOperation : usa
    BankingService --> Processor : usa

Mermaid Diagram
<img width="8192" height="2250" alt="MermaidDiagram" src="https://github.com/user-attachments/assets/ed66546e-b73b-4997-8c98-b7d0bf13907e" />
