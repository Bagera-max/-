mkdir e-wallet
cd e-wallet
npm init -y
npm install express body-parser mongoose bcrypt jsonwebtoken
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');

const app = express();
app.use(bodyParser.json());

mongoose.connect('mongodb://localhost:27017/e-wallet', { useNewUrlParser: true, useUnifiedTopology: true });

const userSchema = new mongoose.Schema({ name: String, email: String, password: String });
const walletSchema = new mongoose.Schema({ userId: String, balance: Number });
const transactionSchema = new mongoose.Schema({ from: String, to: String, amount: Number, date: Date });

const User = mongoose.model('User', userSchema);
const Wallet = mongoose.model('Wallet', walletSchema);
const Transaction = mongoose.model('Transaction', transactionSchema);

app.post('/register', async (req, res) => {
  const { name, email, password } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);
  const user = new User({ name, email, password: hashedPassword });
  await user.save();
  res.send(user);
});

app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });
  if (!user || !await bcrypt.compare(password, user.password)) {
    return res.status(401).send('Invalid credentials');
  }
  const token = jwt.sign({ userId: user._id }, 'secret_key', { expiresIn: '1h' });
  res.send({ token });
});

app.get('/balance', async (req, res) => {
  const token = req.headers.authorization.split(' ')[1];
  const decoded = jwt.verify(token, 'secret_key');
  const wallet = await Wallet.findOne({ userId: decoded.userId });
  res.send(wallet);
});

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
