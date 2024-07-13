# voice-to-text

## 1. Set up XAMPP and VS Code:
-  Install XAMPP on your system, which includes Apache, MySQL, PHP and other components needed for web development.
-  Install VS Code, a popular code editor, on your system.
## 2. Create a new project directory:
 -  In the htdocs folder of XAMPP, create a new directory for your project:`text`.
## 3. Database setup:
 -  Register XAMMP control panel and we can allow Apache and MySQL modules.
 -  Within XAMMP, go to Admin to access the MySQL management interface.
 -  Create a new database, on `voice_to_text`.
 - Create the `transcripts` table within SQL:
 ```
   CREATE TABLE transcripts (
  id INT AUTO_INCREMENT PRIMARY KEY,
  content TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
## 4. Create a voice-to-text conversion page:
-  In VS Code, create a new file named `index.php` in the `text` directory.
  ```
<!DOCTYPE html>
<html>
<head>
  <title>Audio to Text Converter</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f5f5f5;
      margin: 0;
      padding: 0;
    }
    
    .container {
      max-width: 800px;
      margin: 0 auto;
      padding: 40px;
    }
    
    h1 {
      text-align: center;
      margin-bottom: 30px;
    }
    
    #transcript {
      background-color: white;
      border: 1px solid #ccc;
      padding: 20px;
      border-radius: 5px;
      min-height: 200px;
      white-space: pre-wrap;
      font-size: 16px;
      line-height: 1.5;
      margin-top: 30px;
    }
    
    #startRecording {
      display: block;
      margin: 0 auto;
      padding: 10px 20px;
      font-size: 16px;
      background-color: #a0a0a0;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    
    #startRecording:hover {
      background-color: #a0a0a0;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Audio to Text Converter</h1>
    <button id="startRecording">Start Recording</button>
    <div id="transcript"></div>
  </div>

  <script>
    const startRecordingButton = document.getElementById('startRecording');
    const transcriptDiv = document.getElementById('transcript');

    let recognition;

    startRecordingButton.addEventListener('click', () => {
      if (!recognition) {
        recognition = new webkitSpeechRecognition();
        recognition.continuous = true;
        recognition.interimResults = true;

        recognition.onresult = (event) => {
          let transcript = '';
          for (let i = event.resultIndex; i < event.results.length; i++) {
            if (event.results[i].isFinal) {
              transcript += event.results[i][0].transcript + '\n';
            }
          }
          transcriptDiv.textContent = transcript;
        };

        recognition.start();
        startRecordingButton.textContent = 'Stop Recording';
      } else {
        recognition.stop();
        startRecordingButton.textContent = 'Start Recording';

        // Send the transcript to the server for storage in the database
        sendTranscriptToServer(transcriptDiv.textContent);
      }
    });

    function sendTranscriptToServer(transcript) {
      // Create a new XMLHttpRequest object
      const xhr = new XMLHttpRequest();

      // Set the request method and URL
      xhr.open('POST', 'save_transcript.php', true);

      // Set the request headers
      xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');

      // Prepare the data to send
      const data = 'transcript=' + encodeURIComponent(transcript);

      // Send the request
      xhr.send(data);

      // Handle the response
      xhr.onreadystatechange = function() {
        if (xhr.readyState === 4 && xhr.status === 200) {
          console.log('Transcript saved to the database.');
        }
      };
    }
  </script>
</body>
</html>
  ```
## 5.Create a php page:
-  In VS Code, create a new file named `save_transcript.php` in the `text` directory.
```
?php
// Database connection details
$servername = "localhost";
$username = "root";
$password = "";
$dbname = "voice_to_text";

// Create connection
$conn = new mysqli($servername, $username, $password, $dbname);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

// Validate and get the transcript from the POST data
if (isset($_POST['transcript']) && !empty($_POST['transcript'])) {
    $transcript = $_POST['transcript'];
    error_log("Received transcript: " . $transcript);

    // Prepare the SQL query
    $sql = "INSERT INTO transcripts (content, created_at) VALUES (?, NOW())";
    $stmt = $conn->prepare($sql);

    if ($stmt) {
        $stmt->bind_param("s", $transcript);

        if ($stmt->execute()) {
            echo "Transcript saved to the database.";
        } else {
            $errorMessage = "Error executing SQL query: " . $stmt->error;
            error_log($errorMessage);
            echo $errorMessage;
        }

        // Close the prepared statement
        $stmt->close();
    } else {
        $errorMessage = "Error preparing SQL statement: " . $conn->error;
        error_log($errorMessage);
        echo $errorMessage;
    }
} else {
    $errorMessage = "Error: Transcript data is missing or empty.";
    error_log($errorMessage);
    echo $errorMessage;
    exit; // Exit the script to prevent further execution
}

// Close the database connection
$conn->close();
?>
```
## audio to text interface:
<img src="https://github.com/user-attachments/assets/57961dbc-e7c3-45c3-beee-21d30672661f" width="600" height="300" alt="My Image">

<img src="https://github.com/user-attachments/assets/8c67dd35-22ec-444d-b83d-562b663aaff1" width="600" height="300" alt="My Image">

## database:
![Screenshot 2024-07-13 085402](https://github.com/user-attachments/assets/f99f5ff0-a59e-488f-a3d3-1557ac28495b)


