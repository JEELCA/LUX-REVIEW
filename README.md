# LUX-REVIEW
LUXARY PRODUCT REVIEW AND USER CAN UPLOAD THIER LIFE 
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const userRoutes = require('./routes/userRoutes');
const reviewRoutes = require('./routes/reviewRoutes');
const uploadRoutes = require('./routes/uploadRoutes');
const storyRoutes = require('./routes/storyRoutes');

const app = express();

// Middleware
app.use(cors({
  origin: 'http://localhost:5173',
  credentials: true
}));
app.use(express.json());
app.use('/uploads', express.static('uploads'));

// Database connection
mongoose.connect('mongodb://localhost:27017/luxreview', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

// Routes
app.use('/api/users', userRoutes);
app.use('/api/reviews', reviewRoutes);
app.use('/api/upload', uploadRoutes);
app.use('/api/stories', storyRoutes);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    unique: true
  },
  email: {
    type: String,
    required: true,
    unique: true
  },
  password: {
    type: String,
    required: true
  },
  profilePicture: String,
  bio: String,
  following: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }],
  followers: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }],
  createdAt: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model('User', userSchema);const mongoose = require('mongoose');

const reviewSchema = new mongoose.Schema({
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  productName: {
    type: String,
    required: true
  },
  brandName: {
    type: String,
    required: true
  },
  rating: {
    type: Number,
    required: true,
    min: 1,
    max: 5
  },
  review: {
    type: String,
    required: true,
    minlength: 100
  },
  tags: [String],
  images: [String],
  videos: [String],
  upvotes: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }],
  downvotes: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }],
  comments: [{
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User'
    },
    text: String,
    createdAt: {
      type: Date,
      default: Date.now
    }
  }],
  createdAt: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model('Review', reviewSchema);<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>LuxReview</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/client/index.jsx"></script>
  </body>
</html>import React, { useState } from 'react';
import { Rating } from './Rating';
import { ImageUpload } from './ImageUpload';
import { VideoUpload } from './VideoUpload';
import '../styles/Review.css';

const Review = () => {
  const [reviewData, setReviewData] = useState({
    productName: '',
    brandName: '',
    rating: 0,
    review: '',
    tags: [],
    images: [],
    videos: []
  });

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const response = await fetch('/api/reviews', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(reviewData)
      });
      if (response.ok) {
        alert('Review submitted successfully!');
      }
    } catch (error) {
      console.error('Error submitting review:', error);
    }
  };

  return (
    <div className="review-form">
      <h2>Write a Review</h2>
      <form onSubmit={handleSubmit}>
        <div className="form-group">
          <input
            type="text"
            placeholder="Product Name"
            value={reviewData.productName}
            onChange={(e) => setReviewData({...reviewData, productName: e.target.value})}
          />
        </div>
        <div className="form-group">
          <input
            type="text"
            placeholder="Brand Name"
            value={reviewData.brandName}
            onChange={(e) => setReviewData({...reviewData, brandName: e.target.value})}
          />
        </div>
        <Rating
          value={reviewData.rating}
          onChange={(rating) => setReviewData({...reviewData, rating})}
        />
        <div className="form-group">
          <textarea
            placeholder="Write your review (minimum 100 words)"
            value={reviewData.review}
            onChange={(e) => setReviewData({...reviewData, review: e.target.value})}
            minLength={100}
          />
        </div>
        <ImageUpload
          onUpload={(images) => setReviewData({...reviewData, images})}
        />
        <VideoUpload
          onUpload={(videos) => setReviewData({...reviewData, videos})}
        />
        <button type="submit" className="submit-button">
          Submit Review
        </button>
      </form>
    </div>
  );
};

export default Review;.review-form {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.form-group {
  margin-bottom: 20px;
}

input, textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

textarea {
  min-height: 200px;
  resize: vertical;
}

.submit-button {
  background-color: #000;
  color: white;
  padding: 12px 24px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

.submit-button:hover {
  background-color: #333;
}