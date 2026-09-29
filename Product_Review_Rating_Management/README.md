CREATE DATABASE ProductReviewDB;

USE ProductReviewDB;

CREATE TABLE Review (
    ReviewID INT PRIMARY KEY,
    ProductID INT,
    CustomerID INT,
    ReviewText VARCHAR(500),
    ReviewDate DATE
);

CREATE TABLE Rating (
    RatingID INT PRIMARY KEY,
    ReviewID INT,
    Rating INT CHECK (Rating BETWEEN 1 AND 5),
    FOREIGN KEY (ReviewID) REFERENCES Review(ReviewID)
);

INSERT INTO Review
(ReviewID, ProductID, CustomerID, ReviewText, ReviewDate)
VALUES
(1, 101, 1, 'Good quality product', '2026-09-20'),
(2, 102, 2, 'Excellent product', '2026-09-21'),
(3, 101, 3, 'Average quality', '2026-09-22'),
(4, 103, 4, 'Very good product', '2026-09-23'),
(5, 102, 5, 'Worth the price', '2026-09-24');

INSERT INTO Rating
(RatingID, ReviewID, Rating)
VALUES
(1, 1, 4),
(2, 2, 5),
(3, 3, 3),
(4, 4, 4),
(5, 5, 5);

SELECT
    r.ReviewID,
    r.ProductID,
    r.CustomerID,
    r.ReviewText,
    rt.Rating,
    r.ReviewDate
FROM Review r
JOIN Rating rt
ON r.ReviewID = rt.ReviewID;

SELECT
    r.ProductID,
    AVG(rt.Rating) AS AverageRating
FROM Review r
JOIN Rating rt
ON r.ReviewID = rt.ReviewID
GROUP BY r.ProductID;

SELECT
    r.ProductID,
    AVG(rt.Rating) AS AverageRating
FROM Review r
JOIN Rating rt
ON r.ReviewID = rt.ReviewID
GROUP BY r.ProductID
HAVING AVG(rt.Rating) >= 4;
