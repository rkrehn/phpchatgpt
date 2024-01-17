# About
This guide is designed to teach people how to use ChatGPT using a basic "I want one response" in PHP. This code is used in my travel itinerary generation website [ReisPlan](https://www.reisplan.net) and built off my [C# tutorial](https://github.com/rkrehn/csharpchatgpt). I love how PHP handles JSON and XML data, so this tutorial is a bit easier.

# Setup

Declare your API key (insert your [OpenAI API key](https://platform.openai.com/account/api-keys) in the quotes)

```PHP
// API endpoint URL
$url = 'https://api.openai.com/v1/chat/completions';

// Your OpenAI API key
$apiKey = '[API KEY]';
```

# Setting up the request

The following code creates the JSON request to OpenAI. You can review all the options you can include here: [https://platform.openai.com/docs/api-reference/chat/create](https://platform.openai.com/docs/api-reference/chat/create).

It's worth noting that only the "model" and the "messages" are required in the body. There are [many models](https://platform.openai.com/docs/models), but I'm using **gpt-3.5-turbo** in this example. Likewise, there are a few roles you can use such as system, user, assistance, or function. Typically, the **system** role defines what you want to accomplish. In our example, we'll tell ChatGPT **sytem** role that is acting as a travel agent (used on my [ReisPlan](https://www.reisplan.net) site).

First, we prepare the prompt statement:

```PHP
  // Prompt for the conversation
  $messages = array(
      array('role' => 'system', 'content' => 'You are a travel advisor that will deliver a detailed itinerary based on the information provided by the user.'),
      array('role' => 'user', 'content' => 'I am traveling to Denver for 7 days. Things I\'m interested in include museums and hiking.')
  );
```

Next, we build a JSON array. In the below scenario, I'm only sending the prompt and the model using an **array** and then encoding it into JSON data using the **json_encode** function.

```PHP
// Convert messages to JSON
$postData = array(
    'messages' => $messages,
    'model' => 'gpt-3.5-turbo'  // Specify the model here
);

$jsonData = json_encode($postData);
```

Next, we set up the headers using an **array** to send the request using the Authorization header with our API Key and JSON content type. The **$options** array will set up the header, method (POST), and content using the **$jsonData** we built above. 

```PHP
// Set headers for the API request
$headers = array(
    'Content-Type: application/json',
    'Authorization: Bearer ' . $apiKey
);

$options = array(
    'http' => array(
        'header'  => implode("\r\n", $headers),
        'method'  => 'POST',
        'content' => $jsonData
    )
);
```

The **stream_context_create** function will create the send data to OpenAI using an HTTP request.

```PHP
$context = stream_context_create($options);
```

Finally, we'll execute the API request using **file_get_contents** function and check for errors.

```PHP
// Execute the API request
$response = file_get_contents($url, false, $context);

// Check for errors
if ($response === false) {
    $error = error_get_last()['message'];
    echo 'API request failed: ' . $error;
    exit;
}
```

# Receiving the response

We'll just the **json_decode** function to read the $response variable from OpenAI.

```PHP
// Decode the API response
$data = json_decode($response, true);
```
You will receive a JSON response like this in the **$data** variable (don't copy this):

```JSON
{
  "id": "chatcmpl-123",
  "object": "chat.completion",
  "created": 1677652288,
  "model": "gpt-3.5-turbo-0613",
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "content": "\n\nSure, I can recommend you check out Livin the Dream brewery in Littleton and Denver Beer Co in Englewood.",
    },
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 9,
    "completion_tokens": 12,
    "total_tokens": 21
  }
}
```

* choices[0] is the first response
* message is the node within the choices[0]
* content is the response you're looking for

Next, we'll extract the **$data** using the above and then explode individual lines in the response. The **$reply** variable contains the whole response.

But, I used **$lines** to break the reply response so I could get multiple lines of data and have them display on the website.

```PHP
// Extract the assistant's reply
$reply = $data['choices'][0]['message']['content'];

// Output the reply
// Assuming $reply contains the response from the API
$lines = explode("\n", $reply);
```

The following loop examines each **$line** received and displays it on the website using the echo function.

```PHP
// Loop through the lines
$previousLineEmpty = false;
foreach ($lines as $line) {
    // Process each line as needed
    if (empty($line)) {
        $previousLineEmpty = true;
    } else {
        if ($previousLineEmpty) {
            echo "<br><strong>" . $line . "</strong><br>"; // Output the line in bold
            $previousLineEmpty = false;
        } else {
            echo $line . "<br>"; // Output each line
        }
    }
}
```


# Everything together

Now, put it together in all its glory:

```PHP
// API endpoint URL
$url = 'https://api.openai.com/v1/chat/completions';

// Your OpenAI API key
$apiKey = '[API KEY]';

// Prompt for the conversation
$messages = array(
    array('role' => 'system', 'content' => 'You are a travel advisor that will deliver a detailed itinerary based on the information provided the user.'),
    array('role' => 'user', 'content' => 'I am traveling to Denver for 7. Things I\'m interested in include breweries.')
);

// Convert messages to JSON
$postData = array(
    'messages' => $messages,
    'model' => 'gpt-3.5-turbo'  // Specify the model here
);

$jsonData = json_encode($postData);

// Set headers for the API request
$headers = array(
    'Content-Type: application/json',
    'Authorization: Bearer ' . $apiKey
);

$options = array(
    'http' => array(
        'header'  => implode("\r\n", $headers),
        'method'  => 'POST',
        'content' => $jsonData
    )
);

$context = stream_context_create($options);

// Execute the API request
$response = file_get_contents($url, false, $context);

// Check for errors
if ($response === false) {
    $error = error_get_last()['message'];
    echo 'API request failed: ' . $error;
    exit;
}

// Decode the API response
$data = json_decode($response, true);

// Extract the assistant's reply
$reply = $data['choices'][0]['message']['content'];
```

> Sure, I can recommend you check out Livin the Dream brewery in Littleton and Denver Beer Co in Englewood.
