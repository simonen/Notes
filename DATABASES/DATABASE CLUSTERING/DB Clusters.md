
Test PHP page that returns info about the responding DB server. For testing load-balancing

```php
<?php
// Database credentials
$user = 'your_user';
$pass = 'your_password';
$db   = 'your_db';

// List of database hosts (failover order)
$hosts = ['db1.example.com', 'db2.example.com', 'db3.example.com'];

$mysqli = null;

// Try connecting to each host until one succeeds
foreach ($hosts as $host) {
    $mysqli = @new mysqli($host, $user, $pass, $db);
    if (!$mysqli->connect_error) {
        break; // success
    }
}

// If all hosts failed, stop
if ($mysqli->connect_error) {
    die("All hosts failed to connect.");
}

// Query database for info about the responding server
$result = $mysqli->query("SELECT @@hostname AS host, @@port AS port, @@version AS version, @@server_id AS server_id");
$row = $result->fetch_assoc();

// Output information
echo "Database type/version: MariaDB/MySQL " . $row['version'] . "<br>";
echo "Database host: " . $row['host'] . "<br>";
echo "Database port: " . $row['port'] . "<br>";
echo "Database server ID: " . $row['server_id'] . "<br>";

// Close the connection
$mysqli->close();
?>
```

