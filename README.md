# <img align="center" src="./docs/img/polyright-icon.png" height="64">  polyright SDK

The Polyright SDK simplifies the integration of polyright functionalities into a third-party system. All necessary methods to authenticate user, read cards and do polyright or TWINT transactions through the polyright platform are provided.
The security is managed on the Polyright platform.


## Getting Started

### Device Authentication

The SDK allows activation token management. That is: 
- Get a token to activate any device through the Polyright platform
- Send device information to the Polyright platform
- Automatically retry at regular interval until activated
- Store and automatically renew the authentication token


### Card Reader management

-	Initialization of all plugged card readers
  - Automatically get the readers configuration from affiliation polyright system
  - Multiple reader types (Legic2000, Legi4000, NFC)
-	Read the card currently tapped on the reader
-	Raised events (Any reader ready / no reader ready)


### TWINT Beacon management

-	Seamless TWINT beacons initialization
-	Automatic firmware update
-	Ready to pay with TWINT
  - TWINT payment itself is managed by the polyright platform


### Customer information
-	Wait for a card and get owner information (all person properties and custom informations)
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

### Prerequisites
- RFID or NFC reader (for polyright payments)
- TWINT
- Internet connection
- Compatible platforms:
  - .NET 4.5
  - Universal Windows Platform (UWP)
  - Android (with Xamarin)
  - IOS 12 (with Xamarin)


### Installation
1. Install application from App Store
    * [Windows Store](https://www.microsoft.com/store/p/polyright-payment-terminal/9nblggh5263v)
    * [Google Play](https://play.google.com/store/apps/details?id=com.polyright.PaymentTerminal)
    * Apple Store *(coming soon)*
2. Launch application
3. Activate Payment Terminal
4. Ready!

<br>
