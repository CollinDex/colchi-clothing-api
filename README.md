# Stripe Payment Intent API

This project demonstrates how to create a backend API for handling Stripe payment intents using Node.js, Express, and the Stripe Node.js library. The API enables the creation of payment intents that can be used to process payments with Stripe.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [API Endpoints](#api-endpoints)
- [Frontend Integration](#frontend-integration)
- [Running the Server](#running-the-server)
- [License](#license)

## Prerequisites

Before you begin, ensure you have the following installed:

- Node.js (v14 or later)
- npm (v6 or later)

You also need a Stripe account and the corresponding secret key.

## Installation

1. Clone the repository:
    ```sh
    git clone https://github.com/your-username/stripe-payment-intent-api.git
    cd stripe-payment-intent-api
    ```

2. Install the required packages:
    ```sh
    npm install
    ```

3. Create a `.env` file in the root directory and add your Stripe secret key:
    ```env
    STRIPE_SECRET_KEY=your-stripe-secret-key
    ```

## Usage

1. Start the server:
    ```sh
    node server.js
    ```

2. The server will start on `http://localhost:3000`.

## API Endpoints

### Create Payment Intent

- **URL**: `/create-payment-intent`
- **Method**: `POST`
- **Headers**: `Content-Type: application/json`
- **Body**:
    ```json
    {
        "amount": 5000
    }
    ```
    - `amount`: The amount to be charged in cents (e.g., `5000` for $50.00).

- **Response**:
    ```json
    {
        "clientSecret": "your-client-secret"
    }
    ```
    - `clientSecret`: The client secret of the payment intent.

## Frontend Integration

To integrate this API with a frontend application, you can use the following example code in a React application with Stripe elements.

1. Install Stripe libraries:
    ```sh
    npm install @stripe/react-stripe-js @stripe/stripe-js
    ```

2. Implement the payment handler function in your React component:

    ```jsx
    import React, { useState } from 'react';
    import { CardElement, useStripe, useElements } from '@stripe/react-stripe-js';

    const PaymentForm = ({ amount, currentUser }) => {
        const stripe = useStripe();
        const elements = useElements();
        const [isProcessingPayment, setIsProcessingPayment] = useState(false);

        const paymentHandler = async (e) => {
            e.preventDefault();

            if (!stripe || !elements) {
                return;
            }

            setIsProcessingPayment(true);

            const response = await fetch('http://localhost:3000/create-payment-intent', {
                method: 'post',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ amount: amount * 100 })
            }).then((res) => res.json());

            const { clientSecret } = response;
            console.log(clientSecret);

            const paymentResult = await stripe.confirmCardPayment(clientSecret, {
                payment_method: {
                    card: elements.getElement(CardElement),
                    billing_details: {
                        name: currentUser ? currentUser.displayName : 'Guest'
                    },
                },
            });

            setIsProcessingPayment(false);

            if (paymentResult.error) {
                alert(paymentResult.error.message);
            } else {
                if (paymentResult.paymentIntent.status === 'succeeded') {
                    alert('Payment Successful');
                }
            }
        };

        return (
            <form onSubmit={paymentHandler}>
                <CardElement />
                <button type="submit" disabled={isProcessingPayment}>
                    {isProcessingPayment ? 'Processing...' : 'Pay'}
                </button>
            </form>
        );
    };

    export default PaymentForm;
    ```

## Running the Server

Start the server by running the following command in your project directory:
```sh
node server.js
```
The server will run on `http://localhost:3000`.

## License

This project is licensed under the MIT License. See the LICENSE file for details.
