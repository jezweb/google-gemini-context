# Video in Python

*Video understanding and analysis with Gemini*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/vision#video
SDK: google-genai (Python)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: Video understanding, frame analysis, multimodal
-->

## Quick Reference
- **Input formats**: MP4, AVI, MOV, FLV, WebM, MPEG, 3GPP
- **Max duration**: 1 hour
- **Max file size**: 2GB (via File API)
- **Frame sampling**: Automatic at 1 FPS

## Basic Video Understanding

```python
from google import genai
import pathlib

client = genai.Client()

# For videos under 20MB, use inline upload
video_file = pathlib.Path("video.mp4")

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        {"mime_type": "video/mp4", "data": video_file.read_bytes()},
        {"text": "Describe what happens in this video"}
    ]
)

print(response.text)
```

## Upload Large Videos (File API)

```python
from google import genai
import time

client = genai.Client()

# Upload video file
print("Uploading video...")
video_file = client.files.upload(path="large-video.mp4")

# Wait for processing
while video_file.state == "PROCESSING":
    print(".", end="", flush=True)
    time.sleep(5)
    video_file = client.files.get(name=video_file.name)

if video_file.state != "ACTIVE":
    raise ValueError(f"File processing failed: {video_file.state}")

print(f"\nVideo ready: {video_file.uri}")

# Use uploaded video
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        video_file,
        "Analyze this video in detail"
    ]
)

print(response.text)
```

## Video Analysis Class

```python
class VideoAnalyzer:
    def __init__(self):
        self.client = genai.Client()
    
    def analyze_video(self, video_path, analysis_type="general"):
        """Analyze video with different focus areas"""
        prompts = {
            "general": "Provide a comprehensive analysis of this video",
            "scene": """Analyze this video and provide:
                1. Scene-by-scene breakdown with timestamps
                2. Key objects and people identified
                3. Actions taking place
                4. Camera movements and techniques
                5. Overall narrative or purpose""",
            "educational": "Analyze as educational content: topics, teaching methods, key concepts",
            "security": "Analyze security footage: identify people, activities, unusual events",
            "sports": "Analyze sports video: plays, techniques, key moments",
            "tutorial": "Break down tutorial video into steps with timestamps"
        }
        
        # Handle file size
        file_size = pathlib.Path(video_path).stat().st_size
        
        if file_size < 20 * 1024 * 1024:  # Under 20MB
            video_data = pathlib.Path(video_path).read_bytes()
            contents = [
                {"mime_type": self._get_mime_type(video_path), "data": video_data},
                {"text": prompts.get(analysis_type, prompts["general"])}
            ]
        else:
            # Upload large file
            video_file = self._upload_video(video_path)
            contents = [
                video_file,
                {"text": prompts.get(analysis_type, prompts["general"])}
            ]
        
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=contents
        )
        
        return response.text
    
    def _upload_video(self, video_path):
        """Upload and wait for video processing"""
        print(f"Uploading {video_path}...")
        video_file = self.client.files.upload(path=video_path)
        
        while video_file.state == "PROCESSING":
            print(".", end="", flush=True)
            time.sleep(5)
            video_file = self.client.files.get(name=video_file.name)
        
        if video_file.state != "ACTIVE":
            raise ValueError(f"Upload failed: {video_file.state}")
        
        print("\nUpload complete!")
        return video_file
    
    def _get_mime_type(self, file_path):
        """Determine MIME type from file extension"""
        ext = pathlib.Path(file_path).suffix.lower()
        mime_types = {
            '.mp4': 'video/mp4',
            '.avi': 'video/avi',
            '.mov': 'video/quicktime',
            '.webm': 'video/webm',
            '.flv': 'video/x-flv',
            '.mpeg': 'video/mpeg',
            '.3gp': 'video/3gpp'
        }
        return mime_types.get(ext, 'video/mp4')
    
    def extract_text(self, video_path):
        """Extract all visible text from video"""
        if pathlib.Path(video_path).stat().st_size < 20 * 1024 * 1024:
            video_data = pathlib.Path(video_path).read_bytes()
            contents = [
                {"mime_type": self._get_mime_type(video_path), "data": video_data},
                {"text": "List all text that appears in this video (signs, captions, etc.)"}
            ]
        else:
            video_file = self._upload_video(video_path)
            contents = [
                video_file,
                {"text": "List all text that appears in this video (signs, captions, etc.)"}
            ]
        
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=contents
        )
        
        return response.text
    
    def generate_captions(self, video_path):
        """Generate video captions in SRT format"""
        if pathlib.Path(video_path).stat().st_size < 20 * 1024 * 1024:
            video_data = pathlib.Path(video_path).read_bytes()
            contents = [
                {"mime_type": self._get_mime_type(video_path), "data": video_data},
                {"text": "Generate captions for this video with timestamps in SRT format"}
            ]
        else:
            video_file = self._upload_video(video_path)
            contents = [
                video_file,
                {"text": "Generate captions for this video with timestamps in SRT format"}
            ]
        
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=contents
        )
        
        return response.text
```

## Common Use Cases

### Content Moderation
```python
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        video_file,
        "Check this video for inappropriate content, violence, or policy violations"
    ],
    config={
        "safety_settings": [
            {"category": "HARM_CATEGORY_DANGEROUS", "threshold": "BLOCK_LOW_AND_ABOVE"}
        ]
    }
)
```

### Video Summarization
```python
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        video_file,
        "Create a 3-paragraph summary suitable for a video description"
    ]
)
```

### Action Recognition
```python
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        video_file,
        "List all human actions/activities with timestamps"
    ]
)
```

## Streaming Video Analysis

```python
# Stream response for detailed analysis
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        video_file,
        "Provide a detailed minute-by-minute breakdown"
    ],
    config={"stream": True}
)

for chunk in response:
    print(chunk.text, end="", flush=True)
```

## Batch Video Processing

```python
def process_video_batch(video_paths):
    """Process multiple videos efficiently"""
    client = genai.Client()
    results = {}
    
    for path in video_paths:
        try:
            analyzer = VideoAnalyzer()
            results[path] = analyzer.analyze_video(path)
        except Exception as e:
            results[path] = f"Error: {str(e)}"
    
    return results
```

## Error Handling

```python
try:
    response = client.models.generate_content(
        model="gemini-2.5-flash",
        contents=[video_content, prompt]
    )
    print(response.text)
except Exception as e:
    if "exceeds the maximum" in str(e):
        print("Video too large or too long")
    elif "Unsupported video format" in str(e):
        print("Use MP4, AVI, MOV, WebM, etc.")
    elif "PROCESSING" in str(e):
        print("Video still processing, please wait")
    else:
        raise
```

## Best Practices

1. **File Size**: Use File API for videos >20MB
2. **Duration**: Keep under 1 hour for best results
3. **Quality**: Higher quality improves analysis
4. **Formats**: MP4 most reliable
5. **Cleanup**: Delete uploaded files when done
6. **Batch**: Process multiple videos efficiently

## See Also
- [Vision](vision.md) - Image analysis
- [Audio](audio.md) - Audio processing
- [Multimodal](multimodal.md) - Combined inputs