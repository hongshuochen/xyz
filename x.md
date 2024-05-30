Creating a website that records video using a webcam and uploads it to a server involves several steps. Here’s a basic example using Flask for the backend and HTML with JavaScript for the frontend.

### Backend: Flask Setup

1. **Install Flask and other dependencies**:
   ```bash
   pip install Flask flask-cors
   ```

2. **Create the Flask application**:
   - Create a file named `app.py`.

   ```python
   from flask import Flask, request, jsonify
   from flask_cors import CORS
   import os

   app = Flask(__name__)
   CORS(app)

   UPLOAD_FOLDER = 'uploads'
   if not os.path.exists(UPLOAD_FOLDER):
       os.makedirs(UPLOAD_FOLDER)

   @app.route('/upload', methods=['POST'])
   def upload():
       if 'video' not in request.files:
           return jsonify({'error': 'No file part'}), 400
       file = request.files['video']
       if file.filename == '':
           return jsonify({'error': 'No selected file'}), 400
       if file:
           filepath = os.path.join(UPLOAD_FOLDER, file.filename)
           file.save(filepath)
           return jsonify({'message': 'File successfully uploaded'}), 200

   if __name__ == '__main__':
       app.run(debug=True)
   ```

### Frontend: HTML and JavaScript

1. **Create an HTML file**:
   - Create a file named `index.html`.

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Video Recorder</title>
   </head>
   <body>
       <h1>Record Video</h1>
       <video id="video" width="640" height="480" autoplay></video>
       <button id="startButton">Start Recording</button>
       <button id="stopButton" disabled>Stop Recording</button>
       <button id="uploadButton" disabled>Upload Video</button>
       <script src="script.js"></script>
   </body>
   </html>
   ```

2. **Create a JavaScript file**:
   - Create a file named `script.js`.

   ```javascript
   const video = document.getElementById('video');
   const startButton = document.getElementById('startButton');
   const stopButton = document.getElementById('stopButton');
   const uploadButton = document.getElementById('uploadButton');

   let mediaRecorder;
   let recordedChunks = [];

   navigator.mediaDevices.getUserMedia({ video: true, audio: true })
       .then(stream => {
           video.srcObject = stream;
           mediaRecorder = new MediaRecorder(stream);

           mediaRecorder.ondataavailable = function(event) {
               if (event.data.size > 0) {
                   recordedChunks.push(event.data);
               }
           };

           mediaRecorder.onstop = function() {
               const blob = new Blob(recordedChunks, {
                   type: 'video/webm'
               });
               recordedChunks = [];
               const url = URL.createObjectURL(blob);
               const a = document.createElement('a');
               a.style.display = 'none';
               a.href = url;
               a.download = 'video.webm';
               document.body.appendChild(a);
               a.click();
               uploadButton.disabled = false;

               // Optional: save the video blob to a variable for uploading
               window.recordedBlob = blob;
           };
       });

   startButton.addEventListener('click', () => {
       mediaRecorder.start();
       startButton.disabled = true;
       stopButton.disabled = false;
       uploadButton.disabled = true;
   });

   stopButton.addEventListener('click', () => {
       mediaRecorder.stop();
       startButton.disabled = false;
       stopButton.disabled = true;
   });

   uploadButton.addEventListener('click', () => {
       const formData = new FormData();
       formData.append('video', window.recordedBlob, 'video.webm');

       fetch('http://127.0.0.1:5000/upload', {
           method: 'POST',
           body: formData
       })
       .then(response => response.json())
       .then(data => {
           console.log(data);
           alert('Video uploaded successfully!');
       })
       .catch(error => {
           console.error('Error:', error);
           alert('Video upload failed.');
       });
   });
   ```

### Instructions

1. **Run the Flask server**:
   ```bash
   python app.py
   ```

2. **Open `index.html` in your browser**:
   - Open the HTML file directly from your filesystem (e.g., by double-clicking it).

3. **Test the functionality**:
   - Allow access to your webcam when prompted.
   - Click "Start Recording" to begin recording.
   - Click "Stop Recording" to stop and save the recording.
   - Click "Upload Video" to upload the recorded video to the Flask server.

This example sets up a basic website with video recording and uploading capabilities. For a production setup, you might want to enhance error handling, improve UI/UX, and ensure security measures are in place.
