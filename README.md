# Junior MUD Project

A Multiplayer Text-Based Dungeon/Domain (MUD) built with Node.js, Express, WebSockets, React, Hooks, PostgreSQL, and Redis.

**Purpose**

For those that aren't familiar, MUDs were text-based environments used mostly for gaming in the 80's and 90's. Try to imagine a series of connected chat rooms. They have mostly fallen to side because of advances in graphics engines and game companies. However, those same advances make creating and experimenting with one extremely inexpensive and educational.

MUDs had an element that is not seen in more contemporary gaming. Most games would allow players who completed them to become immortals/wizards, effectively making them programmers on the game itself. What recruiting strategy!

The entire purpose of this project is to create a starting point for Apprentice, Junior, and even some Mid-level Engineers to play around with less familiar technologies.

Namely, these are databases (Postgres), caching (Redis), and sockets (socket.io).

As a consequence, there are a lot of other technologies that come along with those, e.g. React, Express, and Node. This is a good place to play with other standard development technologies like ESLint and Vite. Even CSS, if you like! There's enough room here to even add a library you'd like to try out!

Everyone, regardless of level, can use this project as an educational environment. If you follow this README, it should run, but just barely. There are tons of (intentional) bugs. The whole point is to have a safe and interesting space to get some practice refactoring.

My next project will be to provide a more stable and much better developed version.

Additionally, this project sees some traction, I'll happily add some challenges and refactoring suggestions. In the meantime, fork away!

This project is licensed under the **Creative Commons Attribution-NonCommercial 4.0 International License**.

## Getting Started

Follow these instructions to get a copy of the project up and running on your local machine for development and testing purposes.

### Prerequisites

1. **Node.js** and **npm** (or `pnpm`)
2. **PostgreSQL** (v12+)
3. **Redis**

### Installation

1. **Clone the Repository**:

   ```bash
   git clone [https://github.com/your-username/mud-project.git](https://github.com/jacobpaine/JuniorMUD.git)
   cd JuniorMud
   ```

2. **Install dependencies**

To download pnpm, run this from anywhere:

```bash
npm install -g pnpm
```

To install all dependencies, in both the backend and frontend folders, run:

```bash
cd backend
pnpm install

cd ../frontend
pnpm install
```

### Setting Up PostgreSQL

1. **Install PostgreSQL**:

   If you don't already have PostgreSQL installed, you can install it based on your operating system:

   - **macOS** (using Homebrew):

     ```bash
     brew install postgresql@14
     ```

     After installation, you can start PostgreSQL with:

     ```bash
     brew services start postgresql@14
     ```

   - **Ubuntu/Linux**:

     ```bash
     sudo apt update
     sudo apt install postgresql postgresql-contrib
     ```

   - **Windows**:
     You can download the installer from the [official PostgreSQL website](https://www.postgresql.org/download/windows/).

---

### Starting PostgreSQL

After installing PostgreSQL, you need to start the service to interact with the database.

#### macOS (using Homebrew)

If you installed PostgreSQL via Homebrew, you can start PostgreSQL with the following command:

```bash
brew services start postgresql@14
```

You can also stop or restart PostgreSQL with:

```bash
brew services stop postgresql@14
brew services restart postgresql@14
```

To check the status of PostgreSQL, run:

```bash
brew services list
```

**Ubuntu/Linux**
For Linux systems, you can start the PostgreSQL service using the following command:

```bash
sudo service postgresql start
```

To stop or restart PostgreSQL:

```bash
sudo service postgresql stop
sudo service postgresql restart
```

You can check the status of the PostgreSQL service with:

```bash
sudo service postgresql status
```

**Windows**
On Windows, PostgreSQL is usually installed as a service, so it should start automatically. If not, follow these steps:

Open the Services app (you can search for "Services" in the Start menu).
Find PostgreSQL in the list of services.
Right-click on PostgreSQL and choose Start, Stop, or Restart.

**Verifying PostgreSQL is Running**
Once PostgreSQL is started, you can verify that it's running by connecting to the PostgreSQL CLI (psql):

```bash
psql -U postgres
```

### Log into PostgreSQL

First, log in to PostgreSQL using the default superuser postgres. Open a terminal and run:

```bash
psql -U postgres
```

You will be logged into the PostgreSQL shell as the postgres user.

**Create a New Database**
In the PostgreSQL shell, use the following command to create a new database for your project (replace mud_game with the name of your database):

```sql
CREATE DATABASE mud_game;
```

**Create a New User**
Next, create a new user (replace mud_admin and yourpassword with the desired username and password):

```sql
CREATE USER mud_admin WITH ENCRYPTED PASSWORD 'yourpassword';
```

**Grant Privileges to the User**
Now grant all privileges on the newly created database (mud_game) to the new user (mud_admin):

```sql
GRANT ALL PRIVILEGES ON DATABASE mud_game TO mud_admin;
```

**Exit the PostgreSQL Shell**
To exit the PostgreSQL shell, type:

```sql
\q
```

**Verifying the User and Database**
To verify that everything is set up correctly, log in with the new user:

```bash
psql -U mud_admin -d mud_game
```

## Define the Postgres Tables:

```sql
CREATE TABLE players (
  id SERIAL PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL,
  password VARCHAR(100) NOT NULL,
  health INT NOT NULL DEFAULT 100,
  current_room_id INT,
  level INT DEFAULT 1,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE rooms (
  id SERIAL PRIMARY KEY,
  description TEXT NOT NULL,
  north_room_id INT,
  south_room_id INT,
  east_room_id INT,
  west_room_id INT,
  detailed_description TEXT
);

CREATE TABLE items (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  detailed_description TEXT,
  current_room_id INT REFERENCES rooms(id) ON DELETE SET NULL,
  held_by_player_id INT REFERENCES players(id) ON DELETE SET NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE player_inventory (
  player_id INT REFERENCES players(id) ON DELETE CASCADE,
  item_id INT REFERENCES items(id) ON DELETE CASCADE,
  quantity INT DEFAULT 1,
  PRIMARY KEY (player_id, item_id)
);
```

**Add Data to the Tables**

```sql
INSERT INTO players (username, password, health, current_room_id, level)
VALUES
('player1', 'password123', 100, 1, 1),
('player2', 'password456', 90, 2, 2),
('player3', 'password789', 75, 3, 3);

INSERT INTO rooms (description, north_room_id, south_room_id, east_room_id, west_room_id, detailed_description)
VALUES
('A dark and eerie room with strange symbols etched into the walls.', 2, NULL, NULL, NULL, 'The symbols glow faintly as you approach them.'),
('A brightly lit room filled with plants and flowers.', 3, 1, NULL, NULL, 'The air smells fresh, and sunlight streams in from above.'),
('A cold, damp cave with stalactites hanging from the ceiling.', NULL, 2, NULL, NULL, 'The sound of dripping water echoes in the cave.');

INSERT INTO items (name, description, detailed_description, current_room_id, held_by_player_id)
VALUES
  ('Sword', 'A sharp sword with a gleaming blade.', 'The hilt of the sword is adorned with intricate carvings.', 1, NULL),
  ('Shield', 'A sturdy shield made of iron.', 'The shield has a few dents from previous battles.', 2, NULL),
  ('Torch', 'A simple wooden torch.', 'The torch flickers faintly, providing a dim light.', NULL, 1),
  ('Map', 'A map of the surrounding area.', 'The map is old and faded, but still readable.', NULL, 2);

  INSERT INTO player_inventory (player_id, item_id, quantity) VALUES
  (1, 1, 1),  -- Player 1 has 1 Sword
  (2, 2, 1),  -- Player 2 has 1 Shield
  (1, 3, 2),  -- Player 1 has 2 Torches
  (2, 4, 1);  -- Player 2 has 1 Map
```

**Verify the Data**:

Once the data is inserted, you can verify that everything is set up correctly by querying the tables:

```sql
SELECT * FROM players;
SELECT * FROM rooms;
SELECT * FROM items;
```

### Setting Up Redis

Redis is used for session management and caching in this project. Follow these steps to install and set up Redis on your machine.

#### Install Redis

##### macOS (using Homebrew)

If you're using macOS and have Homebrew installed, you can install Redis with:

```bash
brew install redis
```

On Linux systems, you can install Redis using the package manager:

```bash
sudo apt update
sudo apt install redis-server
```

Windows
On Windows, you can install Redis using WSL (Windows Subsystem for Linux) or download Redis from Microsoft’s OpenTech Redis GitHub.

Alternatively, you can use Docker to run Redis on Windows:

```bash
docker run --name redis -d redis
```

### Start Redis

macOS and Linux
To start Redis on macOS or Linux, use the following command:

```bash
redis-server
```

To stop Redis:

```bash
redis-cli shutdown
```

If you're using Homebrew (macOS), you can also use the brew services command to start Redis:

```bash
brew services start redis
```

Windows (with Docker)
If you're using Docker to run Redis, make sure the Redis container is running:

```bash
docker start redis
```

**Check if Redis is Running**

Once Redis is started, you can verify that it's running by using the ping command:

```bash
redis-cli ping
```

If Redis is running, it will return PONG.

## Start the MUD

**Start the Backend**
To start the backend, run the following command:

```bash
pnpm run start
```

**Start the Frontend**
In another terminal window, navigate to the frontend folder and start the frontend:

```bash
cd frontend
pnpm run dev
```

## License

This project is licensed for **educational purposes only**.

You are free to use, modify, and distribute this work as long as it is done for **non-commercial educational purposes**. Commercial use is strictly prohibited.

For more details, see the full [license](./LICENSE).
