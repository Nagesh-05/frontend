const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const cors = require('cors');

const app = express();
const PORT = 5000;

// Middleware
app.use(bodyParser.json());
app.use(cors());

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/sist_food_reviews', {
    useNewUrlParser: true,
    useUnifiedTopology: true
}).then(() => console.log('MongoDB connected'))
  .catch((err) => console.error('Connection error:', err));

// Schemas and Models
const UserSchema = new mongoose.Schema({
    name: String,
    email: { type: String, unique: true },
    password: String
});

const ReviewSchema = new mongoose.Schema({
    foodItem: String,
    review: String,
    rating: Number
});

const User = mongoose.model('User', UserSchema);
const Review = mongoose.model('Review', ReviewSchema);

// Routes

// User Registration
app.post('/register', async (req, res) => {
    const { name, email, password } = req.body;
    try {
        const user = new User({ name, email, password });
        await user.save();
        res.status(201).json({ message: 'User registered successfully!' });
    } catch (err) {
        if (err.code === 11000) {
            res.status(400).json({ error: 'Email already exists!' });
        } else {
            res.status(500).json({ error: 'Something went wrong.' });
        }
    }
});

// Submit Review
app.post('/submit_review', async (req, res) => {
    const { foodItem, review, rating } = req.body;
    try {
        const newReview = new Review({ foodItem, review, rating });
        await newReview.save();
        res.status(201).json({ message: 'Review submitted successfully!' });
    } catch (err) {
        res.status(500).json({ error: 'Could not submit review.' });
    }
});

// Fetch Reviews
app.get('/reviews', async (req, res) => {
    try {
        const reviews = await Review.find();
        res.status(200).json(reviews);
    } catch (err) {
        res.status(500).json({ error: 'Could not fetch reviews.' });
    }
});

// Start the Server
app.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
});
