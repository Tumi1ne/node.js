# node.js
Simple Serve: Mock Media Server

const http = require('http');
const url = require('url');
const { v4: uuidv4 } = require('uuid');

// Sample data
let movies = [
  {
    id: uuidv4(),
    title: 'Inception',
    director: 'Christopher Nolan',
    year: 2010,
    genre: ['Sci-Fi', 'Action', 'Thriller'],
    cast: ['Leonardo DiCaprio', 'Joseph Gordon-Levitt', 'Elliot Page'],
  },
  {
    id: uuidv4(),
    title: 'The Shawshank Redemption',
    director: 'Frank Darabont',
    year: 1994,
    genre: ['Drama'],
    cast: ['Tim Robbins', 'Morgan Freeman'],
  },
];

let series = [
  {
    id: uuidv4(),
    title: 'Breaking Bad',
    creator: 'Vince Gilligan',
    yearsAired: '2008-2013',
    genre: ['Crime', 'Drama', 'Thriller'],
    seasons: [
      { seasonNumber: 1, episodes: 7 },
      { seasonNumber: 2, episodes: 13 },
    ],
  },
  {
    id: uuidv4(),
    title: 'The Queen\'s Gambit',
    creator: 'Scott Frank, Allan Scott',
    yearsAired: '2020',
    genre: ['Drama', 'History'],
    seasons: [{ seasonNumber: 1, episodes: 7 }],
  },
];

let songs = [
  {
    id: uuidv4(),
    title: 'Bohemian Rhapsody',
    artist: 'Queen',
    album: 'A Night at the Opera',
    year: 1975,
    genre: ['Rock', 'Opera'],
  },
  {
    id: uuidv4(),
    title: 'Like a Rolling Stone',
    artist: 'Bob Dylan',
    album: 'Highway 61 Revisited',
    year: 1965,
    genre: ['Folk Rock'],
  },
];

const server = http.createServer((req, res) => {
  const parsedUrl = url.parse(req.url, true);
  const path = parsedUrl.pathname;
  const method = req.method;

  res.setHeader('Content-Type', 'application/json');

  // Helper function to handle GET requests
  const handleGet = (data) => {
    res.writeHead(200);
    res.end(JSON.stringify(data));
  };

  // Helper function to handle POST requests
  const handlePost = (dataArray, req, res) => {
    let body = '';
    req.on('data', (chunk) => {
      body += chunk;
    });
    req.on('end', () => {
      try {
        const newItem = JSON.parse(body);
        newItem.id = uuidv4(); // Assign a unique ID
        dataArray.push(newItem);
        res.writeHead(201); // Created
        res.end(JSON.stringify(dataArray));
      } catch (error) {
        res.writeHead(400); // Bad Request
        res.end(JSON.stringify({ message: 'Invalid JSON data' }));
      }
    });
  };

  // Helper function to handle PUT requests
  const handlePut = (dataArray, req, res) => {
    const segments = path.split('/').filter(Boolean);
    if (segments.length === 2) {
      const itemId = segments[1];
      let body = '';
      req.on('data', (chunk) => {
        body += chunk;
      });
      req.on('end', () => {
        try {
          const updatedItem = JSON.parse(body);
          const index = dataArray.findIndex((item) => item.id === itemId);
          if (index !== -1) {
            dataArray[index] = { ...dataArray[index], ...updatedItem };
            res.writeHead(200); // OK
            res.end(JSON.stringify(dataArray));
          } else {
            res.writeHead(404); // Not Found
            res.end(JSON.stringify({ message: `${segments[0].slice(0, -1)} not found` }));
          }
        } catch (error) {
          res.writeHead(400); // Bad Request
          res.end(JSON.stringify({ message: 'Invalid JSON data' }));
        }
      });
    } else {
      res.writeHead(400); // Bad Request
      res.end(JSON.stringify({ message: 'Invalid request format for PUT' }));
    }
  };

  // Helper function to handle DELETE requests
  const handleDelete = (dataArray, req, res) => {
    const segments = path.split('/').filter(Boolean);
    if (segments.length === 2) {
      const itemId = segments[1];
      const initialLength = dataArray.length;
      dataArray = dataArray.filter((item) => item.id !== itemId);
      if (dataArray.length < initialLength) {
        res.writeHead(200); // OK
        res.end(JSON.stringify(dataArray));
      } else {
        res.writeHead(404); // Not Found
        res.end(JSON.stringify({ message: `${segments[0].slice(0, -1)} not found` }));
      }
    } else {
      res.writeHead(400); // Bad Request
      res.end(JSON.stringify({ message: 'Invalid request format for DELETE' }));
    }
  };

  if (path === '/movies') {
    if (method === 'GET') {
      handleGet(movies);
    } else if (method === 'POST') {
      handlePost(movies, req, res);
    } else if (method === 'PUT') {
      handlePut(movies, req, res);
    } else if (method === 'DELETE') {
      handleDelete(movies, req, res);
    } else {
      res.writeHead(405); // Method Not Allowed
      res.end(JSON.stringify({ message: 'Method not allowed for /movies' }));
    }
  } else if (path === '/series') {
    if (method === 'GET') {
      handleGet(series);
    } else if (method === 'POST') {
      handlePost(series, req, res);
    } else if (method === 'PUT') {
      handlePut(series, req, res);
    } else if (method === 'DELETE') {
      handleDelete(series, req, res);
    } else {
      res.writeHead(405); // Method Not Allowed
      res.end(JSON.stringify({ message: 'Method not allowed for /series' }));
    }
  } else if (path === '/songs') {
    if (method === 'GET') {
      handleGet(songs);
    } else if (method === 'POST') {
      handlePost(songs, req, res);
    } else if (method === 'PUT') {
      handlePut(songs, req, res);
    } else if (method === 'DELETE') {
      handleDelete(songs, req, res);
    } else {
      res.writeHead(405); // Method Not Allowed
      res.end(JSON.stringify({ message: 'Method not allowed for /songs' }));
    }
  } else {
    res.writeHead(404); // Not Found
    res.end(JSON.stringify({ message: 'Not Found' }));
  }
});

const PORT = 3000;
server.listen(PORT, () => {
  console.log(`Server listening on port ${PORT}`);
});