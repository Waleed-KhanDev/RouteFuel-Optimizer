# ⛽ Spotter Fuel Optimizer API

A high-performance Django-based backend application for optimizing fuel stops along driving routes in the United States.

The API calculates cost-effective refueling stops using a **greedy lookahead routing algorithm**, **spatial bounding boxes**, and **real-time OSRM routing data**. It combines route geometry, truck-stop proximity, vehicle fuel efficiency, tank range, and fuel prices to determine an optimized refueling strategy.

---

## 🚀 Project Overview

The **Spotter Fuel Optimizer API** accepts a starting location and destination, calculates the driving route using **OSRM**, identifies truck stops within a strict **5-mile highway corridor**, and determines optimal fuel stops based on:

* Vehicle fuel efficiency (`mpg`)
* Maximum driving range (`max_range`)
* Truck-stop location
* Fuel prices
* Route distance
* Remaining fuel and required fuel

The result is a detailed JSON response containing the optimized fuel stops, route geometry, fuel requirements, and estimated trip cost.

---

## 🏗️ Architecture & Technologies

The project is built using the following technologies:

| Technology       | Purpose                                 |
| ---------------- | --------------------------------------- |
| **Python 3.10+** | Backend runtime                         |
| **Django**       | Web framework and API backend           |
| **PostgreSQL**   | Persistent database for truck-stop data |
| **OSRM**         | Route calculation and road-network data |
| **JSON**         | Input/output API format                 |

### Core Algorithm

The optimization process uses a **greedy lookahead strategy**:

1. Calculate the driving route between the start and destination using OSRM.
2. Build a highway corridor around the calculated route.
3. Identify truck stops located within the **5-mile corridor**.
4. Evaluate reachable fuel stops based on the vehicle's remaining range.
5. Apply a greedy lookahead strategy to select cost-effective refueling points.
6. Calculate the required gallons and estimated fuel cost at each stop.
7. Return the optimized route and fuel-stop recommendations.

---

# 🛠️ Getting Started

Follow the steps below to run the project locally.

## 1. Prerequisites

Make sure the following software is installed:

* **Python 3.10 or higher**
* **PostgreSQL**
* **pip**
* **Git**

---

## 2. Clone the Repository

Clone the repository and navigate into the project directory:

```bash
git clone https://github.com/EasyBazaar-Founder/spotter-assesment
cd <project-directory>
```

---

## 3. Create a Virtual Environment

It is recommended to use a Python virtual environment:

### Windows

```bash
python -m venv venv
venv\Scripts\activate
```

### macOS / Linux

```bash
python3 -m venv venv
source venv/bin/activate
```

---

## 4. Install Dependencies

Install the project's Python dependencies:

```bash
pip install -r requirements.txt
```

---

## 5. Configure Environment Variables

The repository includes an `.env.example` template in the project root.

Create a copy and rename it to `.env`:

```bash
cp .env.example .env
```

On Windows, you can also create the file manually.

Update `.env` with your local PostgreSQL configuration:

```env
DEBUG=True
SECRET_KEY=your-django-secret-key-here

DB_NAME=spotter_fuel_db
DB_USER=postgres
DB_PASSWORD=your_database_password
DB_HOST=localhost
DB_PORT=5432
```

> **Important:** Never commit your `.env` file or production secrets to version control.

---

## 6. Create the PostgreSQL Database

Before running the seeder or starting the server, create the PostgreSQL database specified in your `.env` file.

For example:

```sql
CREATE DATABASE spotter_fuel_db;
```

You can execute this using **psql**, **pgAdmin**, or another PostgreSQL client.

---

## 7. Run Database Migrations

Apply the Django database migrations:

```bash
python manage.py migrate
```

---

## 8. Seed the Database

The project includes a database seeder located at:

```text
scripts/seed.py
```

The seeder bulk-inserts truck-stop records from `success.json` into PostgreSQL.

Run the seeder from the project root:

```bash
python scripts/seed.py
```

The script is designed to efficiently insert the available truck-stop records into the database.

---

## 9. Start the Development Server

Start the Django development server:

```bash
python manage.py runserver 8000
```

The API will be available at:

```text
http://127.0.0.1:8000/
```

---

# 📡 API Usage

## Route Optimization

The route optimization endpoint accepts a starting location, destination, vehicle range, and fuel efficiency, then returns an optimized fuel-stop plan.

### Endpoint

```http
POST /api/route-coords/
```

> The exact URL path depends on the project's Django `urlpatterns` configuration.

### Request

**Method:**

```http
POST
```

**Content-Type:**

```http
application/json
```

### Request Body

```json
{
    "start": "New York, NY, USA",
    "end": "Chicago, IL, USA",
    "max_range": 500,
    "mpg": 10.0
}
```

### Request Parameters

| Parameter   | Type     | Description                                 |
| ----------- | -------- | ------------------------------------------- |
| `start`     | `string` | Starting location                           |
| `end`       | `string` | Destination                                 |
| `max_range` | `number` | Maximum vehicle driving range               |
| `mpg`       | `number` | Vehicle fuel efficiency in miles per gallon |

---

# 📤 Response

The API returns a JSON response containing the calculated route and recommended refueling strategy.

The response includes:

### Trip Summary

Provides high-level trip information such as:

* Total route distance
* Estimated total fuel cost
* Fuel consumption assumptions
* Vehicle range and MPG information

### Route Geometry

Contains the calculated route coordinates returned by OSRM, which can be used to visualize the driving route on a map.

### Recommended Fuel Stops

Provides the optimized sequence of truck stops, including information such as:

* Truck-stop location
* Latitude and longitude
* Distance along the route
* Recommended gallons to purchase
* Retail fuel price
* Estimated cost at each stop

---

# 🧠 Optimization Strategy

The fuel optimization engine follows a route-aware greedy approach rather than simply selecting the cheapest truck stop in the database.

The general flow is:

```text
Start
  │
  ▼
Calculate Route with OSRM
  │
  ▼
Build 5-Mile Route Corridor
  │
  ▼
Find Nearby Truck Stops
  │
  ▼
Filter Reachable Stops
  │
  ▼
Greedy Lookahead Optimization
  │
  ▼
Calculate Fuel Requirements
  │
  ▼
Calculate Estimated Costs
  │
  ▼
Return Optimized Fuel Plan
```

This approach ensures that fuel stops are evaluated in the context of the actual driving route and the vehicle's available range.

---

# 📁 Project Structure

A simplified project structure is shown below:

```text
spotter-fuel-optimizer/
│
├── manage.py
├── requirements.txt
├── .env.example
├── success.json
│
├── scripts/
│   └── seed.py
│
├── <django_project>/
│   ├── settings.py
│   ├── urls.py
│   └── ...
│
└── <api_app>/
    ├── models.py
    ├── views.py
    ├── urls.py
    └── ...
```

> Replace `<django_project>` and `<api_app>` with the actual Django project and application names used in the repository.

---

# 🔌 External Services

## OSRM

The application uses **Open Source Routing Machine (OSRM)** for route calculation.

OSRM provides the road-network routing data required to determine the driving route between the supplied start and destination locations.

The resulting route geometry is then used to identify relevant truck stops along the highway corridor.

---

# ⚙️ Configuration

The application uses environment variables for configuration, particularly database credentials and Django settings.

Example:

```env
DEBUG=True
SECRET_KEY=your-django-secret-key-here

DB_NAME=spotter_fuel_db
DB_USER=postgres
DB_PASSWORD=your_database_password
DB_HOST=localhost
DB_PORT=5432
```

For production deployments:

* Set `DEBUG=False`
* Use a strong, unique `SECRET_KEY`
* Use secure database credentials
* Do not commit secrets to source control
* Configure appropriate allowed hosts and security settings

---

# 🧪 Testing

Run the Django test suite with:

```bash
python manage.py test
```

---

# 🚦 Quick Start

For a quick local setup:

```bash
# Clone repository
git clone <repository-url>
cd <project-directory>

# Create and activate virtual environment
python -m venv venv
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
# Create .env from .env.example

# Create database
# CREATE DATABASE spotter_fuel_db;

# Run migrations
python manage.py migrate

# Seed truck-stop data
python scripts/seed.py

# Start server
python manage.py runserver 8000
```

The API will then be available at:

```text
http://127.0.0.1:8000/
```

---

# 📌 Notes

* Truck stops are considered only when they fall within the configured route corridor.
* Fuel-stop selection depends on the vehicle's `mpg` and `max_range`.
* Route calculations are provided by OSRM.
* Fuel prices and truck-stop data are sourced from the project's seeded dataset.
* API responses are returned in JSON format.

---

# 📄 License

Add the project's applicable license information here.

If this project is proprietary, you may use:

```text
This project is proprietary and confidential.
Unauthorized copying, distribution, or use is prohibited.
```
