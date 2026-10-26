mkdir movie-ticket-booking
cd movie-ticket-booking
npm init -y
npm install express mysql2 cors dotenv
const express = require("express");
const mysql = require("mysql2/promise");
const cors = require("cors");
require("dotenv").config();

const app = express();

app.use(cors());
app.use(express.json());

const pool = mysql.createPool({
  host: process.env.DB_HOST || "localhost",
  user: process.env.DB_USER || "root",
  password: process.env.DB_PASSWORD || "",
  database: process.env.DB_NAME || "movie_booking",
  waitForConnections: true,
  connectionLimit: 10
});

// Check backend
app.get("/", (req, res) => {
  res.json({ message: "Movie Ticket Booking API is running!" });
});

// Get all movies
app.get("/api/movies", async (req, res) => {
  try {
    const [movies] = await pool.query(
      "SELECT * FROM movies ORDER BY id DESC"
    );
    res.json(movies);
  } catch (error) {
    res.status(500).json({ error: "Unable to fetch movies" });
  }
});

// Add a movie
app.post("/api/movies", async (req, res) => {
  try {
    const { title, genre, duration, language } = req.body;

    if (!title || !genre || !duration || !language) {
      return res.status(400).json({
        error: "All movie fields are required"
      });
    }

    const [result] = await pool.execute(
      `INSERT INTO movies (title, genre, duration, language)
       VALUES (?, ?, ?, ?)`,
      [title, genre, duration, language]
    );

    res.status(201).json({
      message: "Movie added successfully",
      movieId: result.insertId
    });
  } catch (error) {
    res.status(500).json({ error: "Unable to add movie" });
  }
});

// Get all shows
app.get("/api/shows", async (req, res) => {
  try {
    const [shows] = await pool.query(`
      SELECT s.id, m.title, s.theatre, s.show_time, s.price
      FROM shows s
      JOIN movies m ON s.movie_id = m.id
      ORDER BY s.show_time
    `);
    res.json(shows);
  } catch (error) {
    res.status(500).json({ error: "Unable to fetch shows" });
  }
});
// Book tickets
app.post("/api/bookings", async (req, res) => {
  const { customer_name, show_id, seats } = req.body;
  const seatCount = Number(seats);

  if (!customer_name || !show_id ||
      !Number.isInteger(seatCount) ||
      seatCount < 1 || seatCount > 10) {
    return res.status(400).json({
      error: "Enter a name, valid show ID, and 1–10 seats"
    });
  }

  const connection = await pool.getConnection();

  try {
    await connection.beginTransaction();

    const [shows] = await connection.execute(
      "SELECT id, price FROM shows WHERE id = ? FOR UPDATE",
      [show_id]
    );

    if (!shows.length) {
      await connection.rollback();
      return res.status(404).json({ error: "Show not found" });
    }

    const [result] = await connection.execute(
      `INSERT INTO bookings
       (customer_name, show_id, seats, total_amount)
       VALUES (?, ?, ?, ?)`,
      [
        customer_name,
        show_id,
        seatCount,
        Number(shows[0].price) * seatCount
      ]
    );

    await connection.commit();

    res.status(201).json({
      message: "Booking successful",
      bookingId: result.insertId,
      seats: seatCount,
      totalAmount: Number(shows[0].price) * seatCount
    });
  } catch (error) {
    await connection.rollback();
    res.status(500).json({ error: "Booking failed" });
  } finally {
    connection.release();
  }
});

// View booking history
app.get("/api/bookings", async (req, res) => {
  try {
    const [bookings] = await pool.query(`
      SELECT b.id, b.customer_name, m.title,
             s.theatre, s.show_time, b.seats,
             b.total_amount
      FROM bookings b
      JOIN shows s ON b.show_id = s.id
      JOIN movies m ON s.movie_id = m.id
      ORDER BY b.id DESC
    `);
    res.json(bookings);
  } catch (error) {
    res.status(500).json({ error: "Unable to fetch bookings" });
  }
});

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});CREATE DATABASE movie_booking;
USE movie_booking;

CREATE TABLE movies (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(150) NOT NULL,
    genre VARCHAR(100) NOT NULL,
    duration INT NOT NULL,
    language VARCHAR(50) NOT NULL
);

CREATE TABLE shows (
    id INT AUTO_INCREMENT PRIMARY KEY,
    movie_id INT NOT NULL,
    theatre VARCHAR(150) NOT NULL,
    show_time DATETIME NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (movie_id) REFERENCES movies(id)
);

CREATE TABLE bookings (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    show_id INT NOT NULL,
    seats INT NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (show_id) REFERENCES shows(id)
);
INSERT INTO movies (title, genre, duration, language)
VALUES
('RRR', 'Action', 187, 'Telugu'),
('Pushpa', 'Action', 179, 'Telugu'),
('Kalki 2898 AD', 'Sci-Fi', 181, 'Telugu');
INSERT INTO shows (movie_id, theatre, show_time, price)
VALUES
(1, 'City Cinemas', '2026-11-01 10:00:00', 150.00),
(2, 'City Cinemas', '2026-11-01 14:00:00', 180.00),
(3, 'City Cinemas', '2026-11-01 18:00:00', 200.00);
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_mysql_password
DB_NAME=movie_booking
PORT=3000
node server.js
API endpoints for your project
Method      Endpoint       Purpose
GET       /api/movies    View movies
POST      /api/movies    Add a movie
GET       /api/shows     View showtimes
POST      /api/bookings  Book tickets
GET       /api/bookings  View bookings
{
  "customer_name": "Pavani",
  "show_id": 1,
  "seats": 2
}
{
  "message": "Booking successful",
  "bookingId": 1,
  "seats": 2,
  "totalAmount": 300
}
