---
description: >-
  This documentation provides an overview of the Java classes and methods for
  interacting with the Leaderboard API.
---

# Java Overview

### Package: `com.teamgames.endpoints.leaderboard`

The `Leaderboard` class provides methods for setting up and submitting player metrics to the leaderboard API.

### Usage Example

```java
List<PlayerMetric> metrics = new ArrayList<>();

// Add player metrics
metrics.add(new PlayerMetric("Attack").setValue(99).setProgress(1000000));
metrics.add(new PlayerMetric("Defense").setValue(99).setProgress(1000000));
// ... add more metrics

// Submit metrics to the leaderboard API
new Leaderboard()
    .setApiKey("your_api_key")
    .setGameMode("Normal Mode")
    .setPlayerName("Nelson")
    .setPlayerMetrics(metrics)
    .setDebugMessage(false)
    .submitAsync();
```

**Methods:**

* `setApiKey(String apiKey)`: Sets the API key for authentication.
* `setGameMode(String gameMode)`: Sets the game mode.
* `setPlayerName(String playerName)`: Sets the player name.
* `setPlayerMetrics(List<PlayerMetric> playerSkills)`: Sets the player metrics.
* `setDebugMessage(boolean debugMessage)`: Sets whether to display debug messages.
* `setIPAddress(String ipAddress)`: Sets the IP address of the player.
* `setUserRole(String role)`: Sets the user role.
* `addMetadata(String key, Object value)`: Adds metadata key-value pair.
* `submit()`: Submits the player metrics to the leaderboard API.
* `submitAsync()`: Submits the player metrics to the leaderboard API asynchronously

#### Class: `PlayerMetric`

The `PlayerMetric` class represents a player's metric, such as a skill level or progress.

**Methods:**

* `PlayerMetric(String name)`: Constructor that sets the name of the metric.
* `setValue(int value)`: Sets the value of the metric.
* `setProgress(int progress)`: Sets the progress of the metric.

### Package: `com.teamgames.https`

#### Class: `Post`

The `Post` class provides methods for sending POST requests to the server.

**Methods:**

* `sendPostData(Map<String, Object> params, String location, String apiKey)`: Sends a POST request with the specified parameters, location, and API key.
* `setRequestProperties(HttpURLConnection conn, byte[] postDataBytes)`: Sets the request properties for the HTTP connection.
