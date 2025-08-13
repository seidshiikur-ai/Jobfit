jobfit/
├─ public/
│  ├─ css/
│  │  └─ style.css
│  ├─ js/
│  │  └─ main.js
│  └─ images/
├─ views/
│  ├─ index.ejs
│  ├─ products.ejs
│  └─ checkout.ejs
├─ server.js
├─ package.json
const express = require('express');
const app = express();
const path = require('path');
const bodyParser = require('body-parser');

// Middleware
app.use(bodyParser.urlencoded({ extended: true }));
app.use(express.static(path.join(__dirname, 'public')));
app.set('view engine', 'ejs');

// Routes
app.get('/', (req, res) => {
    res.render('index');
});

app.get('/products', (req, res) => {
    res.render('products');
});

app.get('/checkout', (req, res) => {
    res.render('checkout');
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`JobFit server running on port ${PORT}`);
});
npm init -y
npm install express ejs body-parser stripe dotenv
jobfit/
├─ public/
│  ├─ css/
│  │  └─ style.css
│  ├─ js/
│  │  └─ main.js
│  └─ images/
├─ views/
│  ├─ index.ejs
│  ├─ products.ejs
│  ├─ checkout.ejs
│  └─ success.ejs
├─ server.js
├─ .env
require('dotenv').config();
const express = require('express');
const bodyParser = require('body-parser');
const path = require('path');
const Stripe = require('stripe');
const stripe = Stripe(process.env.STRIPE_SECRET_KEY);

const app = express();

// Middleware
app.use(bodyParser.urlencoded({ extended: true }));
app.use(express.static(path.join(__dirname, 'public')));
app.set('view engine', 'ejs');

// Routes
app.get('/', (req, res) => {
    res.render('index');
});

app.get('/products', (req, res) => {
    const products = [
        { id: 1, name: 'Professional CV Template', price: 10 },
        { id: 2, name: 'Resume Writing Guide', price: 15 },
    ];
    res.render('products', { products });
});

app.post('/checkout', async (req, res) => {
    const { productId } = req.body;
    const product = { name: 'Test Product', price: 1000 }; // For demo, later map from real products

    const session = await stripe.checkout.sessions.create({
        payment_method_types: ['card'],
        line_items: [{
            price_data: {
                currency: 'usd',
                product_data: { name: product.name },
                unit_amount: product.price * 100, // cents
            },
            quantity: 1,
        }],
        mode: 'payment',
        success_url: `${req.protocol}://${req.get('host')}/success`,
        cancel_url: `${req.protocol}://${req.get('host')}/products`,
    });

    res.redirect(303, session.url);
});

app.get('/success', (req, res) => {
    res.render('success');
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`JobFit running on port ${PORT}`));
STRIPE_SECRET_KEY=sk_test_your_stripe_secret_key_here
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>JobFit - Resume & CV Services</title>
<link rel="stylesheet" href="/css/style.css">
</head>
<body>
<header>
    <h1>Welcome to JobFit</h1>
    <nav>
        <a href="/">Home</a> | <a href="/products">Products</a>
    </nav>
</header>

<section>
    <h2>Our Services</h2>
    <ul>
        <li>Professional CV and Resume Writing</li>
        <li>eBooks & PDF Guides for Career Success</li>
        <li>Job Interview Preparation</li>
    </ul>
</section>

<section>
    <h2>Submit Your CV Details</h2>
    <form action="/submit-cv" method="POST">
        <input type="text" name="name" placeholder="Your Name" required><br>
        <input type="email" name="email" placeholder="Your Email" required><br>
        <textarea name="details" placeholder="Your CV Details"></textarea><br>
        <button type="submit">Submit</button>
    </form>
</section>

</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>JobFit Products</title>
<link rel="stylesheet" href="/css/style.css">
</head>
<body>
<header>
    <h1>JobFit Products</h1>
    <nav>
        <a href="/">Home</a>
    </nav>
</header>

<section>
    <h2>Available Products</h2>
    <ul>
        <% products.forEach(product => { %>
        <li>
            <%= product.name %> - $<%= product.price %>
            <form action="/checkout" method="POST" style="display:inline;">
                <input type="hidden" name="productId" value="<%= product.id %>">
                <button type="submit">Buy Now</button>
            </form>
        </li>
        <% }) %>
    </ul>
</section>

</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Payment Success</title>
<link rel="stylesheet" href="/css/style.css">
</head>
<body>
<h1>Payment Successful! 🎉</h1>
<p>Thank you for purchasing from JobFit.</p>
<a href="/">Go Back Home</a>
</body>
</html>