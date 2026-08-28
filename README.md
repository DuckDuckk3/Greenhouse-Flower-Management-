# Greenhouse Flower Management

A greenhouse flower management application built using a three-tier architecture:

* **Frontend:** Pure HTML, CSS, and JavaScript.
* **Backend:** Node.js, Express, and REST API.
* **Database:** Microsoft SQL Server.

Sensor data is currently simulated at the backend. MQTT has not yet been integrated; when integrated, MQTT payloads will pass through the same processing pipeline currently used by the simulator.

## Key Features

* Dashboard for monitoring temperature, humidity, light, CO2, and alerts.
* Management of the `Farm → Greenhouse → Zone` structure.
* Sensor and actuator management.
* Device control in `AUTO` or `MANUAL` mode.
* Recipe, GrowthStage, and Threshold management.
* Recipe progress tracking for each Zone.
* Individual cycle adjustment for each Zone with a recorded reason.
* Alert creation, acknowledgment, and resolution.
* Audit Log for user operations and automatic actions.
* Account management, role-based access control, and password changes.

The sensor metrics currently used are: `Temperature`, `Humidity`, `SoilHumidity`, `Light`, `CO2`, and `PH`.

## Project Structure

```text
Greenhouse-Flower-Management/
├── index.html
├── css/                  # User interface
├── js/                   # Frontend logic and API client
├── smartfarm-api/        # Express API and simulator
├── SQLQuery1.sql         # Database schema and sample data
└── README.md
```

## Environment Requirements

* Node.js 18 or later.
* Microsoft SQL Server.
* A modern web browser.
* VS Code Live Server or another static HTTP server.

## Database Setup

1. Open SQL Server Management Studio.
2. Run `SQLQuery1.sql` on SQL Server.
3. The script will create the `SmartFarmDB` database, tables, and test data.

> `SQLQuery1.sql` is a complete initialization script and should be run on a clean database. Do not run it repeatedly on a database where the tables already exist.

## Backend Configuration

Inside the `smartfarm-api` directory, create a `.env` file based on `.env.example`:

```env
DB_HOST=localhost
DB_DATABASE=SmartFarmDB
DB_USER=sa
DB_PASSWORD=your-password
DB_PORT=1433
PORT=5000
SIMULATION_ENABLED=true
SIMULATION_INTERVAL_MS=10000
```

Do not commit the `.env` file or real passwords to GitHub.

## Running the Application

### 1. Backend

Open a terminal in the `smartfarm-api` directory:

```bash
cd smartfarm-api
npm install
npm start
```

The backend runs by default at:

```text
http://localhost:5000
```

### 2. Frontend

Open a terminal in the project root directory:

```bash
npx http-server -o
```

You can also open `index.html` using VS Code Live Server.

It is not recommended to open `index.html` directly using `file://` because the frontend uses JavaScript modules.

## Demo Accounts

| **Role**         | **Username**       | **Password** |
| ---------------- | ------------------ | ------------ |
| Primary OWNER    | `greenhouse_owner` | `demo123`    |
| TECHNICIAN       | `agronomist`       | `demo123`    |
| OPERATOR         | `operator_a`       | `demo123`    |
| Pending Approval | `pending_operator` | `demo123`    |

The system may have multiple OWNER accounts, but only one account can have `is_primary_owner = 1`:

* The primary OWNER does not appear in the account management list.
* The primary OWNER cannot be deleted, disabled, or have their role changed.
* Secondary OWNER accounts can be assigned permissions or deleted like regular accounts.
* Only an active OWNER can access the account management page and user-management APIs.

Passwords are stored as plain text in this academic project for demonstration purposes. A production system should use bcrypt/Argon2 and secure JWT or session-based authentication.

## Recipe and Zone Progress

`Recipe`, `GrowthStage`, and `Threshold` define the template growing configuration. Progress is not stored directly on the Recipe. Instead, it is calculated separately for each Zone based on:

```text
current_day = today - start_date + 1
adjusted_day = current_day - cycle_adjustment_days
```

Each Zone displays:

* Current growth stage.
* Progress percentage.
* GrowthStage timeline.
* Expected completion date.
* Thresholds for the current stage.
* Number of adjusted days and the reason for the cycle adjustment.

Adjusting one Zone does not modify the Recipe or other Zones using the same Recipe.

## Backend Simulator

The simulator generates data every 10 seconds by default for active sensors. Each generated value:

1. Is stored in `SensorData`.
2. Updates the Zone temperature or humidity when applicable.
3. Is compared with the Threshold of the current GrowthStage.
4. Creates, updates, or closes an `AlertLog`.
5. Controls an `AUTO` actuator when necessary.
6. Records automatic actions in the Audit Log.

### Simulator Testing API

| **Method** | **Endpoint**               | **Description**                      |
| ---------- | -------------------------- | ------------------------------------ |
| GET        | `/api/simulation/status`   | Check simulator status               |
| POST       | `/api/simulation/start`    | Start the simulator                  |
| POST       | `/api/simulation/stop`     | Stop the simulator                   |
| POST       | `/api/simulation/tick`     | Run one simulation cycle immediately |
| POST       | `/api/simulation/readings` | Send a test sensor reading           |

Example request body for a test sensor value:

```json
{
  "deviceId": 1,
  "value": 35
}
```

## Audit Log

The Audit Log supports the following object types:

```text
DEVICE, ZONE, GREENHOUSE, FARM, RECIPE,
GROWTH_STAGE, ALERT, USER, SIMULATION
```

Logs are created for login/logout, password changes, account management, CRUD operations on greenhouse structures, devices, recipes, alerts, manual control actions, and automatic simulator actions.

## Testing

Inside the `smartfarm-api` directory:

```bash
npm test
npm run test:integration
```

* `npm test`: checks syntax and pure logic functions.
* `npm run test:integration`: tests the `SensorData → AlertLog → AUTO actuator` workflow with SQL Server and automatically cleans up test data.

## Deployment Notes

* The frontend calls the API at `http://localhost:5000/api`.
* The Dashboard and Log pages automatically refresh every 10 seconds while open.
* If the backend source code is changed, stop and restart the backend using `npm start`.
* If the browser is still using cached JavaScript, press `Ctrl + F5`.
