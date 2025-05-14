# music-api-documentation


## Base URL

 `https://api.musicapihost.xyz`

## Endpoints

### 1. Recognize Recording

Identifies music from audio recordings.

**Endpoint:** `POST /api/recognize-recording`

**Request Format:**

python

```python
import requests

def recognize_recording(audio_file_path):
    url = "https://api.musicapihost.xyz/api/recognize-recording"
    
    with open(audio_file_path, 'rb') as file:
        files = {'audioData': file}
        response = requests.post(url, files=files)
    
    if response.status_code == 200:
        if response.text == 'fail':
            print("No match found")
        else:
            result = response.json()
            print(f"Recognized: {result.get('track', {}).get('title')} by {result.get('track', {}).get('subtitle')}")
    else:
        print(f"Error: {response.status_code}")
    
    return response.json() if response.status_code == 200 else None

# Example usage
result = recognize_recording("sample_audio.wav")
```

**Response Format:**

-   Success: JSON object containing track information
-   No match: String "fail"
-   Error: HTTP error status

### 2. Recognize File

Identifies music from uploaded video/audio files at a specific timestamp.

**Endpoint:** `POST /api/recognize-file`

**Request Format:**

python

```python
import requests

def recognize_file(file_path, start_time=0):
    url = "https://api.musicapihost.xyz/api/recognize-file"
    
    with open(file_path, 'rb') as file:
        files = {'videoFile': file}
        data = {'startTime': start_time}
        response = requests.post(url, files=files, data=data)
    
    if response.status_code == 200:
        result = response.json()
        if 'error' in result:
            print(f"Error: {result['error']}")
        elif 'track' in result:
            print(f"Recognized: {result['track']['title']} by {result['track']['subtitle']}")
        else:
            print("No match found")
    else:
        print(f"Error: {response.status_code}")
    
    return response.json() if response.status_code == 200 else None

# Example usage
result = recognize_file("video.mp4", 30)  # Start recognition 30 seconds into the video
```

**Response Format:**

-   Success: JSON object containing track information
-   No match: JSON with error message
-   Error: HTTP error status with JSON error details

### 3. Recognize File Full

Performs comprehensive music identification across an entire file, potentially returning multiple song matches.

**Endpoint:** `POST /api/recognize-file-full`

**Request Format:**

python

```python
import requests

def recognize_file_full(file_path):
    url = "https://api.musicapihost.xyz/api/recognize-file-full"
    
    with open(file_path, 'rb') as file:
        files = {'videoFile': file}
        response = requests.post(url, files=files)
    
    if response.status_code == 200:
        result = response.json()
        if 'results' in result:
            print(f"Found {len(result['results'])} songs:")
            for i, song_match in enumerate(result['results'], 1):
                track = song_match.get('track', {})
                print(f"{i}. {track.get('title')} by {track.get('subtitle')}")
        else:
            print("No matches found")
    else:
        print(f"Error: {response.status_code}")
    
    return response.json() if response.status_code == 200 else None

# Example usage
results = recognize_file_full("music_compilation.mp4")
```

**Response Format:**

-   Success: JSON object containing an array of track matches
-   Error: HTTP error status with JSON error details

### 4. Recognize Link

Identifies music from various content links (YouTube, TikTok, etc.) at a specific timestamp.

**Endpoint:** `POST /api/recognize-link`

**Request Format:**

python

```python
import requests
import json

def recognize_link(url, hours=0, minutes=0, seconds=0, recaptcha_token=None):
    api_url = "https://api.musicapihost.xyz/api/recognize-link"
    
    data = {
        'link': url,
        'hours': str(hours).zfill(2),
        'minutes': str(minutes).zfill(2),
        'seconds': str(seconds).zfill(2)
    }
    
    if recaptcha_token:
        data['recaptchaToken'] = recaptcha_token
    
    headers = {'Content-Type': 'application/json'}
    response = requests.post(api_url, data=json.dumps(data), headers=headers)
    
    if response.status_code == 200:
        if response.text == 'fail':
            print("No match found")
            return None
        
        result = response.json()
        if 'track' in result:
            print(f"Recognized: {result['track']['title']} by {result['track']['subtitle']}")
        return result
    else:
        error_info = response.json() if response.text else {"error": "Unknown error"}
        print(f"Error ({response.status_code}): {error_info.get('error', 'Unknown error')}")
        return None

# Example usage
# Recognize music from a YouTube video at 1:30
result = recognize_link("https://www.youtube.com/watch?v=dQw4w9WgXcQ", minutes=1, seconds=30)

# Recognize music from a TikTok video
result = recognize_link("https://www.tiktok.com/@username/video/1234567890123456789")
```

**Response Format:**

-   Success: JSON object containing track information
-   No match: String "fail"
-   Error: HTTP error status with JSON error details

### 5. Recognize Link Full

Performs comprehensive music identification across an entire linked content, potentially returning multiple song matches.

**Endpoint:** `POST /api/recognize-link-full`

**Request Format:**

python

```python
import requests
import json

def recognize_link_full(url):
    api_url = "https://api.musicapihost.xyz/api/recognize-link-full"
    
    data = {'link': url}
    headers = {'Content-Type': 'application/json'}
    response = requests.post(api_url, data=json.dumps(data), headers=headers)
    
    if response.status_code == 200:
        result = response.json()
        if 'results' in result:
            print(f"Found {len(result['results'])} songs:")
            for i, song_match in enumerate(result['results'], 1):
                track = song_match.get('track', {})
                print(f"{i}. {track.get('title')} by {track.get('subtitle')}")
        else:
            print("No matches found")
        return result
    else:
        error_info = response.json() if response.text else {"error": "Unknown error"}
        print(f"Error ({response.status_code}): {error_info.get('error', 'Unknown error')}")
        return None

# Example usage
# Recognize all songs from a YouTube video
results = recognize_link_full("https://www.youtube.com/watch?v=dQw4w9WgXcQ")
```

**Response Format:**

-   Success: JSON object containing an array of track matches
-   Error: HTTP error status with JSON error details

## Error Handling

Common error responses include:

-   `400 Bad Request` - Invalid input parameters
-   `404 Not Found` - No match found
-   `429 Too Many Requests` - Rate limit exceeded
-   `500 Internal Server Error` - Server-side processing error
-   `503 Service Unavailable` - External service unavailable (e.g., YouTube recognition)
