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

Solution Explanation 
El rediseño del sistema elimina el código espagueti centralizado en la clase BankingService mediante la distribución de responsabilidades usando herencia y polimorfismo. En lugar de que el servicio orquestador evalúe constantemente qué tipo de identidad, operación o procesador está manejando mediante condicionales if/switch, las propias clases asumen el control de su comportamiento.   Herencia (Clasificación y Especialización)
La herencia se utiliza para definir contratos comunes (clases abstractas o interfaces) y permitir que clases más específicas adopten y extiendan esos comportamientos. En el diseño original, una única entidad contenía campos innecesarios para ciertos tipos (como un campo guardian en una identidad de negocios).   
En Identidades: Se define una clase base abstracta Identity con los métodos fundamentales. Las subclases (PersonalIdentity, BusinessIdentity, MinorIdentity, ForeignResidentIdentity) heredan esta estructura y definen únicamente los atributos que les corresponden (por ejemplo, MinorIdentity incluye al guardián, BusinessIdentity incluye el Tax ID y nombre de la empresa).   
En Operaciones: Se crea una clase base BankOperation que agrupa el tipo y el monto. Subclases como Deposit, Payroll y StandardTransfer heredan esta base, especializándose en cómo calculan su impacto en los límites diarios.Polimorfismo (Comportamiento Dinámico sin Condicionales)El polimorfismo permite que el BankingService trate a todos los objetos a través de sus clases base o interfaces, sin importarle la implementación específica subyacente. La decisión de qué código ejecutar se toma dinámicamente en tiempo de ejecución.
Polimorfismo de Identidades: Cuando el servicio necesita validar si un límite ha sido superado, simplemente llama a identity.getDailyLimit(). Si el objeto en memoria es un MinorIdentity, devolverá 100 automáticamente; si es BusinessIdentity, devolverá 50,000. El servicio no necesita preguntar qué tipo de identidad es. Lo mismo ocurre con identity.validateExtraRules(), donde ForeignResidentIdentity verificará la fecha de expiración de residencia y BusinessIdentity validará el código de aprobación para operaciones mayores a 10,000.   
Polimorfismo de Operaciones: Para saber cuánto afecta una transacción al límite diario, el servicio llama a operation.getAmountAgainstLimit(). Gracias al polimorfismo, si la operación es un Deposit, el método sobreescrito devolverá 0. Si es un Payroll, devolverá el monto multiplicado por el número de empleados. El orquestador suma este valor ciegamente, confiando en que la operación se calculó a sí misma correctamente.   Polimorfismo de Procesadores (Interfaces): El servicio maneja una lista de objetos tipo Processor (la interfaz unificada). Al ejecutar processor.process(operation), el polimorfismo garantiza que el adaptador correcto ejecute su lógica. Si es el NationalBankAdapter, manejará respuestas nulas internamente; si es el PacificBankAdapter, convertirá los montos a centavos y analizará el texto de rechazo. El BankingService solo recibe un OperationResult estándar.  
