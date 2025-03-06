# Adapter and Facade Design Patterns

This document contains comprehensive notes on the Adapter and Facade design patterns, two important structural patterns in software engineering.

## Table of Contents
- [Adapter Design Pattern](#adapter-design-pattern)
  - [Real-life Examples](#real-life-examples)
  - [Software Application](#software-application)
  - [Implementation Example](#adapter-implementation-example)
  - [Benefits](#adapter-benefits)
- [Facade Design Pattern](#facade-design-pattern)
  - [Concept](#facade-concept)
  - [Real-world Scenario](#facade-scenario)
  - [Implementation Example](#facade-implementation-example)
  - [Benefits](#facade-benefits)
- [Comparison](#comparison)

## Adapter Design Pattern

The Adapter pattern allows incompatible interfaces to work together. It acts as a bridge between two incompatible interfaces by wrapping an instance of one class into an adapter class that presents the interface expected by clients.

### Real-life Examples

1. **Electrical Plugs and Sockets**
   - Different countries have different socket types (Type 1, Type 2)
   - Travel adapters convert one socket type to another
   - Example: Using a Type 1 plug with a Type 2 socket requires an adapter

2. **Computer Ports and Connectors**
   - A projector with HDMI port cannot directly connect to a Mac with Type-C port
   - Solution: HDMI to Type-C adapter
   - The adapter helps two incompatible devices work together

### Software Application

In software development, the Adapter pattern is especially useful when:

- Working with 3rd party libraries and APIs
- Need to switch libraries due to:
  - Cost considerations
  - New desired functionality
  - Maintenance issues
- Need to ensure loose coupling between application code and 3rd party code

### Adapter Implementation Example

Here's a Java example based on a banking application scenario where different banks have different APIs:

```java
// The target interface that our application expects
interface BankAPIForPhonePe {
    double checkBalance();
    void depositMoney(double amount);
    void withdrawMoney(double amount);
}

// The incompatible interface (e.g., Yes Bank API)
class YesBank {
    public double getBalance(String account) {
        // Implementation details
        return 5000.0;
    }
    
    public void addMoney(String account, double amount) {
        // Implementation details
        System.out.println("Added " + amount + " to account via Yes Bank");
    }
    
    public void subMoney(String account, double amount) {
        // Implementation details
        System.out.println("Subtracted " + amount + " from account via Yes Bank");
    }
}

// Adapter for Yes Bank
class YesBankAdapter implements BankAPIForPhonePe {
    private YesBank yesBank;
    
    public YesBankAdapter() {
        this.yesBank = new YesBank();
    }
    
    @Override
    public double checkBalance() {
        return yesBank.getBalance("default-account");
    }
    
    @Override
    public void depositMoney(double amount) {
        yesBank.addMoney("default-account", amount);
    }
    
    @Override
    public void withdrawMoney(double amount) {
        yesBank.subMoney("default-account", amount);
    }
}

// Another bank with different API (ICICI Bank)
class ICICIBank {
    public double whatIsBalance(String account) {
        // Implementation details
        return 3000.0;
    }
    
    public void putMoney(String account, double amount) {
        // Implementation details
        System.out.println("Put " + amount + " in account via ICICI Bank");
    }
    
    public void removeMoney(String account, double amount) {
        // Implementation details
        System.out.println("Removed " + amount + " from account via ICICI Bank");
    }
}

// Adapter for ICICI Bank
class ICICIBankAdapter implements BankAPIForPhonePe {
    private ICICIBank iciciBank;
    
    public ICICIBankAdapter() {
        this.iciciBank = new ICICIBank();
    }
    
    @Override
    public double checkBalance() {
        return iciciBank.whatIsBalance("default-account");
    }
    
    @Override
    public void depositMoney(double amount) {
        iciciBank.putMoney("default-account", amount);
    }
    
    @Override
    public void withdrawMoney(double amount) {
        iciciBank.removeMoney("default-account", amount);
    }
}

// Client code using the PhonePe application
class PhonePeApp {
    private BankAPIForPhonePe bankAPI;
    
    public void setBankAdapter(BankAPIForPhonePe bankAdapter) {
        this.bankAPI = bankAdapter;
    }
    
    public void processUserRequest(String action, double amount) {
        switch(action) {
            case "check-balance":
                System.out.println("Your balance is: " + bankAPI.checkBalance());
                break;
            case "deposit":
                bankAPI.depositMoney(amount);
                break;
            case "withdraw":
                bankAPI.withdrawMoney(amount);
                break;
            default:
                System.out.println("Invalid action");
        }
    }
}

// Usage example
public class AdapterDemo {
    public static void main(String[] args) {
        PhonePeApp phonePe = new PhonePeApp();
        
        // Use Yes Bank
        phonePe.setBankAdapter(new YesBankAdapter());
        phonePe.processUserRequest("check-balance", 0);
        phonePe.processUserRequest("deposit", 1000);
        
        // Switch to ICICI Bank - client code remains the same!
        phonePe.setBankAdapter(new ICICIBankAdapter());
        phonePe.processUserRequest("check-balance", 0);
        phonePe.processUserRequest("withdraw", 500);
    }
}
```

### Adapter Benefits

1. **Loose Coupling**: Helps decouple application code from third-party code
2. **Easy Switching**: Makes it easy to switch between different vendors/libraries
3. **Clean Code**: Prevents cluttering client code with vendor-specific implementation details
4. **Maintenance**: Easier to maintain as third-party API changes are isolated to adapter classes

## Facade Design Pattern

The Facade pattern provides a simplified interface to a complex subsystem of classes, making the subsystem easier to use.

### Facade Concept

"Facade" refers to the "front face" or "beautiful outward appearance" of a building. In software, it presents a clean, simplified interface to a complex system.

### Facade Scenario

Consider an e-commerce order processing system where:

1. Seller must be notified
2. Inventory must be updated
3. Logistics must be informed
4. Emails should be sent
5. App notifications must be triggered
6. SMS messages must be sent

Without a facade, clients would need to:
- Understand all these subsystems
- Call multiple methods across multiple services
- Handle complex interaction between services

### Facade Implementation Example

```java
// Individual service classes
class SellerService {
    public void notifySeller(String productId, int quantity) {
        System.out.println("Seller notified about order for product: " + productId);
    }
}

class InventoryService {
    public void updateStock(String productId, int quantity) {
        System.out.println("Inventory updated for product: " + productId);
    }
}

class LogisticsService {
    public void notifyLogistics(String productId, String address) {
        System.out.println("Logistics notified for delivery to: " + address);
    }
}

class EmailService {
    public void sendEmail(String to, String subject, String body) {
        System.out.println("Email sent to: " + to);
    }
}

class NotificationService {
    public void raiseNotify(String userId, String message) {
        System.out.println("App notification sent to user: " + userId);
    }
}

class SMSService {
    public void sendSMS(String phoneNumber, String message) {
        System.out.println("SMS sent to: " + phoneNumber);
    }
}

// Facade class that simplifies the complex subsystem
class OrderProcessingFacade {
    private SellerService sellerService;
    private InventoryService inventoryService;
    private LogisticsService logisticsService;
    private EmailService emailService;
    private NotificationService notificationService;
    private SMSService smsService;
    
    public OrderProcessingFacade() {
        this.sellerService = new SellerService();
        this.inventoryService = new InventoryService();
        this.logisticsService = new LogisticsService();
        this.emailService = new EmailService();
        this.notificationService = new NotificationService();
        this.smsService = new SMSService();
    }
    
    // A simple interface to process an order
    public void processOrder(String userId, String productId, int quantity, String address, String phoneNumber) {
        // 1. Notify seller
        sellerService.notifySeller(productId, quantity);
        
        // 2. Update inventory
        inventoryService.updateStock(productId, quantity);
        
        // 3. Notify logistics
        logisticsService.notifyLogistics(productId, address);
        
        // 4. Send email to customer
        emailService.sendEmail(userId + "@example.com", "Order Confirmation", 
                "Your order for " + productId + " has been placed successfully.");
        
        // 5. Send app notification
        notificationService.raiseNotify(userId, "Order placed successfully");
        
        // 6. Send SMS
        smsService.sendSMS(phoneNumber, "Your order for product " + productId + " has been placed.");
    }
    
    // Can also provide other simplified methods for different use cases
    public void cancelOrder(String userId, String productId) {
        // Similar orchestration of multiple service calls
        System.out.println("Order canceled through facade");
    }
    
    public void checkOrderStatus(String orderId) {
        // Check status across services
        System.out.println("Order status checked through facade");
    }
}

// Client using the facade
public class FacadeDemo {
    public static void main(String[] args) {
        // Without facade, client would need to make multiple calls to different services
        // With facade, client code is simple:
        OrderProcessingFacade orderFacade = new OrderProcessingFacade();
        
        // One method call handles the entire complex process
        orderFacade.processOrder("user123", "product456", 2, "123 Main St", "555-123-4567");
        
        // Other operations are equally simplified
        orderFacade.checkOrderStatus("order789");
    }
}
```

### Facade Benefits

1. **Simplification**: Simplifies client interaction with complex subsystems
2. **Decoupling**: Reduces dependencies between client and subsystems
3. **Higher-level Interface**: Provides a higher-level interface that makes subsystems easier to use
4. **Clean Client Code**: Client code becomes cleaner and more focused on business logic
5. **Encapsulation**: Hides subsystem complexities and implementation details
6. **Reduced Learning Curve**: New developers need to understand only the facade, not all subsystems

## Comparison

| Aspect | Adapter Pattern | Facade Pattern |
|--------|----------------|---------------|
| Purpose | Makes incompatible interfaces work together | Simplifies complex subsystem interfaces |
| Number of Interfaces | Typically works with two interfaces | Works with multiple classes/interfaces |
| Focus | Interface conversion | Providing a simplified interface |
| Knowledge | Client knows it's using an adapter | Client is unaware of the subsystem complexities |
| Common Use Cases | Third-party libraries, legacy code integration | Complex domain subsystems, API design |
| Relationship Type | Wrapper around one class | Wrapper around multiple classes |