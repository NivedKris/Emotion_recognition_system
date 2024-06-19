# Emotion Detection Flask App

## Overview
This project is a Flask-based web application that uses OpenCV and the FER (Facial Emotion Recognition) library to detect emotions in real-time from a video feed. The application captures video from the webcam, detects emotions on faces in the frames, and displays the video with emotion labels overlayed on the detected faces.

## Features
- Real-time video capture from the webcam.
- Emotion detection on faces using the FER library.
- Overlay of detected emotions and bounding boxes on the video feed.
- Web interface to view the live video feed.

## Prerequisites
- Python 3.6+
- Pip (Python package installer)
- Webcam

## Installation

1. **Clone the Repository**
    ```bash
    git clone https://github.com/yourusername/emotion-detection-flask-app.git
    cd emotion-detection-flask-app
    ```

2. **Create a Virtual Environment**
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows use `venv\Scripts\activate`
    ```

3. **Install Dependencies**
    ```bash
    pip install -r requirements.txt
    ```

4. **Install OpenCV**
    ```bash
    pip install opencv-python
    pip install opencv-python-headless  # If needed for headless environments
    ```

5. **Install FER Library**
    ```bash
    pip install fer
    ```

## Usage

1. **Run the Application**
    ```bash
    python app.py
    ```

2. **Access the Application**
    Open a web browser and go to `http://localhost:80`.

## Project Structure

```
emotion-detection-flask-app/
│
├── app.py                  # Main application file
├── templates/
│   └── index.html          # HTML template for the web interface
├── static/
│   └── styles.css          # (Optional) CSS styles for the web interface
├── requirements.txt        # Python dependencies
└── README.md               # Project documentation
```

## app.py

This is the main Flask application file. It contains the following key components:

- **Imports and Initialization:**
    ```python
    from flask import Flask, render_template, Response
    import cv2
    from fer import FER

    app = Flask(__name__)
    detector = FER()
    ```

- **Frame Generation Function:**
    This function captures video frames, performs emotion detection, and yields the frames to be displayed in the web interface.
    ```python
    def generate_frames():
        cap = cv2.VideoCapture(1)
        while True:
            ret, frame = cap.read()
            if not ret:
                break

            emotions = detector.detect_emotions(frame)
            for face in emotions:
                x, y, w, h = face['box']
                emotion = max(face['emotions'], key=face['emotions'].get)
                cv2.rectangle(frame, (x, y), (x + w, y + h), (0, 255, 0), 2)
                cv2.putText(frame, emotion, (x, y - 10), cv2.FONT_HERSHEY_SIMPLEX, 0.9, (0, 255, 0), 2)

            ret, buffer = cv2.imencode('.jpg', frame)
            frame = buffer.tobytes()
            yield (b'--frame\r\n'
                   b'Content-Type: image/jpeg\r\n\r\n' + frame + b'\r\n')

        cap.release()
    ```

- **Routes:**
    - `/`: Renders the main page.
    - `/video_feed`: Provides the video feed to be displayed on the main page.
    ```python
    @app.route('/')
    def index():
        return render_template('index.html')

    @app.route('/video_feed')
    def video_feed():
        return Response(generate_frames(), mimetype='multipart/x-mixed-replace; boundary=frame')

    if __name__ == '__main__':
        app.run(host='0.0.0.0', port=80, debug=False)  # Set debug mode to False for production
    ```

## Templates

- **index.html:** The main HTML template for the web interface.
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Emotion Detection</title>
    </head>
    <body>
        <h1>Emotion Detection from Video Feed</h1>
        <img src="{{ url_for('video_feed') }}" alt="Video Feed">
    </body>
    </html>
    ```

## Contributing
Feel free to open issues or submit pull requests with improvements. Contributions are welcome!

## License
This project is licensed under the MIT License.

## Acknowledgments
- [OpenCV](https://opencv.org/)
- [FER Library](https://github.com/justinshenk/fer)
- Flask Documentation (https://flask.palletsprojects.com/en/2.0.x/)

---

