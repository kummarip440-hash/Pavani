// Movie Ticket Booking Backend - Made for Pavani
const express = require('express');
const cors = require('cors');

const app = express();
const PORT = process.env.PORT || 3000;

app.use(cors());
app.use(express.json());

// Sample Data
let movies = [
  { id: 1, title: "Kalki 2898 AD", genre: "Sci-Fi", duration: "3h 1m", price: 180 },
  { id: 2, title: "Devara", genre: "Action", duration: "2h 55m", price: 200 },
  { id: 3, title: "Pushpa 2", genre: "Action", duration: "3h 20m", price: 250 }
];

let shows = [
  { id: 1, movieId: 1, time: "10:00 AM", screen: "Screen 1", totalSeats: 60 },
  { id: 2, movieId: 1, time: "02:30 PM", screen: "Screen 2", totalSeats: 60 },
  { id: 3, movieId: 2, time: "06:00 PM", screen: "Screen 1", totalSeats: 60 }
];

let bookings = []; // {id, movieId, showId, userName, seats[], bookingTime}

// API Routes

// Home
app.get('/', (req, res) => {
  res.json({ message: "Welcome Pavani! Movie Ticket Booking API Running", owner: "Pavani" });
});

// Get all movies
app.get('/api/movies', (req, res) => {
  res.json(movies);
});

// Get shows for a movie
app.get('/api/movies/:id/shows', (req, res) => {
  const movieShows = shows.filter(s => s.movieId == req.params.id);
  res.json(movieShows);
});

// Get booked seats for a show
app.get('/api/shows/:id/seats', (req, res) => {
  const showBookings = bookings.filter(b => b.showId == req.params.id);
  const bookedSeats = showBookings.flatMap(b => b.seats);
  res.json({ showId: req.params.id, bookedSeats, availableSeats: 60 - bookedSeats.length });
});

// Book ticket
app.post('/api/book', (req, res) => {
  const { movieId, showId, userName, seats } = req.body;

  if (!movieId || !showId || !userName || !seats) {
    return res.status(400).json({ error: "movieId, showId, userName, seats required" });
  }

  // Check if seats already booked
  const existingBookings = bookings.filter(b => b.showId == showId);
  const alreadyBooked = existingBookings.flatMap(b => b.seats);
  const conflict = seats.some(s => alreadyBooked.includes(s));

  if (conflict) {
    return res.status(400).json({ error: "Some seats already booked", alreadyBooked });
  }

  const newBooking = {
    id: bookings.length + 1,
    movieId,
    showId,
    userName,
    seats,
    bookingTime: new Date().toISOString()
  };

  bookings.push(newBooking);
  res.json({ message: `Booking Successful for ${userName}!`, booking: newBooking });
});

// Get all bookings (for Pavani admin)
app.get('/api/bookings', (req, res) => {
  res.json(bookings);
});

// Get booking by user
app.get('/api/bookings/:userName', (req, res) => {
  const userBookings = bookings.filter(b => b.userName.toLowerCase() == req.params.userName.toLowerCase());
  res.json(userBookings);
});

// Cancel booking
app.delete('/api/bookings/:id', (req, res) => {
  const index = bookings.findIndex(b => b.id == req.params.id);
  if (index === -1) return res.status(404).json({ error: "Booking not found" });
  
  bookings.splice(index, 1);
  res.json({ message: "Booking cancelled successfully" });
});

app.listen(PORT, () => {
  console.log(`Server running for Pavani on port ${PORT}`);
});{
  "name": "movie-ticket-booking-pavani",
  "version": "1.0.0",
  "description": "Backend for Pavani",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5"
  }
}
