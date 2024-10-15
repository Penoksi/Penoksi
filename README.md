- 👋 Hi, I’m @Penoksi
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
Penoksi/Penoksi is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
// Backend ve Frontend için gerekli paketleri yükleyin:
// npm install express mongoose dotenv cors axios react react-dom

// Backend (Node.js ve Express.js)
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const dotenv = require('dotenv');
const path = require('path');

dotenv.config();
const app = express();
app.use(cors());
app.use(express.json());

// MongoDB bağlantısı
mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost:27017/yildiz-et-galerisi', {
    useNewUrlParser: true,
    useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected'))
.catch((err) => console.error('MongoDB connection error:', err));

// Ürün modeli
const Product = mongoose.model('Product', new mongoose.Schema({
    name: String,
    price: Number,
    stock: Number,
}));

// Ürün listesi alma
app.get('/products', async (req, res) => {
    const products = await Product.find();
    res.json(products);
});

// Yeni ürün ekleme
app.post('/products', async (req, res) => {
    const product = new Product(req.body);
    await product.save();
    res.status(201).json(product);
});

// Frontend (React.js)
const reactCode = `
import React, { useState, useEffect } from 'react';
import ReactDOM from 'react-dom';
import axios from 'axios';

function App() {
    const [products, setProducts] = useState([]);
    const [newProduct, setNewProduct] = useState({ name: '', price: '', stock: '' });

    useEffect(() => {
        axios.get('/products')
            .then(response => setProducts(response.data))
            .catch(error => console.error('Error fetching products:', error));
    }, []);

    const handleInputChange = (e) => {
        setNewProduct({ ...newProduct, [e.target.name]: e.target.value });
    };

    const addProduct = () => {
        axios.post('/products', newProduct)
            .then(response => setProducts([...products, response.data]))
            .catch(error => console.error('Error adding product:', error));
    };

    return (
        <div className="App">
            <h1>Yıldız Et Galerisi</h1>
            <h2>Ürün Listesi</h2>
            <ul>
                {products.map(product => (
                    <li key={product._id}>{product.name} - {product.price} TL - Stok: {product.stock}</li>
                ))}
            </ul>
            <h2>Yeni Ürün Ekle</h2>
            <input type="text" name="name" placeholder="Ürün Adı" onChange={handleInputChange} />
            <input type="number" name="price" placeholder="Fiyat" onChange={handleInputChange} />
            <input type="number" name="stock" placeholder="Stok" onChange={handleInputChange} />
            <button onClick={addProduct}>Ekle</button>
        </div>
    );
}

ReactDOM.render(<App />, document.getElementById('root'));
`;

// React frontend'i sunmak için index.html dosyası
app.get('/', (req, res) => {
    res.send(`
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Yıldız Et Galerisi</title>
        <script src="https://unpkg.com/react@17/umd/react.development.js"></script>
        <script src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
        <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    </head>
    <body>
        <div id="root"></div>
        <script type="text/babel">${reactCode}</script>
    </body>
    </html>
    `);
});

// Sunucu başlatma
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
