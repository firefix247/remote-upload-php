## Remote Upload in php
How to remotely upload a file to a php host without having to download the file on one's PC first


# Step 1: Create the HTML Form (get.html)

First, create a simple HTML file with a form for the user to input the URL.


```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Remote File Uploader</title>
    <style>
        body { font-family: sans-serif; margin: 2em; }
        .container { max-width: 600px; margin: 0 auto; padding: 20px; border: 1px solid #ccc; border-radius: 8px; }
        input[type="text"] { width: 100%; padding: 10px; margin: 10px 0; box-sizing: border-box; }
        button { padding: 10px 20px; background-color: #007bff; color: white; border: none; cursor: pointer; }
    </style>
</head>
<body>
    <div class="container">
        <h2>Remote File Uploader</h2>
        <form action="get.php" method="post">
            <label for="url">Enter the URL of the file to upload:</label>
            <input type="text" id="url" name="url" placeholder="e.g., https://downloads.wordpress.org/release/wordpress-6.5.4.zip" required>
            <button type="submit">Upload File</button>
        </form>
    </div>
</body>
</html>
```


# Step 2: Create the PHP Script (get.php)

Next, create the get.php script that will process the form submission. This script uses cURL to handle the file download, including automatically following redirects.

```
<?php
// Function to upload a file from a URL using cURL
function remoteUploadCurl($url, $destination) {
    // Check if cURL is installed and enabled
    if (!function_exists('curl_init')) {
        return "Error: cURL is not installed. Please enable it in your PHP configuration.";
    }

    $ch = curl_init();

    // Set cURL options
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true); // Return the transfer as a string
    curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true); // Follow redirects

    // Mimic a browser's User-Agent to avoid server-side blocking
    curl_setopt($ch, CURLOPT_USERAGENT, 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36');

    // Execute the cURL request to get the file content
    $fileContent = curl_exec($ch);

    // Check for cURL errors
    if (curl_errno($ch)) {
        $error_msg = curl_error($ch);
        curl_close($ch);
        return "cURL Error: " . $error_msg;
    }

    // Get the HTTP status code
    $http_code = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);

    // Check if the request was successful (HTTP 200 OK)
    if ($http_code !== 200) {
        return "Error: HTTP status code " . $http_code . ". File not found or could not be downloaded.";
    }

    // Save the content to the local destination
    if (file_put_contents($destination, $fileContent) !== false) {
        return "Success: The file was successfully uploaded to " . $destination;
    } else {
        return "Error: Could not save the file to the destination.";
    }
}

// --- Main script logic ---
if ($_SERVER["REQUEST_METHOD"] === "POST" && !empty($_POST['url'])) {
    $url = filter_var($_POST['url'], FILTER_SANITIZE_URL);

    // Sanity check to ensure a valid URL format
    if (!filter_var($url, FILTER_VALIDATE_URL)) {
        echo "Error: The provided URL is not valid.";
        exit;
    }
    
    // Extract the filename from the URL
    $filename = basename($url);

    // Set a safe destination directory
    $destinationDir = "uploads/";

    // Create the 'uploads' directory if it doesn't exist
    if (!is_dir($destinationDir)) {
        mkdir($destinationDir, 0755, true);
    }

    $destinationPath = $destinationDir . $filename;

    // Call the cURL function to upload the file
    echo remoteUploadCurl($url, $destinationPath);

} else {
    // Redirect to the form if accessed directly
    header('Location: get.html');
    exit;
}
?>
```
