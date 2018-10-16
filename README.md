# <img align="center" src="https://github.com/polyright/PaymentTerminal-URI-Scheme/raw/master/docs/img/polyright-icon.png" height="64">  Polyright SDK

The Polyright SDK simplifies the integration of polyright functionalities into a third-party system. All necessary methods to authenticate user, read cards and do polyright or TWINT transactions through the polyright platform are provided.
The security is managed on the Polyright platform.

## Glossary

- **Polyright system:** A Polyright Manager installation, installed on a customer server (on premise). Also known as "affiliation".
- **Polyright platform:** Cloud website connected to multiple polyright systems

## Functionalities

### Device Authentication

The SDK allows activation token management. That is: 
- Get a token to activate any device through the Polyright platform
- Send device information to the Polyright platform
- Automatically retry at regular interval until activated
- Store and automatically renew the authentication token


### Card Reader management

-	Initialization of all plugged card readers
  - Automatically get the readers configuration from affiliation polyright system
  - Multiple reader types (Legic2000, Legic4000, NFC)
-	Read the card currently tapped on the reader
-	Raised events (Any reader ready / no reader ready)


### TWINT Beacon management

-	Seamless TWINT beacons initialization
-	Automatic firmware update
-	Ready to pay with TWINT
  - TWINT payment itself is managed by the polyright platform


### Customer information
-	Wait for a card and get owner information from the polyright system (all person properties and custom informations)
-	Get the transactions of the person

### Financial
- Transaction process
  - Begin a transaction with information (amount, description, sold products, ...)
  - Wait for a customer (polyright card , TWINT customer)
  - Choose the person account to use
  - Wait until the transaction is finalized
  - Cancel the transaction
  
- Financial closing
  - Do a financial closing and get a summary of the transactions done

- Get the transactions done by the device

## Getting Started

### Compatible platforms
Compatible with .NET Standard 1.4:
- .NET Framework 4.6.1
- Universal Windows Platform (UWP)
- Android (with Xamarin)
- IOS 12 (with Xamarin)

### Prerequisites
- RFID or NFC reader (for polyright payments)
- TWINT beacon (for TWINT payment with beacon)
- Internet connection
- A Polyright system to be connected to
- Access to the Polyright NuGet feed : https://nuget.polyright.com/nuget/Internal/

  
All SDK methods are asynchronous. See [Task-based Asynchronous Programming](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/task-based-asynchronous-programming)

### Installation
1. Add the Polyright NuGet feed to the packages sources
2. Install the **Polyright.SDK** NuGet package from Polyright NuGet feed using the NuGet Package Manager

### Initialization

First of all, the SDK must be initialized :
- Send client credentials
- Send device information
- Instance of platform specific cardreaders / TWINT beacons SDKs
- Custom secure storage if necessary


```csharp
var cardReaders = new Polyright.SDK.Devices.CardReaders(new CardReaders());
var beacons = new Polyright.SDK.Devices.Beacons(new Beacons());
var clientMetadata = new ClientMetadata
{
	ApplicationName = "MyTestApplication",
	ApplicationVersion = "1.0.0.0"
};
Polyright.SDK.Windows.Container.Init(cardReaders, beacons, null, clientMetadata);
PolyrightContext.Init(options =>
{
  options.ClientId = "Polyright.PaymentSDK";
  options.SecretKey = "f952a0a18be81faaafd842e24232f96c8052ede6328601d06c9f10a0130b7f7f";
  options.EnvironmentType = EnvironmentType.Development;
});

```
### Connection and authentication

To be used, a device using the Polyright SDK must be manually activated by the Polyright support

```csharp
var connectionManager = new ConnectionManager();
connectionManager.ConnectAsync(eventArgs =>
{
   // Contact Polyright support to activate your device from your activation code
   Console.WriteLine($"Activation code: {eventArgs.ActivationCode}");
   return Task.CompletedTask;
});
await connectionManager.AwaitServiceReadyAsync(token);
```

Use StatusChanged event to be notified in case of connection status change.

```csharp
connectionManager.StatusChanged += (sender, args) =>
{
   Console.WriteLine("Service status changed.");
   Console.WriteLine($"Activation code: {args.Status.ActivationCode}");
   Console.WriteLine($"IsOnline: {args.Status.IsOnline}");
   Console.WriteLine($"IsAuthenticated: {args.Status.IsAuthenticated}");
   Console.WriteLine($"Error: {args.Error.Message}");
};
```

### Wait for devices (card reader, TWINT beacon)

Devices are automatically initialized after the initialization of the SDK.

It is possible to wait until they are ready.

```csharp
var deviceManager = new DeviceManager();
await deviceManager.AwaitCardReaderReadyAsync(token);
await deviceManager.AwaitTwintBeaconReadyAsync(token);

```

Use CardReaderStateChanged and TwintBeaconStateChanged to be notified if device status change.

```csharp
deviceManager.CardReaderStateChanged += (sender, args) => { Console.WriteLine($"Card reader state changed: {args.IsReady}"); };
deviceManager.TwintBeaconStateChanged += (sender, args) => { Console.WriteLine($"TWINT beacon state changed: {args.IsReady}"); };
```


### Do a transaction with Polyright account

First, a transaction must be initialized, with all its information.

It's possible to add actions for each event :
- Transaction in progress (someone put his card on the reader for example)
- Person account to select
- Transaction ready to be validated (ex. : add a discount before validating)
- Transaction completed

```csharp
var financialService = new FinancialService();
var transactionRequest = new TransactionRequest
{
	Amount = -1m,
	Purpose = "Transaction purpose"
}.OnSelectAccount((scope, persons) =>
	{
	  // Select person account. In this example, select use the personal account
	  var person = persons.FirstOrDefault();
	  IAccount account = person?.PersonalAccount;
	  Console.WriteLine($"{person?.FormattedName}: {account?.Balance}");
	  return Task.FromResult(account);
	});
var transactionScope = await financialService.BeginTransactionAsync(transactionRequest);
await transactionScope.AwaitTransactionCompletionAsync();
Console.WriteLine($"Transaction completed. Status: {transactionScope.Transaction.Status}");
```

### Do a transaction with TWINT

The process for a TWINT transaction is the same.

Once the transaction has been initialized, it is possible to get the TWINT code / QR-Code, or wait until someone pay with the TWINT beacon.

```csharp
var financialService = new FinancialService();
var transactionRequest = new TransactionRequest
{
	Amount = -1m,
	Purpose = "Transaction purpose"
};
var transactionScope = await financialService.BeginTransactionAsync(transactionRequest);
var twintData = await transactionScope.GetTwintDataAsync();
Console.WriteLine($"TWINT code: {twintData.Token}");
await transactionScope.AwaitTransactionCompletionAsync();
Console.WriteLine($"Transaction completed. Status: {transactionScope.Transaction.Status}");
```

### Do a financial closing

```csharp
await financialService.DoFinancialClosingAsync();
```
### Get Person Data and transactions

The SDK can be linked to multiple Polyright systems. The same card can be coded in multiple Polyright systems. 

```csharp
var customerService = new CustomerService();
var customer = await customerService.AwaitCustomerAsync();
var persons = await customerService.GetPersonsAsync(customer);
var personTransactions = await customerService.GetPersonTransactionsAsync(persons.First(), new DateTime(2018, 10, 1), new DateTime(2018, 10, 1), 0, 100);
```


