# PAYTM

## Overview
Paytm is a comprehensive platform designed to facilitate secure and seamless digital payments for users and merchants. The application supports features like user authentication, bank transactions, QR code payments, and merchant-specific operations, making it a versatile tool for financial transactions.

This is not the perfect implementation as it lacks a robust QR code system and advanced authentication mechanisms, but it serves as a great project to learn on-ramping, webhooks, and related concepts. In the future, enhancements will include improved authentication, a better UI, and additional features.

This project is built as a **Turborepo**, a monorepo architecture that manages multiple interconnected packages efficiently.

---

## High-Level Design

![System Design](assets/system-design.png)

### **Auth Provider**
- Implements secure user authentication via email or phone number.
- Supports Google Login for merchants.

### **Database**
- **Postgres**: Used for structured data storage.
- **Prisma ORM**: Simplifies database interactions.

### **Backend Stack**
- **Next.js**: Handles server-side rendering and backend logic.
- **Express.js**: Auxiliary backend for handling specific tasks.

### **Frontend Stack**
- **Next.js**: Manages frontend and backend seamlessly.
- **Tailwind CSS**: Provides a responsive and customizable UI.

### **Modules**
- **Common**: Shared utilities and services.
- **UI**: Frontend components and design systems.
- **Backend**: Core server logic and APIs.

### **Cloud Deployment**
- Deployed to a scalable cloud provider for high availability.

---

## Low-Level Design (LLD)
### **Schema**
- Comprehensive database schema designed to handle transactions, user accounts, and merchant operations.

### **Route Signatures**
- RESTful APIs with structured route signatures for seamless communication.

### **Frontend Components**
- Intuitive and interactive components to enhance user experience.

---

## Features
### **User Features**
- Login via email/phone.
- On-ramp and off-ramp transactions to/from bank accounts.
- Peer-to-peer (P2P) transfers using phone number or name.
- QR code scanning for merchant payments.

### **Merchant Features**
- Google-based login.
- QR code generation for payment acceptance.
- Payment notifications and alerts.
- Automatic bank transfer of balances every 2 days.

---

## Stack
- **Frontend and Backend**: Next.js
- **Auxiliary Backend**: Express.js
- **Monorepo Management**: Turborepo
- **Database**: Postgres
- **ORM**: Prisma
- **Styling**: Tailwind CSS

---

## Critical Paths
1. **Sending Money**: Ensures smooth peer-to-peer transactions.
2. **Merchant Withdrawals**: Transfers merchant balances to their bank accounts.
3. **User Withdrawals**: Allows users to transfer balances back to their bank accounts.
4. **Bank Webhooks**: Handles incoming money transfers via bank integrations.

---

## Hot Path Workflow
### Webhooks
- **Definition**: Webhooks are HTTP callbacks that are triggered by specific events in the application. In Paytm, webhooks are used to handle real-time notifications and events from the bank for money transfer activities.
- **Usage in Paytm**:
  - Initiate a new entry in the `onRampTransactions` table when the "Send Money" button is clicked.
  - Process bank webhook events to fulfill transactions and update the database.
  - Ensure seamless integration between user actions and backend processing.

### Workflow
1. Clicking a "Send Money" button initiates an entry in the `onRampTransactions` table.
2. The transaction is fulfilled via the bank-webhook module.
3. Users can transfer money to various wallets after successful on-ramping.
4. Maintains a `P2PTransactions` table to track peer-to-peer activities.

---

## Sample Webhook Code
```javascript
import express from "express";
import db from "@repo/db/client";
const app = express();

app.use(express.json());

app.post("/hdfcWebhook", async (req, res) => {
    // TODO: Add zod validation here?
    // TODO: HDFC bank should ideally send us a secret so we know this is sent by them
    const paymentInformation = {
        token: req.body.token,
        userId: req.body.user_identifier,
        amount: req.body.amount
    };

    try {
        await db.$transaction([
            db.balance.updateMany({
                where: {
                    userId: Number(paymentInformation.userId)
                },
                data: {
                    amount: {
                        increment: Number(paymentInformation.amount)
                    }
                }
            }),
            db.onRampTransaction.updateMany({
                where: {
                    token: paymentInformation.token
                }, 
                data: {
                    status: "Success",
                }
            })
        ]);

        res.json({
            message: "Captured"
        });
    } catch (e) {
        console.error(e);
        res.status(411).json({
            message: "Error while processing webhook"
        });
    }
});

app.listen(3003);
```

---

## Sample Transaction Code
```javascript
const initiateTransaction = async (userId, amount, targetAccount) => {
  try {
    // Create a new transaction entry
    const transaction = await prisma.onRampTransactions.create({
      data: {
        userId,
        amount,
        targetAccount,
        status: 'PENDING',
      },
    });

    // Simulate webhook callback for transaction completion
    setTimeout(async () => {
      await prisma.onRampTransactions.update({
        where: { id: transaction.id },
        data: { status: 'COMPLETED' },
      });
      console.log('Transaction completed successfully');
    }, 5000);

    return transaction;
  } catch (error) {
    console.error('Transaction initiation failed:', error);
    throw new Error('Failed to initiate transaction');
  }
};

module.exports = { initiateTransaction };
```

---

## Installation and Setup

### Step 1: Clone the Repository
```bash
git clone <repository-url>
```

### Step 2: Install Dependencies
```bash
npm install
```

### Step 3: Run Postgres Locally or on Cloud
- **Local**:
```bash
docker run -e POSTGRES_PASSWORD=mysecretpassword -d -p 5432:5432 postgres
```
- **Cloud**:
  Use services like [neon.tech](https://neon.tech) to host your database.

### Step 4: Configure Environment Variables
- Copy all `.env.example` files to `.env` in their respective directories.
- Update `.env` files with the correct database URL.

### Step 5: Database Migration and Seeding
```bash
cd packages/db
npx prisma migrate dev
npx prisma db seed
```

### Step 6: Run the Application
- Navigate to the user app directory:
```bash
cd apps/user-app
npm run dev
```

### Step 7: Test the Application
- Login using the following credentials:
  - **Phone**: `1111111111`
  - **Password**: `alice` (see `seed.ts` for details)

---

## Additional Notes
### ER Diagrams
- **Optional**: Useful for visualizing database relationships.

### Feature Planning
- Features are driven by product requirements.
- Balances simplicity and efficiency to avoid technical debt.

### Scalability
- Designed to grow with user needs and business demands, supporting high availability and performance.

---

## Future Enhancements
- Improve authentication mechanisms.
- Enhance UI/UX for better usability.
- Add advanced features to streamline operations further.

