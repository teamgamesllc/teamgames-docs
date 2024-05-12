# Callback

### Overview

This guide provides documentation for integrating your game server with the TeamGames Toplist. This integration is only if you're using your own vote system, if you're using our API then you should already be ready to go. Benefits include access to exclusive features and updates as well as detailed Vote Analytics.

### Contents

* Registration Process
* Voting Link Configuration
* Receiving Vote Callbacks
* PHP Callback Example
* Technical Support

### Registration Process

#### Steps to Register

1. **Create an Account:** Initiate the process by creating a TeamGames account. This can be done through the following link: [TeamGames Registration](https://teamgames.io/signup).
2. **Server Setup:** Post-registration, you will be directed to the server setup page.
3. **Callback URL Configuration:** Enter the callback URL for your server. This URL is where TeamGames will send notifications for each vote

### Voting Link Configuration

#### URL Format for Voting

Players should be directed to vote via the following URL:

`https://everythingrs.com/server/{sid}/?callback={incentive}`

* `{sid}`: Represents the subdomain assigned to your game server at registration.
* `{incentive}`: A string that is part of your voting script, which is used to process the incentive through your callback URL.

### Receiving Vote Callbacks

#### Vote Data Specifications

When a vote is recorded, TeamGames will transmit the following data to your server:

* **Username**: The voter's username.
* **IP Address**: The IP address from which the vote was made.

Your system must be configured to receive and process this information appropriately.

### PHP Callback Example

#### Implementing the Callback

Below is an example in PHP demonstrating how to handle incoming vote data:

```php
<?php

// Extracting data from the POST request
$username = $_POST['username'] ?? 'unknown';
$ipAddress = $_POST['ipAddress'] ?? '0.0.0.0';

// Handling the vote data
try {
    $pdo = new PDO('mysql:host=your_host;dbname=your_db', 'username', 'password');
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

    $stmt = $pdo->prepare("INSERT INTO votes (username, ip_address, vote_time) VALUES (:username, :ipAddress, NOW())");
    $stmt->bindParam(':username', $username);
    $stmt->bindParam(':ipAddress', $ipAddress);
    $stmt->execute();

    echo json_encode(['status' => 'success', 'message' => 'Vote recorded successfully.']);
} catch (PDOException $e) {
    echo json_encode(['status' => 'error', 'message' => 'Failed to record vote.']);
}
?>
```
