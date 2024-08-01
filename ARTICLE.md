# Building an ecommerce store using Nuxt and Medusa

![cover]()

Medusa is an open source tool that can help you set up a headless ecommerce server backend with relative ease. Couple that with Nuxt, a frontend framework for building web apps. What do you get? A full stack, modular ecommerce app that can support a wide range of use cases.

## Introduction

### What is this tutorial for?

This tutorial will show you how to set up a simple ecommerce web app using Medusa as your store backend and Nuxt for the storefront. It will showcase the fundamental building blocks required to run the app in development and production stages as well as show you how to deploy the app. At the end of this tutorial you should have acquired the overarching knowledge necessary in building ecommerce apps of a composable nature.

### Why Medusa?

Medusa is one of the few open source ecommerce backend that is feature rich allowing developers to make all sorts of ecommerce apps to fit any use case. Medusa is developer-friendly and supports all sorts of ecommerce app architectures.

### Why Nuxt?

Nuxt is meta-framework based on the popular JavaScript framework, Vue.js. Some developers prefer alternatives to the React/Next.js ecosystem. Nuxt can come in handy in that regard.

## Prerequisites

To follow along, you need to know:
- Basic understanding of HTML, CSS, and JavaScript
- Basic understanding of Node.js
- Basic understanding of the command line (Bash)
- Knowledge of Vue.js and Nuxt is a bonus but not a requrirement.
- Knowledge of Medusa is a bonus but not a requirement.

In addition you must have the following tools installed on your computer:
- [Node.js (v18 and above)](https://nodejs.org/en/download/package-manager) 
- [yarn (optional)](https://yarnpkg.com/getting-started/install)
- [git](https://git-scm.com/downloads)

Before proceeding with the tutorial you can check out the following links for useful resources:
- [Video demo](https://www.youtube.com/watch?v=ghMgYLWTUlk).
- [Live link of the app](https://nuxt-medusa-storefront.pages.dev/).
- [Git repo containing the project source code](https://github.com/Marktawa/nuxt-medusa).

## Installation and setup of the Medusa server API

In this step you will install and set up the Medusa Server backend. 

Open up your terminal and create a project folder to contain all the source code for the entire project. Name it `nuxt-medusa`.

```bash
mkdir nuxt-medusa
```

### Set up PostgreSQL on Neon

Medusa uses PostgreSQL as a database for the backend server.

Visit the [Neon - Sign Up](https://console.neon.tech/signup) page and create a new account.

[Create a new project](https://console.neon.tech/app/projects) in the Neon console. Give your project a name like `mystore` and your database a name like `storedb` then click **Create project**.

Take note of your connection string which will look something like: `postgresql://dominggobana:JyyuEdr809p@df-hidden-bonus-ertd7sio.us-east-3.aws.neon.tech/storedb?sslmode=require`. It is in the form `postgres://[user]:[password]@[host]/[dbname]`.You will provide connection string as a database URL to your Medusa server.

### Install Medusa CLI

In your terminal, inside the `my-store` folder run the following command to install the Medusa CLI. We will use it to install the Medusa server.

```bash
npm install @medusajs/medusa-cli -g
```

### Create a new Medusa project

```bash
medusa new my-medusa-store
```

You will be asked to specify your PostgreSQL database credentials. Choose "Skip database setup".

A new directory named `my-medusa-store` will be created to store the server files.

### Configure Database

Add the connection string as the `DATABASE_URL` to your environment variables. Inside `my-medusa-store` create a `.env` file and add the following:

```
DATABASE_URL=postgresql://dominggobana:JyyuEdr809p@df-hidden-bonus-ertd7sio.us-east-3.aws.neon.tech/mystoredb?sslmode=require
```

### Seed Database

Run migrations and seed data to the database by running the following command:

```bash
cd my-medusa-store
medusa seed --seed-file="./data/seed.json"
```

### Start your Medusa Backend Server

```bash
medusa develop
```

The Medusa server will start running on port `9000`.

Test your server:
```bash
curl localhost:9000/store/products
```

If it is working, you should see a list of products.

## Install and Serve Medusa Admin

This section explains how to install the Medusa Admin. The Medusa Admin provides a dashboard UI you can use to interface with the Medusa Backend server.

### Install the package

Inside `my-medusa-store` stop your Medusa server, `CTRL + C` and run this to install the Medusa Admin Dashboard.

```bash
npm install @medusajs/admin
```

Test your install by re-running your server.
```bash
medusa develop
```

Open up your browser and visit `localhost:7001` to see the Medusa Admin Dashboard. Use the email `admin@medusa-test.com` and password `supersecret` to log in.

![Medusa Admin Dashboard Login Page](https://res.cloudinary.com/craigsims808/image/upload/v1722176576/articles/nuxt-medusa/medusa-admin-login_izqmh6.png)

![Medusa Admin Dashboard Products Page](https://res.cloudinary.com/craigsims808/image/upload/v1722176577/articles/nuxt-medusa/medusa-admin-dashboard_d6spwu.png)

## Set Up Nuxt

### Create a Nuxt project

Open up a new terminal session inside the `nuxt-medusa` directory. Run the following commands:

```bash
npx nuxi@latest init storefront
```

Answer the prompts as follows:

- Need to install the following packages: nuxi@3.12.0 Ok to proceed?  
`y`  
- Which package manager would you like to use?  
`npm`
- Nuxt collects completely anonymous data about usage. This will help us improve Nuxt developer experience over time. Read more on https://github.com/nuxt/telemetry. Are you interested in participating?  
`No`
- Initialize git repository?
`Yes`

## Link Storefront to Server

### Configure Nuxt Storefront URL

To link the Medusa server with the Nuxt storefront, first, open up your Medusa project, `my-medusa-store` in your code editor, then open the `.env` file where all your environment variables are set up.

Add the variable `STORE_CORS` with the value of the URL where your storefront will be running. Nuxt by default runs on port `3000`.

```
STORE_CORS=http://localhost:3000
```

After this, your Medusa server will be ready to receive a request from your storefront and send back responses if everything works as expected.

### List Products on Nuxt Storefront

Pages represent views for each specific route pattern. Every file in the `pages/` directory represents a different route displaying its content.

Open up the `storefront` directory in your code editor.

To use pages, add `<NuxtPage />` component to `app.vue` (or remove `app.vue` for default entry).

```vue
<!-- app.vue -->
<template>
  <div>
    <NuxtPage />
  </div>
</template>
```

You can now create more pages and their corresponding routes by adding new files in the `pages/` directory.

Create a new file named `pages/index.vue` and add the following code:

```vue
<!-- pages/index.vue -->
<template>
  <div>
    <h1>Welcome to the Medusa Nuxt Store</h1>
    <h2>Products</h2>
    <ul>
      <li v-for="product in products" :key="product.id">
        {{ product.title }} - ${{ (product.variants[0].prices[0].amount / 100).toFixed(2)}}
      </li>
    </ul>
  </div>
</template>

<script setup>
const { data } = await useFetch('http://localhost:9000/store/products')
const products = data.value.products;
</script>
```

Open up the `storefront` directory in your terminal and start the Nuxt development server.

```bash
npm run dev
```

Visit [`localhost:3000`](http://localhost:3000) in your browser to see the list of products from your Medusa backend.

![Storefront with list of products](https://res.cloudinary.com/craigsims808/image/upload/v1722185984/articles/nuxt-medusa/nuxt-medusa-storefront-home_elo3rj.png)


### Add Medusa Server as environment variable

Create a `.env` file in the root of your `storefront` directory. This will store the URL to your Medusa backend server as an environment variable.

```
MEDUSA_URL=http://localhost:9000
```

Update the `nuxt.config.ts` file in the root of your `storefront` directory as follows:

```ts
// https://nuxt.com/docs/api/configuration/nuxt-config
export default defineNuxtConfig({
  compatibilityDate: '2024-04-03',
  devtools: { enabled: true },
  runtimeConfig: {
    public: {
      MEDUSA_URL: 'http://localhost:9000'
    }
  }
})
```

Lastly, update the `pages/index.vue` file so that it uses an environment variable for the URL instead of a hardcoded URL:

```vue
<!-- pages/index.vue -->
<template>
  <div>
    <h1>Welcome to the Medusa Nuxt Store</h1>
    <h2>Products</h2>
    <ul>
      <li v-for="product in products" :key="product.id">
        {{ product.title }} - ${{ (product.variants[0].prices[0].amount / 100).toFixed(2) }}
      </li>
    </ul>
  </div>
</template>

<script setup>
const runtimeConfig = useRuntimeConfig()
const MEDUSA_URL = runtimeConfig.public.MEDUSA_URL
const { data } = await useFetch(`${MEDUSA_URL}/store/products`)
const products = data.value.products;
</script>
```

Test that everything is working by visiting your storefront home page.

## Add Cart

Update `pages/index.vue` with the following code, to create a cart on page load:

```vue
<!-- pages/index.vue -->
<template>
  <div>
    <h1>Welcome to the Medusa Nuxt Store</h1>
    <h2>Products</h2>
    <ul>
      <li v-for="product in products" :key="product.id">
        {{ product.title }} - ${{ (product.variants[0].prices[0].amount / 100).toFixed(2) }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import  { onMounted } from 'vue'

const runtimeConfig = useRuntimeConfig()
const MEDUSA_URL = runtimeConfig.public.MEDUSA_URL
const { data } = await useFetch(`${MEDUSA_URL}/store/products`)
const products = data.value.products;

onMounted(async () => {
    await fetch(`${MEDUSA_URL}/store/carts`, {
        method: 'POST',
        credentials: "include",
    })
    .then((response) => response.json())
    .then(({ cart }) => {
        localStorage.setItem("cart_id", cart.id)
        console.log("The cart ID is " + localStorage.getItem("cart_id"));
    });
})

</script>
```

This code uses the `onMounted()` lifecycle hook from the Vue.js library to create the cart once the storefront has been hydrated with products.

Revisit `localhost:3000` in  your browser and open the `Console` section of your Dev Tools. You should see a message with a cart ID to confirm creation of the cart.

![Confirmation of Cart Creation](https://res.cloudinary.com/craigsims808/image/upload/v1722273459/articles/nuxt-medusa/confirmation-of-cart-creation_c0g2ts.png)

### Add "Add to Cart" Button

Update each product in `pages/index.vue` with a button to add a product to the cart.

```vue
<!-- pages/index.vue -->
<template>
  <div>
    <h1>Welcome to the Medusa Nuxt Store</h1>
    <h2>Products</h2>
    <ul>
      <li v-for="product in products" :key="product.id">
        {{ product.title }} - ${{ (product.variants[0].prices[0].amount / 100).toFixed(2) }}
        <button @click="addProductToCart(product.variants[0].id)">
                Add To Cart
        </button>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { onMounted } from 'vue'

const runtimeConfig = useRuntimeConfig()
const MEDUSA_URL = runtimeConfig.public.MEDUSA_URL
const { data } = await useFetch(`${MEDUSA_URL}/store/products`)
const products = data.value.products;

async function addProductToCart(variant_id) {
    const id = localStorage.getItem("cart_id");
    await fetch(`${MEDUSA_URL}/store/carts/${id}/line-items`, {
        method: "POST",
        dentials: "include",
        headers: {
            "Content-Type": "application/json",
        },
        body: JSON.stringify({
            variant_id,
            quantity: 1,
        }),
    })
        .then((response) => response.json())
        .then(alert('Added to Cart'))
        //.then(({ cart }) => setCart(cart));
  }

onMounted(async () => {
    await fetch(`${MEDUSA_URL}/store/carts`, {
        method: 'POST',
        credentials: "include",
    })
    .then((response) => response.json())
    .then(({ cart }) => {
        localStorage.setItem("cart_id", cart.id)
        console.log("The cart ID is " + localStorage.getItem("cart_id"));
    });
})  
  
</script>
```

Confirm whether the products are being added to the cart by performing a curl request as follows:

```bash
curl localhost:9000/store/carts/cart_01HRBTB0X79NAGJYY8T6D5BGK6
```

Replace with the specific `cart id` as listed in your browser console. If all is working your cart should be populated with some products.

### Create Cart Page

The next step is to display the cart by creating a page.

Create a new file named `pages/cart.vue`.

Add the following code to `pages/cart.vue`:

```vue
<!-- pages/cart.vue -->
<template>
  <div>
    <h1>Welcome to the Medusa Nuxt Store</h1>
    <h2>Cart</h2>
    <ul>
      <li v-for="item in items" :key="item.id">
        {{ item.title }} - ${{ (item.unit_price / 100).toFixed(2) }} x {{ item.quantity }} = ${{ (item.total / 100).toFixed(2) }}
      </li>
    </ul>
    <p>The total price for your cart is {{ total }}</p>
  </div>
</template>

<script>
import { onMounted } from 'vue'

let data
let total
let items = []

onMounted(async () => {
    const id = localStorage.getItem("cart_id");
    const res = await fetch(`${MEDUSA_URL}/store/carts/${id}`,
    {
        credentials: "include",
    });
    data = await res.json();
    items = data.cart.items;
    total = data.cart.total;
});
</script>
```

Test your cart page by adding some products to your cart then visiting `localhost:3000/cart` in your browser. You should see a list of products with the quantity and price info as well as the cart total.

![Cart Page](/cart-page.png)

### Associate cart with an email address

Next. associate your cart with an email address for the user. This is necessary to complete the cart.

```vue
<!-- pages/cart.vue -->
<template>
  <div>
    <h1>Welcome to the Medusa Nuxt Store</h1>
    <h2>Cart</h2>
    <ul>
      <li v-for="item in items" :key="item.id">
        {{ item.title }} - ${{ (item.unit_price / 100).toFixed(2) }} x {{ item.quantity }} = ${{ (item.total / 100).toFixed(2) }}
      </li>
    </ul>
    <p>The total price for your cart is {{ total }}</p>
    <p>Enter your email to Proceed to Checkout</p>
    <input id="email" type="email" v-model="email">
    <button type="submit" @click="addCustomer">
      Submit
    </button>
  </div>
</template>

<script>
import { onMounted } from 'vue'

let data
let email
let total
let items = []

onMounted(async () => {
    const id = localStorage.getItem("cart_id");
    const res = await fetch(`${MEDUSA_URL}/store/carts/${id}`,
    {
        credentials: "include",
    });
    data = await res.json();
    items = data.cart.items;
    total = data.cart.total;
});

async function addCustomer() {
    const id = localStorage.getItem("cart_id");
    await fetch(`${MEDUSA_URL}/store/carts/${id}`, {
        method: "POST",
        credentials: "include",
        headers: {
            "Content-Type": "application/json",
        },
        body: JSON.stringify({
            email: email,
        }),
    })
    .then((response) => response.json())
    .then(({ cart }) => {
        console.log("Customer ID is " + cart.customer_id)
        console.log("Customer email is " + cart.email)
        alert('Added to Cart')
    })
}
</script>
```

## Add Checkout Functionality

Create a page called, `pages/checkout.vue` to load the checkout page.

Add the following code to initialize the payment session:

```vue
<template>
    <div>
        <h1>Welcome to the Medusa Nuxt Store</h1>
        <h2>Checkout</h2>
    </div>
</template>

<script>
    import { onMounted } from "vue";

    onMounted(async () => {
        const id = localStorage.getItem("cart_id");
        fetch(`${MEDUSA_URL}/store/carts/${id}/payment-sessions`, {
            method: "POST",
            credentials: "include",
        })
        .then((response) => response.json())
        .then(({ cart }) => {
            console.log(cart.payment_sessions)
        })
    });
</script>
```

## Add Cart Completion

Next add Cart completion to the checkout process, by adding the following code:

```vue
<template>
    <div>
        <h1>Welcome to the Medusa Nuxt Store</h1>
        <h2>Checkout</h2>

        <button @click="completeCart">
            Complete Cart
        </button>
    </div>
</template>

<script>
    import { onMounted } from "vue";

    onMounted(async () => {
        const id = localStorage.getItem("cart_id");
        fetch(`${MEDUSA_URL}/store/carts/${id}/payment-sessions`, {
            method: "POST",
            credentials: "include",
        })
        .then((response) => response.json())
        .then(({ cart }) => {
            console.log(cart.payment_sessions)
        })
    });

    async function completeCart() {
        const id = localStorage.getItem("cart_id");
        await fetch(`${MEDUSA_URL}/store/carts/${id}/complete`, {
            method: "POST",
            credentials: "include",
            headers: {
                "Content-Type": "application/json",
            },
        })
        .then((response) => response.json())
        .then(({ type, data }) => {
            console.log(type, data)
            alert('Cart Complete');
        })
    }
</script>
```

Click on the `Complete Cart` button to test if the order was completed using the manual Medusa payment provider. You should see the following:

![Order Complete](/order-complete.png)

## Add Payment Provider

Stripe is a battle-tested and unified platform for transaction handling. Stripe supplies you with the technical components needed to handle transactions safely and all the analytical features necessary to gain insight into your sales. These features are also available in a safe test environment which allows for a concern-free development process.

Using the `medusa-payment-stripe plugin`, you will set up your Medusa project with Stripe as a payment processor.

[Create a Stripe account](https://dashboard.stripe.com/register) and retrieve the [Stripe Secret API Key](https://dashboard.stripe.com/test/apikeys) from your account to connect Medusa to your Stripe account. 

![Stripe Secret key](/stripe-secret-key.png)

Add the key to your environment variables in `.env` in `my-medusa-store`.

```
STRIPE_API_KEY=sk_...
```

### Install Stripe 

Open up your terminal, in the root of your Medusa backend. Stop your Medusa server and run the following command to install the Stripe plugin:

```bash
npm install medusa-payment-stripe
```

### Configure the Stripe Plugin

Next, you need to add configurations for your Stripe plugin.

In `medusa-config.js`, add the following at the end of the plugins array:

```js
const plugins = [
    // ...
    {
        resolve: `medusa-payment-stripe`,
        options: {
            api_key: process.env.STRIPE_API_KEY,
        },
    },
]
```

### Add Stripe to Region in Medusa Admin

Make sure your Medusa backend server is running, then log in to your Medusa Admin Dashboard.

![Medusa Admin Dashboard - Home Page](/medusa-admin-dashboard.png)

Go to **Settings** then select **Regions**.

![Medusa Settings - Select Regions](/medusa-admin-select-region-settings.png)

Select a region to edit.

![Medusa Regions Page](/medusa-admin-region-page.png)

Click on the three dots icon at the top right of the first section on the right. Click on **Edit Region Details** from the dropdown.

Under the providers section, add all the `Stripe` options to the region. Unselect the payment providers you want to remove from the region. Click the **Save and close** button.

![Add Stripe to Region](/edit-region-details-stripe-add-stripe.png)

### Add Stripe Key to Nuxt Storefront

Add your Stripe Publishable API Key to Nuxt storefront environment variables. Open up `.env` in Nuxt storefront and add the following:

```
PUBLIC_STRIPE_KEY=<YOUR_PUBLISHABLE_KEY>
```

### Install Dependencies

Install the necessary dependencies to show the UI and handle the payment confirmation:

```bash
npm install --save stripe @stripe/stripe-js
```

You will also use Medusa's JS Client to easily call Medusa's REST APIs:

```bash
npm install @medusajs/medusa-js
```

### Add Stripe

Update the checkout page for the Stripe payment option. Open up `pages/checkout.vue` in your code editor.

```vue


